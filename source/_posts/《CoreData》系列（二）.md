---
title: 《CoreData》系列（二）
date: 2016-08-27 19:58:16
tags: coredata学习
---

CoreData数据迁移以及版本升级

<!---more--->

[1 概述](#1)

为什么要有数据迁移？
由于CoreData可视化的特殊性，那么当数据模型发生变化时，相应的sqlite数据库的表由于不知道model发生了变化，表结构必须相应的做出调整，否则会导致程序Crash，CoreData的解决方案是通过创建新的sqlite表，然后将旧的数据迁移到新表上得方案来处理。下面分别介绍三种数据迁移的方式，并详细说明三种迁移方式的应用场景和注意事项。

1.轻量级的数据迁移方式
2.默认的迁移方式
3.使用迁移管理器

[1.1 轻量级的数据迁移方式](#2)

轻量级的数据迁移，也就是说，并不需要程序员做很多事情就可以完成数据的迁移，是由系统默认进行的数据迁移。
那么如何进行轻量级的数据迁移呢，当model的表字段发生变化，且应用程序已经发布过版本时，此时千万不能单单修改原model来达到修改model的目的，如果这样做的话，程序会crash。正确的做法是，

1.新建一个model，并将model命名为model2，并将model2设置为当前model。
2.修改NSPersistentStoreCoordinator加载缓存区的配置。具体如下

```objc
NSDictionary *option = @{NSMigratePersistentStoresAutomaticallyOption:@(YES),  
                         NSInferMappingModelAutomaticallyOption:@(YES),  
                         };  
  
_store = [_coordinate addPersistentStoreWithType:NSSQLiteStoreType  
                                   configuration:nil  
                                             URL:[self storeUrl]  
                                         options:option  
                                           error:&error]; 
```
 tips：使用iCloud开发程序的app，只能使用这种迁移方式。

 [1.2 默认的迁移方式](#2)

 正常情况下，使用轻量级的数据迁移已经足够了，但是如果由于开发需要，需要将某个Entity下面的某个Attribute迁移到新的Entity下的某个Attribute，那么轻量级的迁移方式就不能够满足需求，这个时候就需要使用默认的迁移方式来进行数据迁移。这里以一个例子代码来详细阐述如何进行默认的迁移

 ![](http://ock9zbzms.bkt.clouddn.com/20151106182444884.jpg)

  现在要将Model2里面的Measurement下面的name迁移到Account里面的下面的xyz属性下。
1.根据model2来创建一个新model，并命名为model3，然后将model3设置为currentmodel。
2.添加新的entity，并命名为Account，添加attribute xyz。
3.删除model2里面的Measurement，根据model3创建NSManagerObect的子类Account。
4.以model2为soureModel，model3为destinationModel添加一个MappingModel
5.按照下图所示设置映射model即可
6.最后记得将NSInferMappingModelAutomaticallyOption设置为Yes（coredata会优先读取映射model，如果没有就会自己推断），至此，默认的迁移方式就算是搞定了。

![](http://ock9zbzms.bkt.clouddn.com/20151106183204704.jpg)
![](http://ock9zbzms.bkt.clouddn.com/20151106183355504.jpg)

[1.3 迁移管理器](#2)

简单概述下何为迁移管理器，迁移管理器，就是不再使用系统的NSPersistentCoordinator进行数据迁移，而是使用NSMigrationManager进行数据缓存区的迁移。并配合一个数据迁移视图控制器提供优雅的迁移等待界面。等待界面如下，是不是感觉很丑呢，哈哈。那么使用迁移管理器的好处又是什么呢？可以实现更加精细化的数据操作，此外还能向用户报告迁移进度。有这俩点，还不够我们去研究下它么?Let's go!

![](http://ock9zbzms.bkt.clouddn.com/coredata20151106184607971.jpg)

准备工作
	何时启用迁移管理器，即迁移的时机？
	迁移工作如何进行？
	迁移完成如何善后？

下面对上面的问题一一来做解答
迁移的时机，迁移工作需要在载入数据库的时候进行，即上节所讲的 loadStore：的时候进行，但是呢？还需要做一些判断工作。具体代码如下

```objc
- (void)loadStore  
{  
    if (debug) {  
        NSLog(@"Running %@ ,'%@'",[self class], NSStringFromSelector(_cmd));  
    }  
      
    if (_store) {  
        return;  
    }  
    
    BOOL useMigrateManager = MigrationMode;  
  
    if (useMigrateManager && [self isMigrationNecessaryForStore:[self storeUrl]]) {  
        [self performBackgroundManagedMigrationForStore:[self storeUrl]];  
    }else{  
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
}  
```

其中有开关，用来控制是否使用迁移管理器，以及系统是否需要进行迁移的判断。系统是否需要迁移的判断代码如下

```objc
- (BOOL)isMigrationNecessaryForStore:(NSURL *)storeUrl  
{  
    if (debug) {  
        NSLog(@"Running %@ '%@'",[self class],NSStringFromSelector(_cmd));  
    }  
      
    //文件是否存在，如果不存在认为是用户设备上并没有持久化存储区，自然不需要迁移  
    if (![[NSFileManager defaultManager]fileExistsAtPath:[self storeUrl].path isDirectory:nil]) {  
        if (debug) {  
            NSLog(@"Skipped Migration, source database missing");  
        }  
        return NO;  
    }  
      
     NSError *error                         = nil;  
     NSDictionary *sourceMetaData           = [NSPersistentStoreCoordinator metadataForPersistentStoreOfType:NSSQLiteStoreType  
                                                                                                     URL:storeUrl  
                                                                                                      error:&error];
    NSManagedObjectModel *destinationModel = _coordinate.managedObjectModel;  
      
    //比较当前对象模型是否与用户之前安装的应用持久化存储区是否兼容。如果兼容，不需要迁移  
    if ([destinationModel isConfiguration:nil compatibleWithStoreMetadata:sourceMetaData]) {  
        if (debug) {  
            NSLog(@"Skipped Migration, source database is already compatible");  
            return NO;  
        }  
    }  
       
    //所有情况都尝试了，发现还是需要进行数据迁移  
    return YES;  
} 
```

迁移工作如何进行，众所周知，迁移工作是一项比较耗时间的工作，尤其是在数据库比较大的情况下，那么肯定不能放在前台进行,必须放在后台进行，前台展示加载进度，代码如下

```objc
- (void)performBackgroundManagedMigrationForStore:(NSURL *)store  
{  
    if (debug) {  
        NSLog(@"Running %@ '%@'",[self class],NSStringFromSelector(_cmd));  
    }  
      
    UIStoryboard *sb                      = [UIStoryboard storyboardWithName:@"Main" bundle:nil];  
    self.migrationVC                      = [sb instantiateViewControllerWithIdentifier:@"migration"];  
  
    UIApplication *app                    = [UIApplication sharedApplication];  
    UINavigationController *navigationCtl = (UINavigationController *)[app keyWindow].rootViewController;  
      
    [navigationCtl presentViewController:self.migrationVC  
                                animated:YES  
                              completion:nil];  
      
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0), ^{  
         
        BOOL done = [self migrateStore:[self storeUrl]];  
        if (done) {  
            dispatch_async(dispatch_get_main_queue(), ^{  
                NSError *error              = nil;  
  
                NSDictionary *configuration = @{NSMigratePersistentStoresAutomaticallyOption:@(YES),  
                                                NSInferMappingModelAutomaticallyOption:@(YES),  
                                                NSSQLitePragmasOption:@{@"journal_mode":@"DELETE"}};  
  
                _store                      = [_coordinate addPersistentStoreWithType:NSSQLiteStoreType  
                                                   configuration:nil  
                                                             URL:[self storeUrl]  
                                                         options:configuration  
                                                           error:&error];  
                if (_store) {  
                    if (debug) {  
                        NSLog(@"success create store");  
                    }  
                }else {  
                    if (debug) {  
                        NSLog(@"failed, error = %@",error);  
                    }  
                    abort();  
                }  
                  
                [self.migrationVC dismissViewControllerAnimated:YES  
                                                     completion:nil];  
                  
                self.migrationVC = nil;  
            });  
        }  
          
    });  
}
```

接下来是，真正的迁移过程
```objc
- (BOOL)migrateStore:(NSURL *)store  
{  
    if (debug) {  
        NSLog(@"Running %@ '%@'",[self class],NSStringFromSelector(_cmd));  
    }  
      
      
    NSDictionary *sourceMeta               = [NSPersistentStoreCoordinator metadataForPersistentStoreOfType:NSSQLiteStoreType  
                                                                                                        URL:store  
                                                                                                      error:nil];  
  
    NSManagedObjectModel *sourceModel      = [NSManagedObjectModel mergedModelFromBundles:nil  
                                                                         forStoreMetadata:sourceMeta];  
  
    NSManagedObjectModel *destinationModel = _model;  
    NSMappingModel *mappingModel           = [NSMappingModel mappingModelFromBundles:nil  
                                                                      forSourceModel:sourceModel  
                                                                    destinationModel:destinationModel];  
    if (mappingModel) {  
        NSError *error                       = nil;  
  
        NSMigrationManager *migrationManager = [[NSMigrationManager alloc]initWithSourceModel:sourceModel  
                                                                             destinationModel:destinationModel];  
  
        [migrationManager addObserver:self  
                           forKeyPath:@"migrationProgress"  
                              options:NSKeyValueObservingOptionNew  
                              context:nil];  
  
        NSURL *destinationStore              = [[self applicationStoreDirectory]URLByAppendingPathComponent:@"temp.sqlite"];  
        BOOL success                         = NO;  
        success                              = [migrationManager migrateStoreFromURL:store  
                                                    type:NSSQLiteStoreType  
                                                 options:nil  
                                        withMappingModel:mappingModel  
                                        toDestinationURL:destinationStore  
                                         destinationType:NSSQLiteStoreType  
                                      destinationOptions:nil  
                                                   error:&error];  
        if (success) {  
            if (debug) {  
                NSLog(@"Migration Successfully!");  
            }  
            if ([self replaceStore:store withStore:destinationStore]) {  
                [migrationManager removeObserver:self forKeyPath:@"migrationProgress" context:NULL];  
                [[NSNotificationCenter defaultCenter]postNotificationName:someThingChangedNotification object:nil];  
            }  
        }else{  
            if (debug) {  
                NSLog(@"Migration Failed");  
            }  
        }  
    }else{  
        if (debug) {  
            NSLog(@"Mapping model is NULL");  
        }  
    }  
    return YES;  
}
```
最后附上俩个辅助方法，用来观察迁移过程和替换数据库的
```objc
- (BOOL)replaceStore:(NSURL *)old withStore:(NSURL *)new  
{  
    BOOL success   = NO;  
    NSError *error = nil;  
    if ([[NSFileManager defaultManager]removeItemAtURL:old error:&error]) {  
        error = nil;  
        if ([[NSFileManager defaultManager]moveItemAtURL:new toURL:old error:&error]) {  
            success = YES;  
        }else {  
            if (debug) {  
                NSLog(@"failed move new store to old");  
            }  
        }  
    }else{  
        if (debug) {  
            NSLog(@"failed remove old store");  
        }  
    }  
    return success;  
} 
```
```objc
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(voidvoid *)context  
{  
    if ([keyPath isEqualToString:@"migrationProgress"]) {  
        dispatch_async(dispatch_get_main_queue(), ^{  
            float progress                         = [[change objectForKey:NSKeyValueChangeNewKey]floatValue];  
            self.migrationVC.progressView.progress = progress;  
  
            int percenttage                        = progress * 100;  
            NSString *string                       = [NSString stringWithFormat:@"Migration Progress %i%%",percenttage];  
            self.migrationVC.progressLabel.text    = string;  
        });  
    }  
}  
```
至此，三种数据迁移的方式，都已叙述完毕。
[2 小结]（#1）
三种迁移方式，各有各的好处，轻量级的迁移可以配套icloud实现云端存储，默认的数据迁移，支持将属性级别的数据进行任意迁移。迁移管理器，可以管理文件存储路径，并能够报告迁移进度，我们在开发过程中，应该按照自己的需求合理选择迁移方式，下一小节结合NSFetchedResultController进行数据的实际应用。