---
title: SDWebImage学习笔记（三）
date: 2017-01-16 11:41:33
tags: 三方库研究
---
上一小节研究了SDWebImageView里面的缓存实现原理，在这一小节我们继续研究SDWebImage对缓存和下载整体功能的封装。也就是-SDWebImageManager管理类。
<!--more-->

# 组装
SDWebImageManager使用了组装的设计模式，通过内部包含SDWebImageCache和SDWebImageDownloader的成员变量来实现下载和缓存的功能。
```objc
@interface SDWebImageManager : NSObject
@property (strong, nonatomic, readonly, nullable) SDImageCache *imageCache;
@property (strong, nonatomic, readonly, nullable) SDWebImageDownloader *imageDownloader;
```
接入，下载和缓存对象可以由初始化的时候外部传入，但是官方建议的还是在SDWebImageManager初始化的时候进行初始化。
```objc
+ (nonnull instancetype)sharedManager {
    static dispatch_once_t once;
    static id instance;
    dispatch_once(&once, ^{
        instance = [self new];
    });
    return instance;
}

- (nonnull instancetype)init {
    SDImageCache *cache = [SDImageCache sharedImageCache];
    SDWebImageDownloader *downloader = [SDWebImageDownloader sharedDownloader];
    return [self initWithCache:cache downloader:downloader];
}

- (nonnull instancetype)initWithCache:(nonnull SDImageCache *)cache downloader:(nonnull SDWebImageDownloader *)downloader {
    if ((self = [super init])) {
        _imageCache = cache;
        _imageDownloader = downloader;
        _failedURLs = [NSMutableSet new];
        _runningOperations = [NSMutableArray new];
    }
    return self;
}
```

# 代理
SDWebImageManager使用了代理的设计模式，给使用者提供一个可以过滤下载URL和对下载后的图片进行转码的一个外部接口，代码如下：
```objc
@protocol SDWebImageManagerDelegate <NSObject>

@optional

/**
 * 控制图片是否允许被下载，当这个图片在缓存中没找到时候
 *
 * @param imageManager 图片管理器
 * @param imageURL     要下载的图片URL
 *
 * @return 返回NO，当缓存没有命中的时候，阻止图片下载。如果没实现，默认是YES
 * 
 */
- (BOOL)imageManager:(nonnull SDWebImageManager *)imageManager shouldDownloadImageForURL:(nullable NSURL *)imageURL;

/**
 * 允许转变图片，当图片被从网上下载下来，但是还没有缓存到磁盘和内存中之前。
 * NOTE: 这个方法是在全局的队列中调用的。以防对主线程造成阻塞
 *
 * @param imageManager The current `SDWebImageManager`
 * @param image        The image to transform
 * @param imageURL     The url of the image to transform
 *
 * @return The transformed image object.
 */
- (nullable UIImage *)imageManager:(nonnull SDWebImageManager *)imageManager transformDownloadedImage:(nullable UIImage *)image withURL:(nullable NSURL *)imageURL;

@end
```

# 核心API
## <font color=red>外部唤起下载操作</font>
 SDWebImageManager提供了一个外部传入URL和SDWebImageOptions，而后可以获取下载过程回调，以及下载完成回调的一个核心API，该核心API会返回一个确认SDWebImageOperation协议的对象。


## 取消所有请求
## 检测是否在运行
## 缓存某URL对应的图片到缓存
## 异步检测内存中/磁盘中是否有图片
## 获取某个URL对应的缓存key





