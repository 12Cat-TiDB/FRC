# 12Cat
# 基本介绍
## 代码仓库
- 本次魔改的 TiUP 代码：https://github.com/12Cat-TiDB/tiup
- 闪亮登场的 TiUP 组件市场所在仓库：https://github.com/12Cat-TiDB/tiup-community

## 团队名称：12只喵
- [qqqdan](https://github.com/qqqdan): 产品喵，喵2只
- [nexustar](https://github.com/nexustar): 后端研发喵，没有喵
- [srstack](https://github.com/srstack): 后端研发喵，喵8只
- [AstroProfundis](https://github.com/AstroProfundis): 后端研发喵，喵2只

## 项目进展
【完整版，可以提交初赛评审】

1. 使 TiUP 支持同时使用多个软件仓库 @nexustar @AstroProfundis
2. 利用 github pages 或其他服务来搭建一个 “tiup community”软件仓库 @srstack
3. 向“tiup community”软件仓库中添加开发者常用命令行工具 @srstack
4. maybe 让 tiup-playground 和 tiup-cluster也支持多软件仓库 @nexustar @AstroProfundis
5. 增加 tiup 的用法，更方便管理和使用多版本软件 @nexustar

## 项目介绍

使 TiUP 迈向通用包管理工具。TiUP 具有着包管理的能力，用于从 PingCAP 下载 TiDB 的各个组件和周边工具。本项目将会扩展 TiUP 的能力，完成多软件仓库功能和一些使用优化，使得用户能够使用 TiUP 来安装甚至与 TiDB 无关的软件包。并使用 Github Pages 或其他公开服务，搭建一个任何人都可以添加组件的 TiUP 软件仓库。

## 背景 & 动机

tiup 定位于 TiDB 生态的包管理工具，能够从 PingCAP 官方软件仓库下载 TiDB 组件和周边工具。但是从能力上来说，TiUP 可以做到的远不止于此。它是以一个类似“绿色软件”的方式来实现的、无需 root 权限的、支持多版本共存和快速切换的通用的包管理器。如果通过现有的能力，可以反向将其的产品定义变得更加通用。

对比其他包管理工具，tiup 有一些独特且值得保留的特点，12Cat团队将会进一步扩展和应用这些能力。
- 对比 apt/dnf/pacman：跨系统且不需要 root 权限
- 对比 homebrew：专注二进制程序分发而不是源码编译、无需 root 权限
- 对比 nix：使用方便
- 对比 flatpak：跨系统
- 对比其他所有：可以用 tiup xxx:v1.x.x 的方式快速在不同版本间切换使用

### 场景1: 实用工具的快速获取方式

有很多开发者和客户经常使用的软件，却并没有简单的安装方式，比如 mysql client、zstd。用户要么遵从官网复杂的安装步骤，要么依赖系统全局的包管理工具，最好的方式也是复制一行 curl 来安装。将他们添加到一个 TiUP 软件仓库中能够让这些步骤更易用

### 场景2 ：TiDB 生态的发布平台

程序员经常会来了灵感，写一个有用的小工具。却往往由于分发方式不够易用而很难推广给别人（比如众多 hackathon 项目）。12Cat 项目可以提供一个 TiDB 组件的发布平台，让社区开发将自己的组件上传到平台上。 TiUP 的使用者只需要配置了组件平台的镜像，就可以 list 发现各种组件，选择自己想要的进行使用。

### 场景3: 更方便得测试和使用私有组件

由于 TiUP 只支持设置一个镜像，官方镜像又不能添加私有组件，很多开发者不得不在私有镜像和官方镜像之间切换，经常因为切了镜像而找不到组件。12Cat 项目会让 TiUP支持同时设置多个镜像，彻底解决切镜像的苦恼。对于测试场景，也能够将交付的正式版本和日常的众多测试版本进行分离

## 项目设计

本项目主要有三个大的功能点：
- 让 TiUP 支持同时设置多个镜像，并支持快速搭建新镜像。
- 让 TiUP 支持各种工具和组件的安装和升级
- 提供一个可以让社区开发者贡献组件的开发者镜像，打造 TiDB 组件市场

### 使用设计

保留原来的总是使用 tiup 前缀的使用方式，也添加像其他包管理工具一样将组件路径添加到 PATH 的用法

1. tiup 传统用法
```
tiup zstd:v1.2.3
```

2. 新用法
```
tiup install zstd
zstd xxxxxx
tiup unlink zstd
tiup link zstd:v1.2.3
```

### 多镜像设计

![设计图](/media/pic.png)

#### 兼容性

规则：考虑兼容性，如果用户只有一个镜像，mirror相关的操作与老版本一致，不变化。

Check list：
- [ ] 通过命令 tiup mirror init 从零生成
- [ ] 通过命令 tiup mirror clone 默认从已有全部镜像克隆
- [ ] tiup mirror set  不再使用，提示使用新的命令
 
#### 配置多个mirror

规则：
- Mirror有权重、别名、path
- Mirror的权重默认是100，用户可以修改，越小优先级越高。权重相同，按别名字母顺序。
- Mirror的权重和path支持命令修改。别名也可以改。
- 添加mirror时，权重默认100，必须提供别名，别名不允许重名。

命令：
- tiup mirror add <name> <url> --priority=11
- tiup mirror delete <name> 
- tiup mirror set <name> url=XXX  or priority =XXX
- tiup mirror rename <name> <string>
- tiup mirror show ： 显示路径和order
```
    Priority    name    		path
    1	  AAA 			https://tiup-mirrors.pingcap.com
    100	 BBB 			XXXXXXXXXXX
 ```

#### 使用多mirror：多个mirror的处理顺序

- Install，list，status等操作的时候，都会按顺序去每个mirror获取数据
- 如果多个mirror有相同组件的包，用优先级高的mirror

提示信息：
- 每次通过install，list，status逐个查询多个mirror时以及添加镜像时，发现用户配置了超过3个镜像，提示 "You have configed more than 3 tiup mirrors, and this may make tiup response slower."
- 当有一个或多个mirror失联时，提示失联的mirror信息

#### Mirror失联的处理

- 任意mirror失联时就停止工作, 直接报错。
- 提供忽略某个异常 mirror 的参数，类似 yum 的 `--disablerepo`

备注：直接失败可能不太友好，但是规避一些异常场景，第一版本以这种方式发布，如果客户使用有问题，可以讨论改进。

#### 自定义 Mirror

- 基于 tiup server, 自定义满足 tiup mirror 规则的镜像源
- 支持 reproducible-build
- 支持自定义源码编译发布
- 支持静态二进制上传
- 在软件包中增加 meta 文件描述 entrypoint 等关键信息
- 增加安装后 hook 执行入口



## 缺点

## FAQ

## 效果验证
### 环境
### Config
### Workload
### 结果
