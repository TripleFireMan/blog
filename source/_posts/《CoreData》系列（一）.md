---
title: 《CoreData》系列（一）
date: 2016-08-27 16:47:45
tags: coredata学习
comments: true
---



## 《题外篇》

<!---more-->

学习这个东西贵在日积月累，而且事情往往说起来容易，做起来难，我是一个资深dota玩家，从dota1到dota2，从大学到工作，从2008年到2015年。一直看2009的视频，经常吐槽09视频更新速度慢，但是细细想想，09能保持优酷更新401（最近查看）期视频。又有多少人能做的到。
故而最近下了一个决定，每周五务必更新一篇技术博客，就看看自己能坚持多久。

## 《正文》

[1 概述](#1)
本系列研究讨论的是iOS开发中的一种数据持久化技术－coredata。coredata、sqlite、fmdb的优缺点不是我要讨论的重点

这个系列的blog主要会研究讨论以下几点
	1.快速搭建coredata环境，主要是连接数据库、创建数据库托管对象模型（NSManagerObject）、如何保存数据、查询数据？
	2.coredata升级以及数据迁移的三种方式。
	3.coredata与viewcontroller的结合，通过NSFetchedResultController使用coredata数据。
	4.导入默认数据和前后台context。
	5.关系

[2 环境搭建](#1)

[2.1 导入《CoreData》的framework](#2)
默认读者知道如何创建一个空白的项目，建立好空白项目之后，搜索coredata按图2-1操作，点击添加完成framework的引入。
![图 2-1](http://ock9zbzms.bkt.clouddn.com/20151030175726289.jpg)

[2.2 创建Model](#2)
完成上述第一步，意味着我们已经可以使用CoreData提供的接口API了，接下来就是如何使用的事儿了。创建一个CoreData文件夹，专门用来放CoreData引擎，创建好文件夹后，右键点击选择newfile，然后按照图2-2所示创建数据库模型文件，并将其命名为Model,然后点击Model,添加Entity（表\Class）,添加Attribute（字段\属性），到这一部，基本上就把Model，创建出来了。并且里面有了数据模型结构，接下来的问题就是，连接数据库，根据模型创建托管对象了
![图 2-2](http://ock9zbzms.bkt.clouddn.com/20151030180051371.jpg)

[2.3 代码连接数据库](#2)
好啦，前面只是开胃菜，真正的大餐马上就要来了，在吃大餐前，有一些名称需要说明下

1.NSManagedObjectContext - 托管对象上下文，用来干嘛的呢？望文生意，用来管理托管对象的，负责从数据库中获取对象、保存对象、删除对象等等操作。
2.NSManagedObjectModel - 对象模型， 根据我们上面创建的数据模型，创建出托管对象模型，（类似于加工厂的概念，能够用来生产对象的模子）
3.NSPersistentStoreCoordinator - 持久化存储协调器，包含数据库的名称、存储数据类型（Sqlite、Xml、内存）、位置等信息
4.NSPersistentStore - 持久化存储区

另外再附一张图来说明这几者的依赖关系
![](http://ock9zbzms.bkt.clouddn.com/20151030232125923.png)

由于CoreData管理数据的过程较为通用，个人觉的还是封装成一个管理对象较好，方便以后代码复用，这里创建一个CoreDataHelper的类，专门用来管理数据对象，该类的头文件如下

```objc
#import <Foundation/Foundation.h>  
#import <CoreData/CoreData.h>  
@interface CoreDataHelper : NSObject  
@property (nonatomic, strong) NSManagedObjectContext       *context;//托管对象上下文  
@property (nonatomic, strong) NSManagedObjectModel         *model;//托管对象模型  
@property (nonatomic, strong) NSPersistentStoreCoordinator *coordinate;//持久化存储协调器  
@property (nonatomic, strong) NSPersistentStore            *store;//持久化存储区  
- (id)init;             //初始化  
- (void)loadStore;      //加载cordite  
- (void)setupCoreData;  //设置cordite相关信息  
- (void)saveContext;    //保存context  
```
CoreDataHelp的实现文件如下

```objc
#import "CoreDataHelper.h"  
static NSString *storeFileName = @"demo.sqlite";    //测试数据库  
  
@implementation CoreDataHelper  
  
#pragma mark - PATHS  
  
- (NSString *)applicationDocumentDirectory  
{  
    if (debug) {  
        NSLog(@"Running %@ '%@'",[self class],NSStringFromSelector(_cmd));  
    }  
      
    NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);  
    return [paths lastObject];  
}  
  
- (NSURL *)applicationStoreDirectory  
{  
    if (debug) {  
        NSLog(@"Running %@ '%@'",[self class],NSStringFromSelector(_cmd));  
    }  
      
    NSURL *url = [[NSURL fileURLWithPath:[self applicationDocumentDirectory]] URLByAppendingPathComponent:@"stores"];  
    if (![[NSFileManager defaultManager]fileExistsAtPath:[url path]]) {  
        NSError *error = nil;  
        BOOL success   = [[NSFileManager defaultManager]createDirectoryAtURL:url  
                                               withIntermediateDirectories:YES  
                                                                attributes:nil  
                                                                     error:&error];  
        if (success) {  
            if (debug) {  
                NSLog(@"success create directory!");  
            }  
        }else{  
            NSLog(@"failed create directory!");  
        }  
    }  
      
    return url;  
}  
  
- (NSURL *)storeUrl  
{  
    NSURL *storeUrl = [[self applicationStoreDirectory]URLByAppendingPathComponent:storeFileName];  
    NSLog(@"storeurl = %@",storeUrl);  
    return storeUrl;  
}  
  
#pragma mark - SETUP  
- (id)init  
{  
    if (self) {  
        _model      = [NSManagedObjectModel mergedModelFromBundles:nil];  
        _coordinate = [[NSPersistentStoreCoordinator alloc]initWithManagedObjectModel:_model];  
        _context    = [[NSManagedObjectContext alloc]initWithConcurrencyType:NSMainQueueConcurrencyType];  
        [_context setMergePolicy:NSMergeByPropertyObjectTrumpMergePolicy];  
        [_context setPersistentStoreCoordinator:_coordinate];  
          
        _importContext = [[NSManagedObjectContext alloc]initWithConcurrencyType:NSPrivateQueueConcurrencyType];  
          
        [_importContext performBlock:^{  
            [_importContext setPersistentStoreCoordinator:_coordinate];  
            [_importContext setMergePolicy:NSMergeByPropertyObjectTrumpMergePolicy];  
            [_importContext setUndoManager:nil];  
        }];  
    }  
    return self;  
}  
  
- (void)loadStore  
{  
    if (debug) {  
        NSLog(@"Running %@ ,'%@'",[self class], NSStringFromSelector(_cmd));  
    }  
      
    if (_store) {  
        return;  
    }  
  
    NSError *error;  
          
        //NSMigratePersistentStoresAutomaticallyOption coreData尝试将低版本的数据模型向高版本进行迁移  
        //NSInferMappingModelAutomaticallyOption    coredata会自动创建迁移模型，会去自动尝试  
        NSDictionary *option = @{NSMigratePersistentStoresAutomaticallyOption:@(YES),  
                                 NSInferMappingModelAutomaticallyOption:@(YES),  
                                 NSSQLitePragmasOption:@{@"journal_mode":@"DELETE"}};  
          
        _store = [_coordinate addPersistentStoreWithType:NSSQLiteStoreType  
                                           configuration:nil  
                                                     URL:[self storeUrl]  
                                                 options:option  
                                                   error:&error];  
        if (!_store) {  
            if (debug) {  
                NSLog(@"failed load store,error = %@",error);  
                abort();  
            }  
        }  
        else/**/{  
            NSLog(@"successfully add store : %@",_store);  
        }  
}  
  
- (void)setupCoreData  
{  
    [self loadStore];  
}  
  
- (void)saveContext  
{  
    if ([_context hasChanges]) {  
        NSError *error = nil;  
        if ([_context save:&error]) {  
            NSLog(@"context save successfully");  
        }else{  
            NSLog(@"failed save %@",error);  
        }  
    }else{  
        NSLog(@"skipped context save , there is no changes");  
    }  
}  
  
@end  
```
[2.4 最后附上查询和保存数据库的代码](#2)
在AppDelegate.m文件里写一个方法，用来初始化CoreData数据库

```objc
- (CoreDataHelper *)cdh  
{  
    if (!_cdh) {  
        static dispatch_once_t onceToken;  
        dispatch_once(&onceToken, ^{  
            _cdh = [CoreDataHelper new];  
              
        });  
        [_cdh setupCoreData];  
    }  
    return _cdh;  
}  
```

下面是插入数据，查询、保存数据的方法
```objc
- (void)demo  
{  
    Item *bananer = [NSEntityDescription insertNewObjectForEntityForName:@"Item" inManagedObjectContext:[[self cdh] context]];  
    bananer.unit = kg;  
    bananer.name = @"bananer";  
      
    Item *oranger = [NSEntityDescription insertNewObjectForEntityForName:@"Item" inManagedObjectContext:[[self cdh] context]];  
    oranger.unit = kg;  
    oranger.name = @"Oranger";  
      
    NSFetchRequest *request = [NSFetchRequest fetchRequestWithEntityName:@"Item"];  
    NSArray *result = [[[self cdh]context] executeFetchRequest:request error:nil];  
    for (Item *item in result) {  
        if (debug) {  
            NSLog(@"item.name = %@",item.name);  
        }  
    }  
    [[self cdh]saveContext];  
} 
```
[2.5 小结](#2)
经过这么一番下来，终于将CoreData技术应用到我们的项目中了，我们现在能做到，把数据插入到数据库、也能从数据库中读取出数据来，也能保存数据。但是要注意这才是刚刚开始，接下来还有更多的coredata问题等着我们，比方说下节要介绍的数据迁移问题。
