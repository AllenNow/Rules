# iOS-CodeObfuscation
APP加固之代码混淆

<p id="目录"></p>
## 目录
* [一、为什么要加固](#为什么要加固)
* [二、加固的几个方面](#加固的几个方面)
* [三、目前的加固方法](#目前的加固方法)
* [四、已使用加固的项目](#已使用加固的项目)
* [五、其他](#其他)


<p id="为什么要加固"></p>
## 一、为什么要加固
> [< 返回目录](#目录)

裸奔app的安全隐患
一部越狱的iOS设备，外加上述的逆向工具，给裸奔的iOS应用程序带来哪些威胁呢?

>
```　　
　　任意读写文件系统数据
　　HTTP(S)实时被监测
　　重新打包ipa
　　暴露的函数符号
　　未加密的静态字符
　　篡改程序逻辑控制流
　　拦截系统框架API
　　逆向加密逻辑
　　跟踪函数调用过程(objc_msgSend)
　　可见视图的具体实现
　　伪造设备标识
　　可用的URL schemes
　　runtime任意方法调用
　　……
```


<p id="加固的几个方面"></p>
## 二、加固的几个方面
> [< 返回目录](#目录)

现在的加固工具总的来说都是从以下几个方面来做的：
> 
一、字符串加密：
> 
现状：对于字符串来说，程序里面的明文字符串给静态分析提供了极大的帮助，比如说根据界面特殊字符串提示信息，从而定义到程序代码块，或者获取程序使用的一些网络接口等等。
> 
加固：对程序中使用到字符串的地方，首先获取到使用到的字符串，当然要注意哪些是能加密，哪些不能加密的，然后对字符串进行加密，并保存加密后的数据，再在使用字符串的地方插入解密算法，这样就很好的保护了明文字符串。
> 
二、类名方法名混淆
> 
现状：目前市面上的IOS应用基本上是没有使用类名方法名混淆的，所以只要我们使用class-dump把应用的类和方法定义dump下来，然后根据方法名就能够判断很多程序的处理函数是在哪。从而进行hook等操作。
> 
加固：对于程序中的类名方法名，自己产生一个随机的字符串来替换这些定义的类名和方法名，但是不是所有类名，方法名都能替换的，要过滤到系统有关的函数以及类，可以参考下开源项目：https://github.com/Polidea/ios-class-guard
> 
三、程序代码混淆
> 
现状：目前的IOS应用找到可执行文件然后拖到Hopper Disassembler或者IDA里面程序的逻辑基本一目了然。
> 
加固：可以基于Xcode使用的编译器clang，然后在中间层也就是IR实现自己的一些混淆处理，比如加入一些无用的逻辑块啊，代码块啊，以及加入各种跳转但是又不影响程序原有的逻辑。可以参考下开源项目：https://github.com/obfuscator-llvm/obfuscator/  当然开源项目中也是存在一些问题的，还需自己再去做一些优化工作。
> 
四、加入安全SDK
> 
现状：目前大多数IOS应用对于简单的反调试功能都没有，更别说注入检测，以及其它的一些检测了。
> 
加固：加入SDK，包括多处调试检测，注入检测，越狱检测，关键代码加密，防篡改等等功能。并提供接口给开发者处理检测结果。 
> 
当然除了这些外，还有很多方面可以做加固保护的，相信大家会慢慢增加对IOS应用安全的意识，保护好自己的APP。

<p id="目前的加固方法"></p>
## 三、目前的加固方法
> [< 返回目录](#目录)

#### 1、操作方法的总结：

```
①、添加PrefixHeader.h，正确设置路径
②、添加`codeObfuscation.h`(将来存放混淆内容)，并将其`#import`进PrefixHeader.h

含特征量的方法名混淆过程：
③、为方法名添加特征量，如`lcof_`前缀
④、添加`混淆脚本confuse.sh`文件和`需混淆的函数名类名列表func.list`文件。(注意：最后不要将.sh打包进项目。)
⑤、配置`Build Phase`，新增`Run Script`，设置脚本路径或者脚本内容
⑥、编译执行脚本。（这时候你会发现原本为空的`需混淆的函数名类名列表func.list`文件及我们最后需要使用的`codeObfuscation.h`文件中有内容生成了。同时该脚本还为你生成了一个你`SYMBOL_DB_FILE`所指定的文件。）
至此含特征量的方法名已混淆完成

不含特征量的方法名、类名、回调的混淆过程：
⑦、修改`混淆脚本confuse.sh`，注释掉其中的`grep`部分，同时在`需混淆的函数名类名列表func.list`文件中添加要补充混淆的方法名、类名、回调名等。
⑧、编译执行脚本，更新`codeObfuscation.h`文件中的内容。

其他补充：
⑨、增加混淆'原本不需要混淆的类'，用于混淆'所混淆的类'是重要类

善后：
⑩、可删除多余文件。如`混淆脚本confuse.sh`、`需混淆的函数名类名列表func.list`、SYMBOL_DB_FILE。
```
#### 2、操作整个过程中的几个易错注意点：
* `PrefixHeader.h`中未`#import "codeObfuscation.h"`。
* `#import "codeObfuscation.h"`未防止在`PrefixHeader.h`最前面，导致可能有些混淆类的在该import前就import了，而编译不通过。
* sh脚本不应打包进项目。
* 容易引起app崩溃的注意：使用如interface链接的事件，即使该事件已被我们宏混淆，虽然在classdump和ida上是混淆的，但是在我们使用app的时候发现触发该事件会产生崩溃。
* 容易引起app没法进入指定页面效果的注意：通过nibName从xib中生成的控制器，即使我们已将该控制器的类名进行了混淆，但是在最后使用的时候发现无法正常加载xib上的界面元素。
* 不要试图去简单混淆静态库暴露在外面的接口方法。

#### 3、脚本内容如下：

```
#!/usr/bin/env bash
# $PROJECT_DIR/$PROJECT_NAME/Resource/confuse.sh

TABLENAME=symbols
SYMBOL_DB_FILE="symbols"
STRING_SYMBOL_FILE="$PROJECT_DIR/LuckyClient/Resource/func.list"

CONFUSE_FILE="$PROJECT_DIR/LuckyClient"

HEAD_FILE="$PROJECT_DIR/LuckyClient/Resource/codeObfuscation.h"

export LC_CTYPE=C

#取以.m或.h结尾的文件以+号或-号开头的行 |去掉所有+号或－号|用空格代替符号|n个空格跟着<号 替换成 <号|开头不能是IBAction|用空格split字串取第二部分|排序|去重复|删除空行|删掉以init开头的行>写进func.list
grep -h -r -I  "^[-+]" $CONFUSE_FILE  --include '*.[mh]' |sed "s/[+-]//g"|sed "s/[();,: *\^\/\{]/ /g"|sed "s/[ ]*</</"| sed "/^[ ]*IBAction/d"|awk '{split($0,b," "); print b[2]; }'| sort|uniq |sed "/^$/d"|sed -n "/^lcof_/p" >$STRING_SYMBOL_FILE


#维护数据库方便日后作排重,一下代码来自念茜的微博
createTable()
{
echo "create table $TABLENAME(src text, des text);" | sqlite3 $SYMBOL_DB_FILE
}

insertValue()
{
echo "insert into $TABLENAME values('$1' ,'$2');" | sqlite3 $SYMBOL_DB_FILE
}

query()
{
echo "select * from $TABLENAME where src='$1';" | sqlite3 $SYMBOL_DB_FILE
}

ramdomString()
{
openssl rand -base64 64 | tr -cd 'a-zA-Z' |head -c 16

}

rm -f $SYMBOL_DB_FILE
rm -f $HEAD_FILE
createTable

touch $HEAD_FILE
echo '#ifndef LuckyCoffee_codeObfuscation_h
#define LuckyCoffee_codeObfuscation_h' >> $HEAD_FILE
echo "//confuse string at `date`" >> $HEAD_FILE
cat "$STRING_SYMBOL_FILE" | while read -ra line; do
if [[ ! -z "$line" ]]; then
ramdom=`ramdomString`
echo $line $ramdom
insertValue $line $ramdom
echo "#define $line $ramdom" >> $HEAD_FILE
fi
done
echo "#endif" >> $HEAD_FILE


sqlite3 $SYMBOL_DB_FILE .dump
```



<p id="已使用加固的项目"></p>
## 四、已使用加固的项目
> [< 返回目录](#目录)

#### 1、LuckyClient项目加固

一、采用的方案：如上

二、已涉及到的范围：

* 1、支付敏感信息

>
```
LuckyPayService类及其下的方法
LuckyUnitePay类及其下的方法
```

* 2、登录的敏感信息

>
```
LuckyLoginModel类及其下的方法
```

* 3、用于混淆'所混淆的类'是重要类的'原本不需要混淆的类'

>
```
LuckyGiveCoffeeModel
UserInfo
```

三、不去修复的地方：

`UPPaymentControl`虽是支付部分，但其是属第三方静态库`libPaymentControl.a`，若只混淆暴露出来的`.h`文件中的类和方法名，而不去改变编译后的`.a`文件，容易出问题，所以不去处理。

四、未来可能需添加涉及的范围：

* 加密库



<p id="其他"></p>
## 五、其他
> [< 返回目录](#目录)

其他更详细的请查看:

* [APP加固(一、代码混淆CodeObfuscation)](https://www.jianshu.com/p/ccfe5623483d)