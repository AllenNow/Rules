# Lucky iOS 用户端加固介绍

## 代码混淆

[iOS-CodeObfuscation](iOS-CodeObfuscation.md)

## 应用完整性校验

### 签名校验

#### 1. 方案

用户使用自己的 Apple ID 从 App Store 中购买并下载客户端时，得到的 `*.ipa` 文件中包含了用户的 Apple ID 以及购买信息。该 ipa 无法安装到别的 Apple ID 登录的 iOS 设备上。所以可以猜测，`*.ipa` 中包含了可以标致一个用户的值（可能是某些签名相关的文件）

#### 2. 校验合法性

在一些冷门界面上加入一个后门。触发后门后，使用 `NSFileManager` 将 `+[NSBundle mainBundle]` 拷贝到 *Documents* 目录中，并用 `ZipArchive` 库将其打包成 zip 文件，并通过文件共享的形式导入到别的 App（百度网盘） 中。

冷门界面选择 LuckyWebView，在《用户注册协议》界面的 titleLabel 中加入手势（三次点击，一次长按）即可触发后门。

#### 3. 执行

提交至 App Store 审核。审核通过后，内部使用各自的手机触发后门，得到各自 Apple ID 下载的 ipa 包。再使用控制变量法对内部文件进行 Hash 值比较。判断哪些文件的 Hash 是不同的。再依据结论来制定之后的校验规则。

### 砸壳校验

应用上传至 App Store 后会被 Apple 加壳（已购买该 App 的 Apple ID 作为秘钥加密二进制文件指令集）。只有被这 Apple ID 授权的手机才能安装该 ipa。

三方应用分发平台往往用一个账号购买 ipa 包后，手动脱去 Apple 加的壳。即可在其他任意设备上安装该 ipa。

应用内部可以做砸壳校验。判断该应用的合法性。

**但是，由于上传至 App Store 是没有加壳的，待 Apple 审核通过时才会加壳。如果此时做砸壳判断，Apple 在审核时就会出错，或者出现数据异常。所以砸壳判断需要配合服务器来远程控制开关。目前暂未实现。**



## 入侵检测

### 禁止调试

使用 ptrace 函数防止 gdb、lldb 进程依附。


```objc
#import <dlfcn.h>
#import <sys/types.h>

#ifndef PT_DENY_ATTACH
#define PT_DENY_ATTACH 31
#endif

static __attribute__((always_inline)) void LCDenyAttach() {
    //  使用常规办法调用 ptrace。该方法会产生符号表，可能会被 MobileSubstrate 或者 fishhook 干掉
    typedef int (*ptrace_ptr_t)(int _request, pid_t _pid, caddr_t _addr, int _data);
    void* handle = dlopen(0, RTLD_GLOBAL | RTLD_NOW);
    char *name = malloc(7 * sizeof(char));
    strcpy(name, "osqzbd"); // confuse: osqzbd + 1 = ptrace
    for (int i = 0; i < 6; i++) {
        name[i] = name[i] + 1;
        if (name[i] > 'z') {
            name[i] = 'a';
        }
    }
    ptrace_ptr_t ptrace_ptr = dlsym(handle, name);
    ptrace_ptr(PT_DENY_ATTACH, 0, 0, 0);
    dlclose(handle);
}
```

由于 ptrace 包含了符号，有可能会被干掉的嫌疑。所以再加入 arm 指令集直接操作。

```objc
static __attribute__((always_inline)) void LCAntiDebug() {
    //  使用 arm 汇编调用 syscall 传入 26 = ptrace。这个方法在 32 位设备上无效
#ifdef __arm64__
    __asm__("mov X0, #26\n"
            "mov X1, #31\n"
            "mov X2, #0\n"
            "mov X3, #0\n"
            "mov X4, #0\n"
            "mov w16, #0\n"
            "svc #0x80");
#endif
}
```

指令集操作只有在 arm64 设备上有用，所以最终是两个方案同时进行。

使用 always_inline 标识为内联函数，可以在编译时直接编译到调用者中，不会产生符号表，无法直接 hook。

## NSUserDefaus 加密处理




## HTTPS + DNS 优化