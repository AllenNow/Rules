# iOS 开发代码风格

## 基础语法部分

### 注释格式

```objc
/*
 * 接口注释一律使用 Xcode 自带的 ⌘ + option + / 来完成
 **/
- (void)method

extern NSString *const XXXX;    ///< 单行定义可以用这种

/** 单行定义也可以用这种 */
extern NSString *const YYYY;

方法集注释：#pragma mark - 注释内容

```

### Tab 和空格之间的选择

1. 代码前面的缩进一律使用 Tab
2. 代码中间的分隔符一律使用空格
3. 注释对齐推荐使用 Tab，如： `代码{tab或者空格}//{tab}注释`

### 缩进的 Tab 使用 4 个空格的宽度

可以在 Project 配置中设置

### 使用驼峰命名方法

使用英文全称命名，勿用首字母缩写，不使用拼音命名。

如果开头是专有名词，可以用大写。

如 `+[NSUUID UUID]`、`+[NSURL URLWithString:]`

### 左花括号不要另起一行

### 定义枚举请使用 NS_ENUM

### 建议在方法与方法之间应该有且只有一行，这样有利于在视觉上更清晰和更易于组织

使用 `NS_ENUM` 来定义枚举，如果是位开关，可以使用 `NS_OPTION`

勿用 `enum` 关键字

### 常量应该使用 const 来声明

勿用 `#define` 来定义常量，`#define` 只能用于函数

如果需要对外开放，使用 `FOUNDATION_EXTERN` 或者 `UIKIT_EXTERN`。

### 字面值的使用

建议 `NSString`、`NSDictionary`、`NSArray` 和 `NSNumber` 的字面值应该在创建这些类的不可变实例时被使用。请特别注意 `nil` 值不能传入 `NSArray` 和 `NSDictionary` 字面值，因为这样会导致 crash。

### switch-case 用法

用花括号，break 写在花括号里。

```c
switch (number) {
	case 1: {
		//xxxxxxxxxxxxxxxxx
		// 错
	}
		break;
	case 3: {
		//xxxxxxxxxxxxxxxxx
		// 对
		break;
	}
}
```


## Objective-C 语法部分

### 类前缀命名

应用前缀使用 LC(X) 命名，如：用户端 LCC，配送端 LCD，OA LCE

### 分类方法前缀

必须要用前缀，并以功能或者作者命名，否则统一使用 `luckin_`

### 尽量不要重写 `+ load` 方法

重写 `+ load` 方法会在 dyld 加载二进制的时候执行代码，会有以下后果：

1. 大量的 `+ load` 方法会延长 App 的启动时间
2. 在 `+ load` 方法中执行面向对象特性的代码可能会有隐患

如果要做启动时的 **Method Swizzling** 请尽量异步

```objc
+ (void)load {
    dispatch_async(dispatch_runtime_queue, ^{
        dispatch_once(^{
            //  Method Swizzling
        });
    });
    //  dispatch_runtime_queue
    //  会在 UIApplicationDidFinishLaunching 之后异步执行
}
```

如果 Method Swizzling 针对的对象不会在启动的时候创建，可以通过懒加载实现：

```objc
@implementation LuckyWebView

- (instancetype)init {
    self = [super init];
    if (self) {
        dispatch_once(^{
            //  针对 UIWebView 或者 WKWebView 的 Method Swizzling
        });
        _webView = [[UIWebView alloc] init];
    }
    return self;
}

@end
```

### NSString 属性统一使用 copy 关键字修饰

防止传入 NSMutableString 导致不安全



### 初始化方法，类构造方法返回值使用使用instancetype而不是id
### 在dealloc方法中，禁止将self作为参数传递出去
如果self被retain住，到下个runloop周期再释放，则会造成多次释放crash。
```
-(void)dealloc{
    [self unsafeMethod:self];
}
```
> 因为当前已经在self这个指针所指向的对象的销毁阶段，销毁self所指向的对象已经在所难免。如果在unsafeMethod:中把self放到了autorelease poll中，那么self会被retain住，计划下个runloop周期在进行销毁。但是dealloc运行结束后，self所指向的对象的内存空间就直接被回收了，但是self这个指针还没有销毁(即没有被置为nil)，导致self变成了一个名副其实的野指针。到了下一个runloop周期，因为self所指向的对象已经被销毁，会因为非法访问而造成crash问题。

### init 和 delloc中禁止使用self.xxx访问属性
如果存在继承的情况下，很有可能导致崩溃。

### 方法如果参数太多，分行时冒号对齐

```objc
[NSObject want:@""
            to:@""
          call:@""];
```

**PS**: 在 `+[UIView animationXXX]` 方法中，冒号对其不一定会增加易读性，反而会出现不必要、不整齐的代码缩进。

### 私有方法开头

在 *.m* 文件中用两个下划线开头表示私有方法 (因为一个下划线被苹果占用了):

```objc
- (void)__thisIsAPrivateMethod {
}
```

### 定制的子类方法排序

使用 `#pragma mark - XXXXX` 来分割代码。排序的规则是

1. 重写父类的方法 > 子类自己的方法
2. 经常调用的方法 > 很少调用的方法
3. 公开方法 > 私有方法
4. Notification > Target-Action > Data Source > Delegate
5. setter > getter
6. 懒加载的 getter 写在最后

```objc
#pragma mark - 生命周期、重写父类
#pragma mark - 公开方法
#pragma mark - 私有方法
#pragma mark - Selector (通知、Target-Action等)
#pragma mark - Protocol (Data Source、Delegate等)
#pragma mark - Property Setter
#pragma mark - Property Getter (Lazy load)
```

### 定制的子类方法个数控制

打开编辑器顶部的方法导航菜单，菜单不得超过屏幕高度，需要直接完整显示。如果超过，可以考虑分离代码。

### 属性定义

属性的 attribute 排序

1. nonatomic, atomic
2. strong, weak, assign, retain
3. readonly, readwrite(Optional)
4. setter=
5. getter=
6. nullabel, nonull (Optional)

`(non)atomic` 在 `strong` 之前是为了更好的对齐。

### 关联对象

用这个属性的 getter 当做传入的 `(const void *)key`

```objc
- (void)setAssociationObject:(id)object {
    objc_setAssociationObject(self, @selector(demo), object, RETAIN);
}
- (id)associationObject {
    objc_getAssociationObject(self, @selector(demo));
}
```

### 特殊 Key 和特殊 Name 处理

应用场景：UIApplicationLaunchOptionsKey, NSNotificationName

定义类型与直接使用宏的比较

1. 内存占用一样
2. 都可以防止外部直接打 `@""` 时出现手残打错现象
3. 自定义类型可以在敲代码的时候缩小提示范围，调用者会更加明确该如何传参

如果你的模块需要对外发送通知，或者传递个特殊的 NSDictionary 给外部，需要如下定义

头文件中

```objc
#if UIKIT_STRING_ENUMS
typedef NSString * LCCustomKey NS_EXTENSIBLE_STRING_ENUM;
#else
typedef NSString * LCCustomKey;
#endif

FOUNDATION_EXTERN LCCustomKey const LCCustomXXXXXKey;

@interface LCObject : NSObject
@end
```

实现文件中

```objc
@implementation LCObject

@end

LCCustomKey const LCCustomXXXXXKey = @"LCCustomXXXXXKey";

```

### 单例

使用 `+ sharedInstance` 配合 `dispatch_once` 完成单例。并在头文件中注明注释

请勿重写 `+ allocWithZone:` 方法达到强制单例的目的。
请慎重使用单例，避免产生不必要的常驻内存。

### 类的内部修改自己的属性值（有争议）

对象修改自己的公开属性，直接通过调用 `_实例变量 = 值` 完成。

而不是调用 `self.属性 = 值` 来完成修改。

但是这种方法和 KVO 和 RAC 有冲突，具体怎么执行待商议。

### 懒加载
懒加载适合的场景
- 一个对象的创建依赖于其他对象；
- 一个对象在控制器生命周期或者app生命周期中可能用到也可能不用到；
- 一个对象的创建需要消耗大量性能；
除了以上，应该少用懒加载；懒加载的本质是延迟初始化某个对象，如果一个对象在懒加载后，某些场景下又被设置为nil,可能懒加载还会再次触发。

### 执行 Block

```objc
//  对于没有返回值的 block
!block?:block(参数);

//  对于有返回值的 block
!block?nil:block(参数);
```

### 低版本的 API 适配

```objc
//	使用
if (@available(iOS 10.0, *)) {
	[[UIApplication sharedApplication] openURL:URL option:@{} completed:nil];
} else {
	[[UIApplication sharedApplication] openURL:URL];
}
```

```objc
// 禁止
if (UIDevice.currentDevice.systemVersion.floatValue >= 10.0) {
}

//	禁止
if ([UIApplication.sharedApplication respondsToSelector:@selector(openURL:)]) {
}
```

### 可空标识

为了适应 Swift 项目，OC 需要在对外头文件添加可空标识。可以通过以下两个方法来定义，推荐第二种方法。**所有 CocoaPods 库均要服从此规范，业务代码可以不用服从此规范**

**1. 使用 Xcode 模板标识**

在文件头部和尾部都添加了 Nonull 标识，表示所有对象均不为空：

```objc
NS_ASSUME_NONNULL_BEGIN

//	文件内容

NS_ASSUME_NONNULL_END
```

**2. 针对每一个对象引用独立标识**

使用 `nonnull`，或者 `nullable` 来标识属性

```objc
@property (nonatomic, strong, nonnull) id object;
```

使用一个下划线 + 大写开头的单词 `_Nonnull` 和 `_Nullable` 来标识非属性。

注意：

1. 对于一个单词的引用，如：`id`，`NSNotificationName`，标识写在类型后，并用空格分割。
2. 对于类引用，如：`NSObject *`, `NSString *`，标识写在 `*` 后，并用空格分割。
3. 不管是上述哪种引用，如果后面显示定义变量名称，如：`NSObject *object`，要在标识符后加入空格：`NSObject * _Nonnull object`

```objc
//	 返回值
- (id _Nonnull)getObject;

//	普通参数
- (void)setObject:(id _Nullable)object;

// block 参数
- (void)onCompleted:(void (^ _Nullable)(void))completed;
- (void)onSuccess:(LuckinNetworkingSuccessBlock _Nullable)success;

//	block 里的参数
- (void) onCompleted:(void (^ _Nullable)(NSDictionary * _Nullable response, NSError * _Nullable error))completed;
```

## 非语法部分

### 处理长逻辑和错误中断

**不推荐**

```objc
//  不推荐 1
if (第一步成功) {
    if (第二步成功) {
        if (第三步成功) {
            处理代码
        }
    }
}
扫尾工作

//  不推荐 2
if (第一步成功 == NO) {
    扫尾工作
    return;
}
if (第二步成功 == NO) {
    扫尾工作
    return;
}
if (第三步成功 == NO) {
    扫尾工作
    return;
}
处理代码
```

**推荐**

```objc
NSError *error = nil;
do {
    if (第一步成功 == NO) {
        error = [NSError new];
        break;
    }
    if (第二步成功 == NO) {
        error = [NSError new];
        break;
    }
    if (第三步成功 == NO) {
        error = [NSError new];
        break;
    }
    处理代码
} while (NO)
扫尾工作
```

### 条件逻辑

*推荐*
```
if (someObject) {}  
if (![anotherObject boolValue]) {}  
```
*不推荐*
```
if (someObject == nil) {}  
if ([anotherObject boolValue] == NO) {}  
if (isAwesome == YES) {}  
```

### if 条件语句使用大括号

建议条件语句主体为了防止出错应该使用大括号包围，即使条件语句主体能够不用大括号编写

**推荐**

```objc
if (!error) {
	return success;
}
```

**不推荐**

```objc
if (!error)
	return success 
	
// 或者

if (!error) return success;
```
### 三目运算符
三目运算符禁止嵌套使用和嵌入复杂取值过程；
推荐
```
NSInteger value = 5;  
result = (value != 0) ? x : y;  
BOOL isHorizontal = YES;  
result = isHorizontal ? x : y; 
```
不推荐
```
result = a > b ? x = c > d ? c : d : y; 

![LuckyGateKeeperManager sharedManager].useDataUnion? (![Utility isEmptyObj:[Utility getDeviceIdentifier]]? [Utility getDeviceIdentifier]:@""):dataUnion
```

### 禁止出现 Warning

1. CocoaPods 可以在 `Podfile` 中关闭警告
2. 手动拖拽的三方库可以加入 ignore warning 宏来关闭警告
3. 自己写的代码不允许出现警告
4. 建议直接在`Build Settings` 中打开`Treat Warnings as Error`

### 对外方法使用断言判断参数合法性

模块开发者需要使用断言来判断模块调用者传入的参数是否合法。

这么做可以有效的在开发过程中处理掉一些因为参数错误的 crash 情况。

### 错误处理
当方法通过引用来返回一个错误参数，需要预先设置`NSError *error = nil`。
(在成功的情况下，有些Apple的APIs记录垃圾值(garbage values)到错误参数(如果non-NULL)，那么判断错误值会导致false负值和crash)

例如

```
NSError *error = nil;  
[self trySomethingWithError:&error];
if (error) {  
// Handle Error  
}
```

###  IO相关
1. 尽量少用NSUserDefaults;
[[NSUserDefaults standardUserDefaults] synchronize] 会block住当前线程，知道所有的内容都写进磁盘，如果内容过多，重复调用的话会严重影响性能
2. 一些经常被使用的文件建议做好缓存。避免重复的IO操作。建议只有在合适的时候再进行持久化操作;

### 不建议将UIView类的对象加入到NSArray、NSDictionary、NSSet中。

不建议将UIView类的对象加入到NSArray、NSDictionary、NSSet中。如有需要可以添加到NSMapTable 和 NSHashTable。因为NSArray、NSDictionary、NSSet会对加入的对象做strong引用（即使你把加入的对象进行了weak）。而NSMapTable、NSHashTable会对加入的对象做weak引用。

```
// 错误的例子：
@implementation WSObject
- (void)dealloc {
NSLog(@"dealloc");
}
@end

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
WSObject *object = [WSObject new];
// 即使对object进行了weak弱化，数组也会强引用这个object对象。dealloc方法不会被执行。
__weak typeof(object) weakObject = object;
[self.arrM addObject:weakObject];

dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
NSLog(@"count = %ld",self.arrM.count);
});
}

// 打印结果：
// count = 1
```

```
// 正确的例子：
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
WSObject *object = [WSObject new];

NSHashTable *hashTable = [NSHashTable weakObjectsHashTable];
[hashTable addObject:object];
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
NSLog(@"count = %ld",hashTable.count);

if (hashTable && hashTable.count) {
WSObject *object = [hashTable anyObject];
NSLog(@"object = %@",[object self]);
}
});
}

// 打印结果：
6834:4305636] dealloc
6834:4305636] count = 1
6834:4305636] object = (null)

```
### 以 “自动释放池块” 降低内存峰值
- 通常for循环会不断创建新对象加入自动释放池里，循环结束才释放。因此，可能会占用大量内存。手动加入自动释放池块（@autoreleasepool）：每次for循环都会直接释放内存，从而降低了内存的峰值。尤其是在遍历处理一些大数组或者大字典的时候，可以使用自动释放池来降低内存峰值.

```
NSArray *qiShare = /*一个很大的数组*/
NSMutableArray *qiShareMembersArray = [NSMutableArray new];
for (NSStirng *name in qiShare) {
    @autoreleasepool {
        QiShareMember *member = [QiShareMember alloc] initWithName:name];
        [qiShareMembersArray addObject:member];
    }
}
```

### Notification
1. 尽量少用。
2. post通知时，object通常是指发出notification的对象，如果在发送notification的同时要传递一些额外的信息，请使用userInfo，而不是object。

3.(求证) NSNotificationCenter在iOS8及更老的系统有一个多线程bug，selector执行到一半可能会因为self的销毁而引起crash，解决的方案是在selector中使用weak_strong_dance。如下：
```
- (void)onMultiThreadNotificationTrigged:(NSNotification *)notify {
    __weak typeof(self) wself = self; __strong typeof(self) sself = wself; 
    if (!sself) { return; }
    [self doSomething]; 
}
```
4. 在多线程应用中，Notification在哪个线程中post，就在哪个线程中被转发，而不一定是在注册观察者的那个线程中。如果post消息不在主线程，而接受消息的回调里做了UI操作，需要让其在主线程执行。
> 每个进程都会创建一个NotificationCenter，这个center通过NSNotificationCenter defaultCenter获取，当然也可以自己创建一个center。
NoticiationCenter是以同步（非异步，当前线程，会等待，会阻塞）的方式发送请求。即，当post通知时，center会一直等待所有的observer都收到并且处理了通知才会返回到poster。如果需要异步发送通知，请使用notificationQueue，在一个多线程的应用中，通知会发送到所有的线程中。

```

