---
title: SDWebImage学习笔记（二）
date: 2016-12-30 14:46:12
tags: 三方库研究
---

# 缓存
这一小节主要研究下SDWebImage的cache功能是如何实现的，首先找到在SDWebImage中充当缓存功能的类，这一步很简单，直接定位到SDImageCache类。也就是说，接下来主要就研究这个类了。
## SDImageCacheConfig 缓存配置
SDWebImage提供了一个专门的类SDImageCacheConfig，来进行Cache的一些配置工作，主要配置的信息包括以下几点

1. 是否允许解压原图
2. 是否支持iCloud，默认不支持
3. 是否允许将图片缓存到内存，默认支持
4. <font color=red><B>最大缓存周期（默认是一周）</B></font>
5. 最大缓存大小（默认不设置）

## SDImageCache 缓存类
SDImageCache是一个单例，作为执行图片缓存的工具类，整个App只需要有一个全局的实例就可以了，这没毛病。
### SDImageCache文件系统
在开始缓存操作之前，除了配置项需要配置外，我个人感觉还有一个地方需要提前准备好的就是缓存的文件系统。
#### 默认的文件存储路径
SDImageCache的默认文件存储路径是由单例去控制的，在单例初始化的时候，传入了默认的文件夹名称default，这样的话，默认的文件路径就是../cache/default/com.hackemist.SDWebImageCache. default/ 文件夹。
#### 用户自己定义的文件路径
SDImageCache还支持用户自己定义的命名空间，所需要做的就是在创建的时候传入namespace，即可，这样的话，假如用户传入了YoukuImages，那么 缓存的文件路径就是../cache/YoukuImages/com.hackemist.SDWebImageCache.YoukuImages/ 文件夹了。
#### 辅助的文件路径方法
这个并不是很复杂，之所以要提一嘴，主要是比较赞赏SDWebImage的函数命名，代码如下
```objc
- (nullable NSString *)makeDiskCachePath:(nonnull NSString*)fullNamespace {
    NSArray<NSString *> *paths = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES);
    return [paths[0] stringByAppendingPathComponent:fullNamespace];
}
```
#### 图片存储路径
图片的存储路径采用了对URL进行MD5加密的方式，进行处理，通过这种方式防止文件被盗链给服务端增加流量损耗。
MD5也是很常见的加密方式，下面附上代码，方便自己以后查看。注意，这里稍微和别的MD5加密的不同的就是，会把文件扩展名带上，比如.png,.jpg之类的。这个方法，其实是提供了一个URL到文件名的映射
```objc
- (nullable NSString *)cachedFileNameForKey:(nullable NSString *)key {
    const char *str = key.UTF8String;
    if (str == NULL) {
        str = "";
    }
    unsigned char r[CC_MD5_DIGEST_LENGTH];
    CC_MD5(str, (CC_LONG)strlen(str), r);
    NSString *filename = [NSString stringWithFormat:@"%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%@",
                          r[0], r[1], r[2], r[3], r[4], r[5], r[6], r[7], r[8], r[9], r[10],
                          r[11], r[12], r[13], r[14], r[15], [key.pathExtension isEqualToString:@""] ? @"" : [NSString stringWithFormat:@".%@", key.pathExtension]];

    return filename;
}
```
另外提供俩个函数，方便获取文件默认存储路径下指定key图片的方法
```objc
// 获取指定路径下名称为key的图片路径
- (nullable NSString *)cachePathForKey:(nullable NSString *)key inPath:(nonnull NSString *)path {
    NSString *filename = [self cachedFileNameForKey:key];
    return [path stringByAppendingPathComponent:filename];
}
```
```objc
// 获取默认路径下key的图片路径，一般情况都是直接使用该方法就获取到图片的路径了
- (nullable NSString *)defaultCachePathForKey:(nullable NSString *)key {
    return [self cachePathForKey:key inPath:self.diskCachePath];
}
```

### SDImageCache的初始化
自从Xcode8之后，编译器好像做了一些优化，init方法需要去调用其他的NS_DESIGNATED_INITIALIZER标记的初始化方法，SDImageCache显然也是响应了这样的号召，所以提供了俩个自定义的初始化方法initWithNamespace:和initWithNamespace:diskCacheDirectory:，分别支持按照命名空间及存储路径的自定义初始化。但是单例的初始化会调用默认的初始化方法，使用单例的时候，会在App的Cache文件夹下面创建你一个default的文件夹，并在该Default文件夹下创建com.hackmisk.SDWebImageCache.default的一个文件夹，使用该文件夹来存储图片。下面是初始化的一些代码，并不是很复杂。
```objc
+ (nonnull instancetype)sharedImageCache {
    static dispatch_once_t once;
    static id instance;
    dispatch_once(&once, ^{
        instance = [self new];
    });
    return instance;
}

- (instancetype)init {
    return [self initWithNamespace:@"default"];
}

- (nonnull instancetype)initWithNamespace:(nonnull NSString *)ns {
    NSString *path = [self makeDiskCachePath:ns];
    return [self initWithNamespace:ns diskCacheDirectory:path];
}

- (nonnull instancetype)initWithNamespace:(nonnull NSString *)ns
                       diskCacheDirectory:(nonnull NSString *)directory {
    if ((self = [super init])) {
        NSString *fullNamespace = [@"com.hackemist.SDWebImageCache." stringByAppendingString:ns];
        
        // Create IO serial queue
        _ioQueue = dispatch_queue_create("com.hackemist.SDWebImageCache", DISPATCH_QUEUE_SERIAL);
        
        //目前这里使用的都是默认配置
        _config = [[SDImageCacheConfig alloc] init];
        
        // Init the memory cache
        // 创建内存缓存
        _memCache = [[AutoPurgeCache alloc] init];
        _memCache.name = fullNamespace;

        // Init the disk cache
        if (directory != nil) {
        // 单例形式创建的时候，这里肯定是能执行到的，之后就会创建com.hackmisk.SDWebImageCache.default的缓存路径
            _diskCachePath = [directory stringByAppendingPathComponent:fullNamespace];
        } else {
            NSString *path = [self makeDiskCachePath:ns];
            _diskCachePath = path;
        }
        // 此处需要稍微注意一下，因为io队列主要是对图片进行增删读写等操作，需要保证fileManager创建在io队列上。
        dispatch_sync(_ioQueue, ^{
            _fileManager = [NSFileManager new];
        });

#if SD_UIKIT
        // Subscribe to app events
        [[NSNotificationCenter defaultCenter] addObserver:self
                                                 selector:@selector(clearMemory)
                                                     name:UIApplicationDidReceiveMemoryWarningNotification
                                                   object:nil];

        [[NSNotificationCenter defaultCenter] addObserver:self
                                                 selector:@selector(deleteOldFiles)
                                                     name:UIApplicationWillTerminateNotification
                                                   object:nil];

        [[NSNotificationCenter defaultCenter] addObserver:self
                                                 selector:@selector(backgroundDeleteOldFiles)
                                                     name:UIApplicationDidEnterBackgroundNotification
                                                   object:nil];
#endif
    }

    return self;
}
//辅助方法，便捷式创建缓存路径
- (nullable NSString *)makeDiskCachePath:(nonnull NSString*)fullNamespace {
    NSArray<NSString *> *paths = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES);
    return [paths[0] stringByAppendingPathComponent:fullNamespace];
}
```
### SDImageCache的缓存操作
#### 存
这里同样提供了三个方法方便使用者根据不同的需求进行图片存储功能的选择。
```objc
// 使用默认的存储路径，并默认存储到磁盘
- (void)storeImage:(nullable UIImage *)image
            forKey:(nullable NSString *)key
        completion:(nullable SDWebImageNoParamsBlock)completionBlock {
    [self storeImage:image imageData:nil forKey:key toDisk:YES completion:completionBlock];
}

// 该方法可供用户选择是否能够保存到磁盘
- (void)storeImage:(nullable UIImage *)image
            forKey:(nullable NSString *)key
            toDisk:(BOOL)toDisk
        completion:(nullable SDWebImageNoParamsBlock)completionBlock {
    [self storeImage:image imageData:nil forKey:key toDisk:toDisk completion:completionBlock];
}

// 此处提供了图片的NSdata数据，可供用户直接存储。
- (void)storeImage:(nullable UIImage *)image
         imageData:(nullable NSData *)imageData
            forKey:(nullable NSString *)key
            toDisk:(BOOL)toDisk
        completion:(nullable SDWebImageNoParamsBlock)completionBlock {
    // 做下异常校验，如果图片或者key不存在，直接返回    
    if (!image || !key) {
        if (completionBlock) {
            completionBlock();
        }
        return;
    }
    // 检验下配置类是否允许将图片保存到内存，如果允许就进行保存处理
    if (self.config.shouldCacheImagesInMemory) {
        NSUInteger cost = SDCacheCostForImage(image);
        [self.memCache setObject:image forKey:key cost:cost];
    }
    // 是否需要保存到磁盘
    if (toDisk) {
        // 保存需要在IO队列中进行，先检验图片NSdata是否存在，如果只有UIImage对象，需要先转换成Data类型
        dispatch_async(self.ioQueue, ^{
            NSData *data = imageData;
            
            if (!data && image) {
                SDImageFormat imageFormatFromData = [NSData sd_imageFormatForImageData:data];
                data = [image sd_imageDataAsFormat:imageFormatFromData];
            }
            
            [self storeImageDataToDisk:data forKey:key];
            if (completionBlock) {
                dispatch_async(dispatch_get_main_queue(), ^{
                    completionBlock();
                });
            }
        });
    } else {
        if (completionBlock) {
            completionBlock();
        }
    }
}

// 保存数据到磁盘，注意保存操作要在IO队列上进行。并根据是否要保存到iCloud对URL进行设置
- (void)storeImageDataToDisk:(nullable NSData *)imageData forKey:(nullable NSString *)key {
    if (!imageData || !key) {
        return;
    }
    
    // 检查下是否是在IO队列上，因为读写操作要在单独的异步队列进行
    [self checkIfQueueIsIOQueue];
    
    // 检查是否在当前的目录下存在文件夹，如不存在，先创建之
    if (![_fileManager fileExistsAtPath:_diskCachePath]) {
        [_fileManager createDirectoryAtPath:_diskCachePath withIntermediateDirectories:YES attributes:nil error:NULL];
    }
    
    // 获取缓存图片地址文件夹路径
    NSString *cachePathForKey = [self defaultCachePathForKey:key];
    // 转变为文件URL
    NSURL *fileURL = [NSURL fileURLWithPath:cachePathForKey];
    
    // 保存操作
    [_fileManager createFileAtPath:cachePathForKey contents:imageData attributes:nil];
    
    // 阻止图片保存到iCloud
    if (self.config.shouldDisableiCloud) {
        [fileURL setResourceValue:@YES forKey:NSURLIsExcludedFromBackupKey error:nil];
    }
}
```
#### 取

#### 删

#### 查