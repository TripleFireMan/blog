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
    if (!image || !key) {
        if (completionBlock) {
            completionBlock();
        }
        return;
    }
    // if memory cache is enabled
    if (self.config.shouldCacheImagesInMemory) {
        NSUInteger cost = SDCacheCostForImage(image);
        [self.memCache setObject:image forKey:key cost:cost];
    }
    
    if (toDisk) {
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
    
    [self checkIfQueueIsIOQueue];
    
    if (![_fileManager fileExistsAtPath:_diskCachePath]) {
        [_fileManager createDirectoryAtPath:_diskCachePath withIntermediateDirectories:YES attributes:nil error:NULL];
    }
    
    // get cache Path for image key
    NSString *cachePathForKey = [self defaultCachePathForKey:key];
    // transform to NSUrl
    NSURL *fileURL = [NSURL fileURLWithPath:cachePathForKey];
    
    [_fileManager createFileAtPath:cachePathForKey contents:imageData attributes:nil];
    
    // disable iCloud backup
    if (self.config.shouldDisableiCloud) {
        [fileURL setResourceValue:@YES forKey:NSURLIsExcludedFromBackupKey error:nil];
    }
}
```