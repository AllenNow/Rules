# CocoaPods 使用规范

随着业务迭代和扩展，公司私有框架变得越来越多，需要用到大量的 CocoaPods 仓库来管理这些私有框架。在使用 CocoaPods 管理私有框架的过程中，每个人、每个项目、每个框架的配置都有差异，因为配置的差异会导致硬盘的不必要的资源占用，同时也可能会出现不同的权限拒绝的情况。为了防止使用 CocoaPods 管理私有框架导致的各种问题，现制定以下使用标准。

## 专有名词释义

在制定规范之前，需要对下文产生的一些名词作解释，防止出现歧义。

**Project**: 项目，或者项目的 git 仓库，或者项目的 *Podfile* 文件

> 完整的 App 项目包含有：用户端、配送端、OA、咖啡师

**Spec**: 框架，或者私有框架的 git 库，或者私有框架的 **.podspec* 文件

> 现有的私有框架都存放在 **ios-team** 的账号下，每个私有框架都单独一个 git 仓库。并且每个私有框架的根目录都有一个 **.podspec* 文件，存放此私有库的配置信息。

**Repositories**: 仓库，

> 公司里的 git-lab 里只有一个 Repo 的 git 仓库。该仓库存放所有私有框架的 **.podspec* 文件。
> 地址为: http://git.luckincoffee.com/ios-team/LuckySpace

## 传输规范

为了防止权限问题，所有私有库传输都使用 SSH 传输。通过配置 ssh-key 避免输入账号密码的情况。同时为了防止多个 ssh-key 混用的问题，规定使用默认的 ssh key 文件名称（如果你自信的话，可以自己使用别的 ssh-key 名称）。

对于 GitHub 上的公共库，推荐使用 https 来传输，此处不做限制。

**SSH 传输限制仅对内网私有库有效**

#### 1. 创建默认的 ssh-key

如果你已经创建了默认的 ssh-key（存在 ~/.ssh/id_rsa）则可以跳过此步骤

创建默认 ssh-key 的方式如下：

```shell
$ ssh-keygen
```

提示输入秘钥路径：

```
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/user/.ssh/id_rsa):

#此处直接回车，使用默认的路径。
```

提示输入秘钥的密码：

```
Enter passphrase (empty for no passphrase):

#直接回车（空白密码）
```



提示再次输入密码：

```
Enter same passphrase again:

#直接回车（再次输入空白密码）
```

即可得到密钥对。

输出公钥

```shell
$ cat ~/.ssh/id_rsa.pub
```

#### 2. 配置账号权限

将上述公钥添加到 gitlab 的账号中即可。

## 配置规范

### 1. 框架配置（podspec 编写规范）

配置文件规范如下（仅提到需要规范的字段，未提到的字段没有规范）：

```ruby
Pod::Spec.new do |s|

# 框架名称
  s.name         = "name"
  
# 框架版本，使用 x.x.x 格式，不带 v 或者 version
  s.version      = "0.0.2"

# 框架概要
  s.summary      = "一句话说明框架功能，不写的话会有警告，规定要写"
  s.description  = <<-DESC
好几句话详细说明框架功能，不写会有警告，规定要写，且和 summary 不能一样
                   DESC

# 框架地址
  s.homepage     = "http://git.luckincoffee.com/ios-team/name"
  
# 框架 git 地址
# 1. git 字段必须写 ssh 地址
# 2. tag 字段必须写 "#{s.version}"
  s.source       = { :git => "git@git.luckincoffee.com:ios-team/name.git", :tag => "#{s.version}" }

end
```

此规范仅对公司内部私有库有效
1. 如果要开发公司私有框架，`s.source` 地址使用 SSH
2. 如果你要发布公开框架到 GitHub 上，并且推送到 CocoaPods 的公开仓库 `master` 上，推荐 `s.source` 使用 https 地址。

### 2. 仓库配置（本地 repo 存储规范）

规定每个人电脑上本地仓库名称、地址都需要统一（方便给大家编写私有库发布、更新脚本）。

> 由于发布、更新私有框架需要涉及到此私有库的本地名称，同时私有库的 URL 和名称又是绑定的。所以为了编写统一的发布、更新脚本，就必须要统一私有库的 URL 和名称

查看本地仓库列表的方法：

```bash
$ pod repo
```

得到列表

```
93-luckyspace
- Type: git (master)
- URL:  git@10.204.93.171:ios-team/LuckySpace.git
- Path: /Users/user/.cocoapods/repos/93-luckyspace

master
- Type: git (master)
- URL:  https://github.com/CocoaPods/Specs.git
- Path: /Users/user/.cocoapods/repos/master
```

第一个就是本地仓库信息，该仓库可以自己通过 pod 命令创建，同时也会通过 `pod install` 或者 `pod update` 时使用 `Podfile` 文件中的 `source` 配置来自动创建。

为了统一第一个私有库（公司的私有库）的命名以及 URL 地址，规定统一使用 CocoaPods 自动生成的仓库名称和仓库地址

> 自动生成的方法
> 
> 通过命令删除你自己已存在的仓库：
> 
> ```shell
> $ pod repo remove 你的仓库名称 # 仓库名称就是上面的 93-luckyspace
> 
> # 旧的仓库都是使用 HTTP 传输的，名称也是通过 URL 自动拼接出来的。
> # 所以名称和 URL 都不能直接使用。
> ```
> 
> 找到一个经过下面第 3 点项目配置过的项目（用户端、配送端等等），自己手动执行一次 
> 
> ```shell
> $ pod install
> ```
> 
> 列出本地仓库列表，查看是否自动生成私有库
> 
> ```shell
> $ pod repo
> ```
> 

### 3. 项目配置（Podfile 编写规范）

Podfile 编写规范如下：

```ruby
# 第一行 source 必须写公司私有库，使用 SSH 地址
source 'git@git.luckincoffee.com:ios-team/LuckySpace.git'
# 第二行 source 必须写官方公开库，使用 HTTP 地址
source 'https://github.com/CocoaPods/Specs.git'

# 删除所有警告
inhibit_all_warnings!

target 'LuckyClient' do
    platform :ios, '8.0'
    
    # 私有库指定地址的，使用 SSH 地址
    pod 'InsightSDK_dylib', :git => 'git@git.luckincoffee.com:ios-team/LuckinInsightARPod.git', :branch => 'master'
    
    # 共有库指定地址的，推荐使用 https，此处不做限制
    pod 'ReactiveObjC', :git => 'https://github.com/ReactiveObjC/ReactiveObjC.git'
end
```

## CocoaPods 开发流程

### 1. 开发框架

创建私有框架（也就是创建 *.podspec 文件）方式有两种。

##### 创建——对于已有代码的库

```shell
$ cd 框架根目录
$ pod spec create 名称
```

##### 创建——对于还没有代码的库

```shell
$ pod lib create
```
##### 开发

正常开发并在 git 上提交代码

### 2. 发布框架

* 编辑配置好 **.podspec* 文件并 commit 到 git 库
* 打 Tag 并推送到远程 git 库

> Tag 名称为 `*.*.*` 格式的版本号，此版本号应和 **.podspec* 文件一致

* 校验框架合法性

> ```shell
> $ cd podspec所在目录
> # pod lib lint 参数列表
> $ pod lib lint --allow-warnings --use-libraries --source=luckincoffee-luckyspace,master
> ```

* 发布框架 

> ```shell
> $ cd podspec所在目录
> # pod repo push 本地私有库名称 参数列表
> $ pod repo push luckincoffee-luckyspace --allow-warnings --use-libraries --sources=luckincoffee-luckyspace,master
> ```

命令行参数释义：

1. **--allow-warnings**: 允许警告，不填写的话出现警告就会验证不通过。**.podspec* 文件中的 `s.source` 使用 SSH 地址必然会有警告。

2. **--use-libraries**: 校验时使用 Library 模式，部分仓库会因为 sh 脚本执行失败导致验证不通过，加入此项可以解决。

3. **--sources=luckincoffee-luckyspace,master**: 指定源，`luckincoffee-luckyspace` 是本地私有库名称，`master` 是官方公开库名称。如果此时校验的私有框架依赖了别的私有框架，需要添加此参数并且 `luckincoffee-luckyspace` 写在 `master` 前面用英文逗号分隔。如果没有依赖私有框架，此参数可以不写。

### 3. 引用框架

**添加 source**

```ruby
source 'git@git.luckincoffee.com:ios-team/LuckySpace.git'
source 'https://github.com/CocoaPods/Specs.git'
```

**引用**

`pod 'Name'`

## CocoaPods 更新流程

### 1. 更新代码

正常提交代码

### 2. 发布版本

1. 修改 **.podspec* 配置（主要是 `s.version` 选项，版本号要提高）并提交到 git
2. 添加 Tag，Tag 名称和 1 中的 `s.version` 值一样并 push 到远程 git
3. 校验框架合法性，同**开发框架**上的说明
4. 发布框架，同**开发框架**上的说明

### 3. 更新项目

发布框架后，可以到项目中拉取最新的框架代码。更新步骤如下：

```shell
$ cd 项目根目录
$ pod update 框架名称 --no-repo-update
```

如果你要更新 GitHub 上的公开库，则删掉 `--no-repo-update` 选项。

> 经过上述第 2 步骤后，你的本地私有 repo 已经是最新状态，所以在执行 `pod update` 的时候可以不用将 `repo` 更新到最新，这样可以避免更新 `master` 这个 `repo` 的大量耗时。
> 
> 但是如果要更新公开框架，更新 `master` 是必要的，所以不能添加 `--no-repo-update` 参数。