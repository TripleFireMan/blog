---
title: SDWebImage学习笔记（一）
date: 2016-12-30 10:36:10
tags: 三方库研究
---
# 简介

SDWebImage是ios开发中，最常见的图片加载框架，它主要实现了图片异步加载、图片缓存，并提供了UIImageView、UIButton、MKAnnotationview的类目，使用体验很友好，也很方便，成为广大ios开发者加载网络图片的选择，今天我主要是来通过分析其源码来研究下，SDWebimage到底是如何进行设计的，架构的? 

<!--more-->

# 特性
* 提供UIimageview、UIbutton、MKAnnotationview的类目加载网络图片及缓存管理
* 异步的图片下载
* <font color=red><B>异步的图片内存+磁盘图片缓存，并支持自动的缓存过期处理</B></font>
* 图片的后台解压
* <font color=red><B>确保同一个url不会下载多次（是优点也是缺点）</B></font>
* 错误的url不会不停的下载
* 永远不会阻塞主线程
* 性能提升
* 采用GCD和ARC

# 支持的图片格式
* 支持JPEG,PNG,GIF
* 支持WEBP

# 使用要求
* ios 7.0 +
* tvos 9.0 +
* watchos 2.0 + 
* osx 10.8 + 
* xcode 7.3 +

# 常见问题
* 如果UITableViewCell使用了动态的图片大小，图片展示可能会有问题，也就是说SDWebImage是根据placeholder的大小来设置UIImageView的大小的，如果要展示的图片大小和placeholder的图片大小不一致就会有一些问题，解决方案[<font color=blue>点击这里</font>](http://www.wrichards.com/blog/2011/11/sdwebimage-fixed-width-cell-images/)

* 手动去刷新图片，SDWebImage使用了暴力的图片缓存方式，不会关注HTTP 的header里面缓存的策略，直接根据图片的URL地址进行缓存，也就是说一个URL会对应一张图片，如果图片地址不发生变化的话，图片永远不会重新下载，因此在某些场景下，你需要手动去刷新图片。
 
# 架构图
![架构图](http://ock9zbzms.bkt.clouddn.com/SDWebImageClassDiagram.png)

# 正文
上文是SDWebImageView官方的一些文档，我这里给简要的翻译了下，可以看的出来，SDWebImage虽然功能很强大，但是依然还是有一些使用中存在的问题。接下来，我将会通过逐个分析代码的方式，将SDWebImageView从下载、缓存、管理等等一层一层剥开它神秘的面纱。在这个过程中，我尽量避免过多的纠结于一些细节，但是同样的，有些时候为了说明一些问题，难免也会贴一些代码。

## 下载
SDWebimageview的下载是通过NSURLSession的方式，并通过继承NSOperation来异步的进行下载。下载过程中是通过发送通知的方式进行消息通信。
```objc
NSString *const SDWebImageDownloadStartNotification = @"SDWebImageDownloadStartNotification";
NSString *const SDWebImageDownloadReceiveResponseNotification = @"SDWebImageDownloadReceiveResponseNotification";
NSString *const SDWebImageDownloadStopNotification = @"SDWebImageDownloadStopNotification";
NSString *const SDWebImageDownloadFinishNotification = @"SDWebImageDownloadFinishNotification";
```
### SDWebImageDownloaderOperation任务的创建及取消
 SDWebImageDownloaderOperation通过确认下面的这俩个协议实际上执行下载图片的工作，接下来就研究下其内部是怎么工作的。
```objc
@protocol SDWebImageDownloaderOperationInterface<NSObject>
//通过该方法进行Opearation的创建
- (nonnull instancetype)initWithRequest:(nullable NSURLRequest *)request
                              inSession:(nullable NSURLSession *)session
                                options:(SDWebImageDownloaderOptions)options;
//添加下载过程中需要的回调，主要包括下载进度、下载完成的回调
- (nullable id)addHandlersForProgress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                            completed:(nullable SDWebImageDownloaderCompletedBlock)completedBlock;
//是否支持解压图片
- (BOOL)shouldDecompressImages;
//设置是否支持解压图片
- (void)setShouldDecompressImages:(BOOL)value;
//URL鉴定
- (nullable NSURLCredential *)credential;
//设置URL鉴定
- (void)setCredential:(nullable NSURLCredential *)value;

@end

@protocol SDWebImageOperation <NSObject>

- (void)cancel;

@end
```
### SDWebImageDownloaderOperation任务的初始化以及任务的执行
忽略掉那些细节，这里只关注关键的几个地方，SDWebImageDownloaderOperation，以下简称‘下载操作对象’，通过持有Request，注入session，以及options，创建好下载操作对象。之后通过外部调用start方法，开启下载任务。并通过设置session任务的代理，来监听下载过程，同时发出相应的通知进行对象间的消息通信。

1. 初始化
```objc
- (nonnull instancetype)initWithRequest:(nullable NSURLRequest *)request
                              inSession:(nullable NSURLSession *)session
                                options:(SDWebImageDownloaderOptions)options {
    if ((self = [super init])) {
        _request = [request copy];
        _shouldDecompressImages = YES;
        _options = options;
        _callbackBlocks = [NSMutableArray new];
        _executing = NO;
        _finished = NO;
        _expectedSize = 0;
        _unownedSession = session;
        responseFromCached = YES; // Initially wrong until `- URLSession:dataTask:willCacheResponse:completionHandler: is called or not called
        _barrierQueue = dispatch_queue_create("com.hackemist.SDWebImageDownloaderOperationBarrierQueue", DISPATCH_QUEUE_CONCURRENT);
    }
    return self;
}
```
2. 执行任务
```objc
- (void)start {
    @synchronized (self) {
        if (self.isCancelled) {
            self.finished = YES;
            [self reset];
            return;
        }

#if SD_UIKIT
        Class UIApplicationClass = NSClassFromString(@"UIApplication");
        BOOL hasApplication = UIApplicationClass && [UIApplicationClass respondsToSelector:@selector(sharedApplication)];
        //对后台下载任务的支持，app可以开启后台下载任务，并返回一个后台下载的ID，并设置一个过期的callback，此处注意当任务失效的时候，需要将任务终止，并设置taskid为无效
        if (hasApplication && [self shouldContinueWhenAppEntersBackground]) {
            __weak __typeof__ (self) wself = self;
            UIApplication * app = [UIApplicationClass performSelector:@selector(sharedApplication)];
            self.backgroundTaskId = [app beginBackgroundTaskWithExpirationHandler:^{
                __strong __typeof (wself) sself = wself;

                if (sself) {
                    [sself cancel];

                    [app endBackgroundTask:sself.backgroundTaskId];
                    sself.backgroundTaskId = UIBackgroundTaskInvalid;
                }
            }];
        }
#endif
        //默认是有注入session，如果没有的话，需要内部创建一个session
        NSURLSession *session = self.unownedSession;
        if (!self.unownedSession) {
            NSURLSessionConfiguration *sessionConfig = [NSURLSessionConfiguration defaultSessionConfiguration];
            sessionConfig.timeoutIntervalForRequest = 15;
            
            /**
             *  Create the session for this task
             *  We send nil as delegate queue so that the session creates a serial operation queue for performing all delegate
             *  method calls and completion handler calls.
             */
            self.ownedSession = [NSURLSession sessionWithConfiguration:sessionConfig
                                                              delegate:self
                                                         delegateQueue:nil];
            session = self.ownedSession;
        }
        //此处根据当前的session和request创建datatask
        self.dataTask = [session dataTaskWithRequest:self.request];
        self.executing = YES;
    }
    
    //datatask开始工作
    [self.dataTask resume];

    if (self.dataTask) {
        //通知所有监听回调，任务开始执行了。俩种方式，一种是callback方式，一种是通知
        for (SDWebImageDownloaderProgressBlock progressBlock in [self callbacksForKey:kProgressCallbackKey]) {
            progressBlock(0, NSURLResponseUnknownLength, self.request.URL);
        }
        dispatch_async(dispatch_get_main_queue(), ^{
            [[NSNotificationCenter defaultCenter] postNotificationName:SDWebImageDownloadStartNotification object:self];
        });
    } else {
        //对异常情况的处理，如果datatask为nil，进行错误的回调
        [self callCompletionBlocksWithError:[NSError errorWithDomain:NSURLErrorDomain code:0 userInfo:@{NSLocalizedDescriptionKey : @"Connection can't be initialized"}]];
    }

#if SD_UIKIT
    Class UIApplicationClass = NSClassFromString(@"UIApplication");
    if(!UIApplicationClass || ![UIApplicationClass respondsToSelector:@selector(sharedApplication)]) {
        return;
    }
    if (self.backgroundTaskId != UIBackgroundTaskInvalid) {
        UIApplication * app = [UIApplication performSelector:@selector(sharedApplication)];
        [app endBackgroundTask:self.backgroundTaskId];
        self.backgroundTaskId = UIBackgroundTaskInvalid;
    }
#endif
}
```
3. 接下来就是NSURLSession任务的开启、执行中、结束或者错误回调
这里就不去贴代码了，要不然这篇博文很大篇幅都被代码占据了，其实任务执行开始之后，主要就是各种异常的处理，因为正常的处理其实是比较简单的，之后就会回调到外面去，让外面调用‘下载操作对象’去决定到底该如何处理。
（补充：经过后面对SDWebImageDownloader的深入研究，发现SDWebImageDownloaderOperation创建好之后，直接扔到SDWebImageDownloader的队列里面去了，此外session由于是注入的，代理设置的是SDWebImageDownloader，回调自然也由SDWebImageDownloader接收，然后分发给各自对应的SDWebImageDownloaderOperation）

### SDWebImageDownloader对下载任务的封装
SDWebImageDownloader是对SDWebImageDownloaderOperation的进一步管理和封装，通过下载队列对SDWebImageDownloaderOperation的任务并发数(默认并发数为6个)，执行顺序（默认是FIFO）进行管理。这里我简单梳理下任务的下载流程
* client（此处指的是使用SDWebImageDownloader的客户，可以是用户自己的类，也可以是其他SDWebImage类）首先需要创建一个SDWebImageDownloader，创建好之后，就准备好了下载需要的downloadQueue，下载相关的一些配置（诸如执行顺序、并发数、超时等等）。
* client之后调用SDWebImageDownloader的downloadImageWithURL:options:progress:completed:进行实际的下载操作。在这里我有一个疑惑了好久的问题，就是session的回调问题，因为SDWebImageDownloaderOperation里面有session，而SDWebImageDownloader也有session，那么任务执行过程中，岂不是俩处都会收到回调？这难道是SDWebImage的BUG？呵呵，其实并不是，直到我看到下面这段代码。
```objc
        NSURLSession *session = self.unownedSession;
        if (!self.unownedSession) {
            NSURLSessionConfiguration *sessionConfig = [NSURLSessionConfiguration defaultSessionConfiguration];
            sessionConfig.timeoutIntervalForRequest = 15;
            
            /**
             *  Create the session for this task
             *  We send nil as delegate queue so that the session creates a serial operation queue for performing all delegate
             *  method calls and completion handler calls.
             */
            //注意这里，只有ownedSession才会设置代理为自己，也就是说注入的session不会设置代理。
            self.ownedSession = [NSURLSession sessionWithConfiguration:sessionConfig
                                                              delegate:self
                                                         delegateQueue:nil];
            session = self.ownedSession;
        }
        
        self.dataTask = [session dataTaskWithRequest:self.request];
```
 看了上面的代码自然就明白了，<font color=red>当使用SDWebImageDownloader的时候，session的回调只有SDWebImageDownloader能接收到，这也是为什么SDWebImageDownloader需要在接收到回调之后要进行转发的缘故。</font>

1. SDWebImageDownloader的初始化
```objc
- (nonnull instancetype)initWithSessionConfiguration:(nullable NSURLSessionConfiguration *)sessionConfiguration {
    if ((self = [super init])) {
        _operationClass = [SDWebImageDownloaderOperation class];
        _shouldDecompressImages = YES;
        //设置任务的执行顺序
        _executionOrder = SDWebImageDownloaderFIFOExecutionOrder;
        //创建执行任务的操作队列，并对操作队列进行配置
        _downloadQueue = [NSOperationQueue new];
        _downloadQueue.maxConcurrentOperationCount = 6;
        _downloadQueue.name = @"com.hackemist.SDWebImageDownloader";
        _URLOperations = [NSMutableDictionary new];
#ifdef SD_WEBP
        _HTTPHeaders = [@{@"Accept": @"image/webp,image/*;q=0.8"} mutableCopy];
#else
        _HTTPHeaders = [@{@"Accept": @"image/*;q=0.8"} mutableCopy];
#endif
        _barrierQueue = dispatch_queue_create("com.hackemist.SDWebImageDownloaderBarrierQueue", DISPATCH_QUEUE_CONCURRENT);
        _downloadTimeout = 15.0;

        sessionConfiguration.timeoutIntervalForRequest = _downloadTimeout;

        /**
         *  Create the session for this task
         *  We send nil as delegate queue so that the session creates a serial operation queue for performing all delegate
         *  method calls and completion handler calls.
         */
        self.session = [NSURLSession sessionWithConfiguration:sessionConfiguration
                                                     delegate:self
                                                delegateQueue:nil];
    }
    return self;
}
```
2. SDWebImageDownloader执行下载操作
这里请原谅我贴上大段的代码，因为这段代码实在是太漂亮了。通过俩个函数的设计将SDWebImageDownloaderOperation的初始化和SDWebImageDownloadToken初始化分割开。并在初始化operation完成之后将任务扔到队列中去执行，还持有了operation的引用，给SDWebImageDownloader有机会保存到字典中。

```objc
- (nullable SDWebImageDownloadToken *)downloadImageWithURL:(nullable NSURL *)url
                                                   options:(SDWebImageDownloaderOptions)options
                                                  progress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                                                 completed:(nullable SDWebImageDownloaderCompletedBlock)completedBlock {
    __weak SDWebImageDownloader *wself = self;

    return [self addProgressCallback:progressBlock completedBlock:completedBlock forURL:url createCallback:^SDWebImageDownloaderOperation *{
        __strong __typeof (wself) sself = wself;
        NSTimeInterval timeoutInterval = sself.downloadTimeout;
        if (timeoutInterval == 0.0) {
            timeoutInterval = 15.0;
        }

        // In order to prevent from potential duplicate caching (NSURLCache + SDImageCache) we disable the cache for image requests if told otherwise
        NSMutableURLRequest *request = [[NSMutableURLRequest alloc] initWithURL:url cachePolicy:(options & SDWebImageDownloaderUseNSURLCache ? NSURLRequestUseProtocolCachePolicy : NSURLRequestReloadIgnoringLocalCacheData) timeoutInterval:timeoutInterval];
        request.HTTPShouldHandleCookies = (options & SDWebImageDownloaderHandleCookies);
        request.HTTPShouldUsePipelining = YES;
        if (sself.headersFilter) {
            request.allHTTPHeaderFields = sself.headersFilter(url, [sself.HTTPHeaders copy]);
        }
        else {
            request.allHTTPHeaderFields = sself.HTTPHeaders;
        }
        SDWebImageDownloaderOperation *operation = [[sself.operationClass alloc] initWithRequest:request inSession:sself.session options:options];
        operation.shouldDecompressImages = sself.shouldDecompressImages;
        
        if (sself.urlCredential) {
            operation.credential = sself.urlCredential;
        } else if (sself.username && sself.password) {
            operation.credential = [NSURLCredential credentialWithUser:sself.username password:sself.password persistence:NSURLCredentialPersistenceForSession];
        }
        
        if (options & SDWebImageDownloaderHighPriority) {
            operation.queuePriority = NSOperationQueuePriorityHigh;
        } else if (options & SDWebImageDownloaderLowPriority) {
            operation.queuePriority = NSOperationQueuePriorityLow;
        }

        [sself.downloadQueue addOperation:operation];
        if (sself.executionOrder == SDWebImageDownloaderLIFOExecutionOrder) {
            // Emulate LIFO execution order by systematically adding new operations as last operation's dependency
            [sself.lastAddedOperation addDependency:operation];
            sself.lastAddedOperation = operation;
        }

        return operation;
    }];
}
```
这里有对SDWebImageDownloaderOperation进行保存处理，确保同一个URL只会创建一次，避免内存的额外消耗。添加对token的初始化和管理，把operation的初始化和token的初始化进行业务上一些小分离，代码层次立马清晰起来了，赞！
```objc
- (nullable SDWebImageDownloadToken *)addProgressCallback:(SDWebImageDownloaderProgressBlock)progressBlock
                                           completedBlock:(SDWebImageDownloaderCompletedBlock)completedBlock
                                                   forURL:(nullable NSURL *)url
                                           createCallback:(SDWebImageDownloaderOperation *(^)())createCallback {
    // The URL will be used as the key to the callbacks dictionary so it cannot be nil. If it is nil immediately call the completed block with no image or data.
    if (url == nil) {
        if (completedBlock != nil) {
            completedBlock(nil, nil, nil, NO);
        }
        return nil;
    }

    __block SDWebImageDownloadToken *token = nil;

    dispatch_barrier_sync(self.barrierQueue, ^{
        SDWebImageDownloaderOperation *operation = self.URLOperations[url];
        if (!operation) {
            operation = createCallback();
            self.URLOperations[url] = operation;

            __weak SDWebImageDownloaderOperation *woperation = operation;
            operation.completionBlock = ^{
              SDWebImageDownloaderOperation *soperation = woperation;
              if (!soperation) return;
              if (self.URLOperations[url] == soperation) {
                  [self.URLOperations removeObjectForKey:url];
              };
            };
        }
        id downloadOperationCancelToken = [operation addHandlersForProgress:progressBlock completed:completedBlock];

        token = [SDWebImageDownloadToken new];
        token.url = url;
        token.downloadOperationCancelToken = downloadOperationCancelToken;
    });

    return token;
}
```

# 小结
 好了，SDWebImageView的下载环节就到这里，看来那句话确实没说错，源码面前，没有神秘，哈哈！下一小节主要研究下SDWebImageView是如何对图片进行缓存操作的。



***
参考资料
[1]. [SDWebimage在github上的地址](https://github.com/rs/SDWebImage)
[2]. [英文版的使用说明](http://cocoadocs.org/docsets/SDWebImage/3.8.2/)
[3]. [使用SDWebimage的app列表](https://github.com/rs/SDWebImage/wiki/Who-Uses-SDWebImage)
***