---
title: 点点滴滴：NSScanner的用法小结
date: 2016-08-27 20:36:20
tags: 点点滴滴
toc: true
---

NSScanner类是一个类簇的抽象父类,该类簇为一个从NSString对象扫描值的对象提供了程序接口。
<!--more-->
NSScanner对象把NSString 对象的的字符解释和转化成 number和string 类型的值。在创建NSScanner对象的时候为它分配字符(string )，当你从NSScanner对象获取内容的时候，它会从头到尾遍历字符串(string)。
由于类簇的属性， scanner对象并不是 NSScanner类的实例，而是它一个私有子类的实例。尽管scanner对象的类是私有的，但是它的接口是公开的（抽象父类已经声明）。 NSScanner 的原始方法是string和Configuring a Scanner方法下面列举的所有的方法。
在 NSScanner 对象扫描字符串的时候，你可以通过设置属性charactersToBeSkipped忽略某些字符。在扫描字符串之前，那些位于忽略字符集中的字符将会被跳过。默认的忽略字符是空格和回车字符。
可以通过[[scanner string] substringFromIndex:[scanner scanLocation]]获取未扫描的字符串。

#### [创建 Scanner对象](#1)
```objc
+ scannerWithString:
+ localizedScannerWithString:
- initWithString: Designated Initializer
```
scannerWithString，返回值是 扫描过aString字符串的NSScanner 对象，该方法通过调用initWithString设置扫描字符串;
localizedScannerWithString,返回值是 通过用户默认的 locale方式扫描字符串的NSScanner 对象，该方法也是通过调用initWithString设置扫描字符串;
initWithString，返回值是NSScanner 对象，该对象通过扫描aString完成初始化

#### [获取Scanner对象字符串](#1)
```objc
string Property
```

#### [设置scanner对象](#1)
```objc
scanLocation Property
caseSensitive Property
charactersToBeSkipped Property
locale Property

scanLocation,下次扫描开始的位置，如果该值超出了string的区域，将会引起NSRangeException,该属性在发生错误后重新扫描时非常有用。
caseSensitive，是否区分字符串中大小写的标志。默认为NO，注意：该设置不会应用到被跳过的字符集。
charactersToBeSkipped,在扫描时被跳过的字符集，默认是空白格和回车键。被跳过的字符集优先于扫描的字符集：例如一个scanner被跳过的字符集为空格，通过scanInt:去查找字符串中的整型数时，首先做的不是扫描，而是跳过空格，直到找到十进制数据或者其他的字符。在字符被扫描的时候，跳过功能就失效了。如果你扫描的字符和跳过的字符是一样的，结果将是未知的。被跳过的字符是一个唯一值，scanner不会将忽略大小写的功能应用于它，也不会用这些字符做一些组合，如果在扫描字符换的时候你想忽略全部的元音字符，就要这么做（比如：将字符集设置成“AEIOUaeiou”};
locale,scanner 的locale对它从字符串中区分数值产生影响，它通过locale的十进制分隔符区分浮点型数据的整数和小数部分。一个没有locale的scanner用非定域值。新的scanner若没有设置locale，使用默认locale。
```

#### [扫描字符串](#1)
```objc
- scanCharactersFromSet:intoString:
- scanUpToCharactersFromSet:intoString:
- scanDecimal:
- scanDouble:
- scanFloat:
- scanHexDouble:
- scanHexFloat:
- scanHexInt:
- scanHexLongLong:
- scanInteger:
- scanInt:
- scanLongLong:
- scanString:intoString:
- scanUnsignedLongLong:
- scanUpToString:intoString:
atEnd Property

scanCharactersFromSet:intoString:扫描字符串中和NSCharacterSet字符集中匹配的字符，是按字符单个匹配的，例如，NSCharacterSet字符集为@"test123Dmo"，scanner字符串为 @" 123test12Demotest"，那么字符串中所有的字符都在字符集中，所以指针指向的地址存储的内容为"123test12Demotest"
scanUpToCharactersFromSet:intoString：扫描字符串直到遇到NSCharacterSet字符集的字符时停止，指针指向的地址存储的内容为遇到跳过字符集字符之前的内容
scanString:intoString:从当前的扫描位置开始扫描，判断扫描字符串是否从当前位置能扫描到和传入字符串相同的一串字符，如果能扫描到就返回YES,指针指向的地址存储的就是这段字符串的内容。例如scanner的string内容为123abc678,传入的字符串内容为abc，如果当前的扫描位置为0，那么扫描不到，但是如果将扫描位置设置成3，就可以扫描到了。
scanUpToString:intoString:从当前的扫描位置开始扫描，扫描到和传入的字符串相同字符串时，停止，指针指向的地址存储的是遇到传入字符串之前的内容。例如scanner的string内容为123abc678,传入的字符串内容为abc，存储的内容为123
scanDecimal:扫描NSDecimal类型的值，有关NSDecimal类型的值更多的信息可以查看：NSDecimalNumber
scanDouble :扫描双精度浮点型字符，溢出和非溢出都被认为合法的浮点型数据。在溢出的情况下scanner将会跳过所有的数字，所以新的扫描位置将会在整个浮点型数据的后面。double指针指向的地址存储的数据为扫描出的值，包括溢出时的HUGE_VAL或者 –HUGE_VAL，即未溢出时的0.0。
scanFloat：扫描单精度浮点型字符，具体内容同scanDouble
scanHexDouble: 扫描双精度的十六进制类型，溢出和非溢出都被认为合法的浮点型数据。在溢出的情况下scanner将会跳过所有的数字，所以新的扫描位置将会在整个浮点型数据的后面。double指针指向的地址存储的数据为扫描出的值，包括溢出时的HUGE_VAL或者 –HUGE_VAL，即未溢出时的0.0。数据接收时对应的格式为 %a 或%A ，双精度十六进制字符前面一定要加  0x或者 0X。
scanHexInt 扫描十六进制无符整型，unsigned int指针指向的地址值为 扫描到的值，包含溢出时的UINT_MAX。
scanHexLongLong 同scanHexDouble
scanInt 扫描整型，溢出也被认为是有效的整型，int 指针指向的地址的值为扫描到的值，包含溢出时的INT_MAX或INT_MIN。
scanInteger 同scanInt
scanLongLong 扫描LongLong 型，溢出也被认为是有效的整型，LongLong指针指向的地址的值为扫描到的值，包含溢出时的LLONG_MAX 或 LLONG_MIN。
```

### [如何使用](#1)
#### [场景1：获取URL中的请求key和value。](#2)

我们知道在iOS中，系统封装的URL会有一个属性叫做Query，如果是一个正确的URL会有类似的格式 
@"https://wwww.baidu.com/home.php?name=cheng&age=19&nickname=hehe"

其中？后面的就是URL的Query，那么如何分别获取key和value呢？

```objc
void scannerStringDemo(){
    
    NSURL *url = [NSURL URLWithString:@"https://wwww.baidu.com/home.php?name=cheng&age=19&nickname=hehe"];
    NSScanner *scanner = [[NSScanner alloc]initWithString:url.query];
    NSCharacterSet *delimiterSet = [NSCharacterSet characterSetWithCharactersInString:@"&"];
    NSMutableDictionary *paris = [@{} mutableCopy];
    
    while (![scanner isAtEnd]) {
        NSString *pairString = nil;
        [scanner scanUpToCharactersFromSet:delimiterSet intoString:&pairString];
        [scanner scanCharactersFromSet:delimiterSet intoString:NULL];
        NSRange range = [pairString rangeOfString:@"="];
        if (range.location != NSNotFound) {
            NSString *key = [pairString substringToIndex:range.location];
            NSString *value = [pairString substringFromIndex:range.location + range.length];
            [paris setObject:key forKey:@"key"];
            [paris setObject:value forKey:@"value"];
        }
    }
}

```
#### [场景2：判断当前的字符串是否是一个纯数字](#2)

```objc
- (BOOL)isPureInt:(NSString*)string{
    NSScanner* scan = [NSScanner scannerWithString:string];
    int val;
    return[scan scanInt:&val] && [scan isAtEnd];
}
```

#### [场景3：判断TextField输入的字符串是不是需要的格式，这里要求是最多三位小数的数字](#2)
```objc
+(BOOL)isValidAboutInputText:(UITextField *)textField shouldChangeCharactersInRange:(NSRange)range replacementString:(NSString *)string decimalNumber:(NSInteger)number{
    NSScanner      *scanner    = [NSScanner scannerWithString:string];
    NSCharacterSet *numbers;
    NSRange         pointRange = [textField.text rangeOfString:@"."];
    if ( (pointRange.length > 0) && (pointRange.location < range.location  || pointRange.location > range.location + range.length) ){
        numbers = [NSCharacterSet characterSetWithCharactersInString:@"0123456789"];
    }else{
        numbers = [NSCharacterSet characterSetWithCharactersInString:@"0123456789."];
    }
    if ( [textField.text isEqualToString:@""] && [string isEqualToString:@"."] ){
        return NO;
    }
    short remain = number; //保留 number位小数
    NSString *tempStr = [textField.text stringByAppendingString:string];
    NSUInteger strlen = [tempStr length];
    if(pointRange.length > 0 && pointRange.location > 0){ //判断输入框内是否含有“.”。
        if([string isEqualToString:@"."]){ //当输入框内已经含有“.”时，如果再输入“.”则被视为无效。
            return NO;
        }
        if(strlen > 0 && (strlen - pointRange.location) > remain+1){ //当输入框内已经含有“.”，当字符串长度减去小数点前面的字符串长度大于需要要保留的小数点位数，则视当次输入无效。
            return NO;
        }
    }
    NSRange zeroRange = [textField.text rangeOfString:@"0"];
    if(zeroRange.length == 1 && zeroRange.location == 0){ //判断输入框第一个字符是否为“0”
        if(![string isEqualToString:@"0"] && ![string isEqualToString:@"."] && [textField.text length] == 1){ //当输入框只有一个字符并且字符为“0”时，再输入不为“0”或者“.”的字符时，则将此输入替换输入框的这唯一字符。
            textField.text = string;
            return NO;
        }else{
            if(pointRange.length == 0 && pointRange.location > 0){ //当输入框第一个字符为“0”时，并且没有“.”字符时，如果当此输入的字符为“0”，则视当此输入无效。
                if([string isEqualToString:@"0"]){
                    return NO;
                }
            }
        }
    }
    NSString *buffer;
    if ( ![scanner scanCharactersFromSet:numbers intoString:&buffer] && ([string length] != 0) ){
        return NO;
    }else{
        return YES;
    }
}
```

### [注意事项](#2)
```objc
[scanner scanCharactersFromSet:delimiterSet intoString:NULL];可以使用这个方法，跳过不需要的字符。
```
目前能想到的用法就这些，如果有更多的用法，以后慢慢补充！