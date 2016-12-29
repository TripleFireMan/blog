---
title: SDWebImage学习笔记
date: 2016-12-29 14:36:10
tags: 点点滴滴
---
# 简介

SDWebImage是ios开发中，最常见的图片加载框架，它主要实现了图片异步加载、图片缓存，并提供了UIImageView、UIButton、MKAnnotationview的类目，使用体验很友好，也很方便，成为广大ios开发者加载网络图片的选择，今天我主要是来通过分析其源码来研究下，SDWebimage到底是如何进行设计的，架构的?

<!--more-->

# 特性
* 提供UIimageview、UIbutton、MKAnnotationview的类目加载网络图片及缓存管理
* 异步的图片下载
* 异步的图片内存+磁盘图片缓存，并支持自动的缓存过期处理
* 图片的后台解压
* 确保同一个url不会下载多次（是优点也是缺点）
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
* 如果UItableviewcell使用了动态的图片大小，图片展示可能会有问题，也就是说SDWEBimage是根据placeholder的大小来设置UIimageview的大小的，如果要展示的图片大小和placeholder的图片大小不一致就会有一些问题，解决方案[点击这里](http://www.wrichards.com/blog/2011/11/sdwebimage-fixed-width-cell-images/)

* 手动去刷新图片，SDWebimage使用了暴力的图片缓存方式，不会关注HTTP 的header里面缓存的策略，直接根据图片的URL地址进行缓存，也就是说一个URL会对应一张图片，如果图片地址不发生变化的话，图片永远不会重新下载，因此在某些场景下，你需要手动去刷新图片。
 
# 架构图
![架构图](http://ock9zbzms.bkt.clouddn.com/SDWebImageClassDiagram.png)

# 正文
之前是SDWebimageview官方的一些文档，我这里给简要的翻译了下，可以看的出来，SDWebImage虽然功能很强大，但是依然还是有一些使用中存在的问题。接下来，我将会通过逐个分析代码的方式，将SDWebImageView从下载、缓存、管理等等一层一层剥开它神秘的面纱。在这个过程中，我尽量避免过多的纠结于一些细节，但是同样的不贴代码似乎也没办法说明一些事情。

## 下载
SDWebimageview的下载是通过NSURLSession的方式，并通过继承NSOperation来异步的进行下载。下载过程中是通过发送通知的方式进行消息通信。
```objc
NSString *const SDWebImageDownloadStartNotification = @"SDWebImageDownloadStartNotification";
NSString *const SDWebImageDownloadReceiveResponseNotification = @"SDWebImageDownloadReceiveResponseNotification";
NSString *const SDWebImageDownloadStopNotification = @"SDWebImageDownloadStopNotification";
NSString *const SDWebImageDownloadFinishNotification = @"SDWebImageDownloadFinishNotification";
```
### 任务的创建及取消
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
[参考资料]

* [SDWebimage在github上的地址](https://github.com/rs/SDWebImage)
* [英文版的使用说明](http://cocoadocs.org/docsets/SDWebImage/3.8.2/)
* [使用SDWebimage的app列表](https://github.com/rs/SDWebImage/wiki/Who-Uses-SDWebImage)