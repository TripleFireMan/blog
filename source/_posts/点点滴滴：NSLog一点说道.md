---
title: 点点滴滴：NSLog一点说道
date: 2016-08-27 21:03:09
tags: 点点滴滴
---

今天的小结主要讲述在IOS开发中实现自定义的NSLog方法
<!--more-->
#### [1 为什么要对NSLog进行重定义？](#1)

在开始这个知识点的讲解之前，首先说下，为什么要对NSLog宏进行重定义。在项目开发中，经常需要对程序进行调试。由于调试分布在项目的各种地方，当项目发布时，如果再将调试信息去掉，显示会消耗很大的人力，物力。幸好，强大的xcode给我们提供了一个非常方便的功能。在项目的Build Settings中给Apple LLVM Preprocessing中的 preprocessor macros下面的Debug添加一个调试宏DEBUG=1,记住在Release下面不要添加任何东西！添加这个东西的意思就是告诉编译器，在调试阶段，项目中进行了一个DEBUG的宏定义，但是Release阶段不定义。

#### [2 如何对NSLog宏进行定义了](#1)

```objc
   #ifdef DEBUG
   #define NSLog(args...)  ExtendNSLog(__FILE__,__LINE__,__PRETTY_FUNCTION__,args);
   #else
   #define NSLog(x...)
   #endif
```

下面来对上述宏进行解释，如果定义了DEBUG宏，那么就对NSLog(args...)进行重定义，如果没有定义，将NSLog(args...)设置为空，不做任何处理，

#### [3 如何对定义的信息进行输出，并附带.文件名，打印行数，方法名.接下来对ExtendNSLog()进行解释](#1)

```objc
 void ExtendNSLog(const char *file, int lineNumber, const char *functionName, NSString *format, ...)
{
    // Type to hold information about variable arguments.
    va_list ap;
    // Initialize a variable argument list.
    va_start (ap, format);
    // NSLog only adds a newline to the end of the NSLog format if
    // one is not already there.
    // Here we are utilizing this feature of NSLog()
    if (![format hasSuffix: @"\n"])
    {
        format = [format stringByAppendingString: @"\n\n⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯⎯\n\n"];
    }
    NSString *body = [[NSString alloc] initWithFormat:format arguments:ap];
    // End using variable argument list.
    va_end (ap);
    NSString *fileName = [[NSString stringWithUTF8String:file] lastPathComponent];
    fprintf(stderr, "[%s LINE:%d]%s:\n%s",
            [fileName UTF8String],lineNumber,
            (functionName[0])=='-'?(&functionName[1]):functionName,
            [body UTF8String]);
 }
```

下面来对上述代码进行解释
1. 获取参数列表类
2. 启动参数列表类和格式化字符串的关联
3. 获取格式化字符串的实际输出文本
4. 关闭参数列表类和格式化字符串的关联
5. 调用C函数fprintf(),将打印信息输出。