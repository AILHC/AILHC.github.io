---
title: 框架的诞生-一：我想要的框架
categories:
  - 框架设计
tags:
  - EasyGameFramework
  - 框架设计
comments: true
abbrlink: 29232
date: 2020-11-29 17:23:49
img: "https://coding-pages-bucket-434147-7588795-4574-367535-1255530080.cos.ap-guangzhou.myqcloud.com/img/dog-wenhao.jpg"
---
## 目录

[TOC]

## 我想要的框架是什么样的？

<img src="https://coding-pages-bucket-434147-7588795-4574-367535-1255530080.cos.ap-guangzhou.myqcloud.com/img/dog-wenhao.jpg" style="zoom: 50%;" />

上一篇文章: 《框架的诞生-零：为什么写框架？》里有讲 什么是框架。

框架是一个架子，在游戏程序中，抛开渲染层引擎框架，我们的指的框架就是支撑业务逻辑的架子，也是一个框框，规范和约束着开发人员。

每个框架都有着自己的边界，解决特定领域问题的。

那我们从去分析我遇到了什么问题，有什么需求，如何解决问题，如何实现需求。

无论是小项目还是大项目，或者是一个会不断扩张的项目，都希望能够项目代码结构清晰有条理，管理好不同模块。

### 我尝试过的方式

#### Manager of Managers

很多框架都用这种方式（包括我之前写的框架）

这是刘钢老师的《[Unity 项目架构设计与开发管理](https://zhuanlan.zhihu.com/p/64858713)》中讲到的一种相对较好的方式 ▼

<img src="https://coding-pages-bucket-434147-7588795-4574-367535-1255530080.cos.ap-guangzhou.myqcloud.com/img/manger-of-mangers.png" style="zoom:67%;" />

优点是:类似于分级结构，各司其职；比如音频管理，场景管理，关卡管理等，每一个都是一个单例脚本，配合使用。结构相对清晰。可以复用

但我个人不太喜欢的点

- 调用不方便，调用链太长，费手指。比如 UIManager.getInstance().showUI 或者 UIMgr.ins.showUI
  - 我想这样: m.uiMgr.showUI 少敲几个字母，也不想 import UIMgr
- 无生命周期统一管理，单例初始化和依赖不太可控。惰性初始化，等到调用 getInstance 才初始化。
- 控制台调试调用不方便，单例可能得单独绑定暴露到全局才能，每个模块都得这样做才行
- 直接依赖不可动态替换
  - 有个模块我想在不同的平台切换成不同的实现，单例做不到。
- 大多耦合引擎来开发，只能在同引擎项目进行复用

#### 模块字典挂载到全局变量 window

这是我之前的框架使用的方式

就是将所有的模块初始化，然后注入一个模块字典

然后将字典挂载到全局的一个变量中。

```ts
export class Main {
    constructor(){
        const moduleMap = {

        };
        moduleMap["uiMgr"] = new UIMgr();
        moduleMap["netMgr"] = new NetMgr();

        window["aa"] = moduleMap;
    }
}
class UIMgr {

}
class NetMgr {

}
```

优点：调用方便

缺点: 有点危险，别人知道了可以在控制台调用进行调试。

这些方式在模块管理上都有问题，先不考虑如何方便调用，先实现个如何管理模块的核心机制。

### 一个具有生命周期的模块化机制

#### pomelo 给我的灵感

这个想法的灵感来自 pomelo 一个网易开发的基于 nodejs 的分布式服务器框架。

pomelo 支持可插拔的 component 扩展架构

用户只要实现 component 相关的接口： start, afterStart, stop, 就可以加载自定义的组件：

```ts
app.load([name], comp, [opts])
```

start, afterStart 这些生命周期 接口跟 cocos 和 Unity 的组件式接口很像。
主要是方便处理不同模块之间的依赖引用。比如：A 依赖了 B，但 B 还未初始化。

各自的初始化，都在 start 里处理，然后在 afterStart 里进行依赖调用。

可能对于不同的业务，这些生命周期可能不够用，可以根据具体业务进行扩展，满足自定义需求。

比如登录业务相关的：

C 模块依赖 A 和 B 登录后的数据状态，那么增加两个接口 onLoginInit,onAfterLoginInit。

那么 A 和 B 实现 onLoginInit 接口进行登录初始化，C 在 onAfterLoginInit 接口进行依赖调用。

## 怎么实现我想要的框架？

**模块生命周期图** ▼

![模块生命周期图](https://coding-pages-bucket-434147-7588795-4574-367535-1255530080.cos.ap-guangzhou.myqcloud.com/img/EgfCoreLifeCircle.png)

**接口设计**

```ts
declare global {
    namespace egf {

        interface IModule {
            /**模块名 */
            key?: string
            /**
             * 当初始化时
             */
            onInit?(app: IApp): void;
            /**
             * 所有模块初始化完成时
             */
            onAfterInit?(app: IApp): void;
            /**
             * 模块停止时
             */
            onStop?(): void;
        }
        type BootEndCallback = (isSuccess: boolean) => void;
        /**
         * 引导程序
         */
        interface IBootLoader {
            /**
             * 引导
             * @param app
             */
            onBoot(app: IApp, bootEnd: BootEndCallback): void;
        }
        /**
         * 主程序
         */
        interface IApp<ModuleMap = any> {
            /**
             * 程序状态
             * 0 未启动 1 引导中, 2 初始化, 3 运行中
             */
            state: number;
            /**
             * 模块字典
             */
            moduleMap: ModuleMap;
            /**
             * 引导
             * @param bootLoaders
             */
            bootstrap(bootLoaders: egf.IBootLoader[]): Promise<boolean>;
            /**
             * 初始化
             */
            init(): void;
            /**
             * 加载模块
             * @param module
             */
            loadModule(module: IModule | any, key?: keyof ModuleMap): void;
            /**
             * 停止
             */
            stop(): void;
            /**
             * 获取模块实例
             * @param moduleKey
             */
            getModule<T extends IModule = any>(moduleKey: keyof ModuleMap): T;
            /**
             * 判断有没有这个模块
             * @param moduleKey
             */
            hasModule(moduleKey: keyof ModuleMap): boolean;

        }
    }
}

// eslint-disable-next-line @typescript-eslint/semi
export { }
```

#### Bootloader: CatLib 给我的灵感

这里有一个 bootloader 的东西我没有讲到，它的灵感来自 CatLib，一个我觉得很棒的 Unity 框架。

这个机制是什么呢？以开发测试环境和生产环境举例。

有一个 debugBootLoader，这个引导程序处理一些测试用的模块加载和初始化，杂七杂八的。

当你发布生产环境时，可以通过 debug 变量屏蔽加载这个引导程序，也可以通过编译工具剔除这段代码。

具体实现可以看：https://github.com/AILHC/EasyGameFrameworkOpen/tree/main/packages/core#readme

#### 怎么使用?

具体使用请看 demo 工程

cocoscreator2.x 的 demo https://github.com/AILHC/egf-ccc-empty

cocoscreator3d 的 demo https://github.com/AILHC/egf-ccc3d-empty

**如何接入项目**▼

```ts
//FrameworkLoader.ts
import { HelloWorld } from "../HelloWorld";
export class FrameworkLoader implements egf.IBootLoader {
    onBoot(app: egf.IApp, bootEnd: egf.BootEndCallback): void {
        const helloWorld = new HelloWorld();
        app.loadModule(helloWorld);
        bootEnd(true);
    }

}
//AppMain.ts
import { App } from "@ailhc/egf-core"
import { FrameworkLoader } from "./boot-loaders/FrameworkLoader";
import { setModuleMap, m } from "./ModuleMap";
/**
 * 这是一种启动和初始化框架的方式，在cocos加载脚本时启动
 * 不依赖场景加载和节点组件挂载
 */
export class AppMain {
    public static app: App<IModuleMap>;
    public static initFramework() {
        const app = new App<IModuleMap>();
        AppMain.app = app;
        app.bootstrap([new FrameworkLoader()]);
        setModuleMap(app.moduleMap);
        app.init();
        window["m"] = m;//挂在到全局，方便控制台调试，生产环境可以屏蔽=>安全
        m.helloWorld.say();
    }

}
AppMain.initFramework();

```

接入项目很简单，new 一下，bootstrap，init 就可以了~

注入模块也很简单

```ts
//在UIMgr.ts开头增加个声明
declare global {
    interface IModuleMap {
        uiMgr:UIMgr
    }
}
//在初始化地方，注入实例
app.loadModule(UIMgr.getInstance(),"uiMgr");
```

注入的模块是什么类型的，不限制，你可以将业务模块 比如 HeroModule 注入进去，那么业务模块之间就可以直接调用了。也不用担心 typescript 的循环引用了。

举个栗子（随便的）:

```ts
// BattleModule.ts
m.hero.showHero(1);

//HeroModule.ts
m.battle.startTestBattle(1);

```

就像服务端的 rpc 调用一样。

```ts
app.rpc.chat.chatRemote.kick(session, uid, player, function(data){
});
```

至于怎么使得接口调用更方便，这个看个人的喜好，我呢，用了一点点魔法，让自己用着舒服又有点安全感。具体实现细节请看 demo

### 我想在 CocosCreator 和 C3d 中使用

由于我的工作中是用 Laya 的，项目也用了这个框架。但我私底下都是玩 CocosCreator 和 CocosCreator3d 的(为什么啊？你懂得 😉😉)

我不想在项目之间将源码拷贝来拷贝去，迭代更新同步麻烦。

如果能像 npm 包一样 安装就好了。而且核心模块是一个模块，其他模块也是一个模块。

于是我开发了一个模块编译发布的工具，开发之前以为很简单，实际上，踩坑了好久 😂。

#### 这个模块编译发布工具有什么功能？

- 编译模块成 iife、commonjs、systemjs 格式的 js 文件
- 自动生成单个.d.ts 声明文件

这个 systemjs 格式的 js 文件可以让不支持 npm 包的 CocosCreator3d 可以像使用 npm 包一样使用。即使到时 Cocos3.0 支持 npm 了，使用方式也一模一样。使用 C3d1.2.0 发布 web，微信小游戏，验证运行没有问题。

```ts
import { App } from '@ailhc/egf-core';//像引用npm包一样引用
import { _decorator, Component, Node } from 'cc';
import { m, setModuleMap } from './ModuleMap';
import { FrameworkLoader } from './boot-loaders/FrameworkLoader';
const { ccclass, property } = _decorator;
@ccclass('AppMainComp')
export class AppMainComp extends Component {
    /* class member could be defined like this */
    // dummy = '';

    /* use `property` decorator if your want the member to be serializable */
    // @property
    // serializableDummy = 0;

    onLoad() {
        this._initFramework();
    }
    private _initFramework() {
        const app = new App<IModuleMap>();
        // new TestImport();
        app.bootstrap([new FrameworkLoader()]);
        // app.bootstrap([new FrameworkLoader2()]);
        setModuleMap(app.moduleMap);
        app.init();
        window["m"] = m;//挂在到全局，方便控制台调试，生产环境可以屏蔽=>安全
        m.helloWorld.say();
    }
    start() {

    }

    // update (dt) {}
}
```

#### 如何开发一个模块

1. 克隆项目 git clone https://github.com/AILHC/EasyGameFrameworkOpen.git
2. 复制 packages/package-template 项目，改文件夹名，改 package.json 里的项目名等信息
3. npm install 初始化项目
4. 然后用 typescript 进行开发，使用 index.ts 文件将所有代码 export（可以使用 export-typescript 插件自动化，插件版本必须是 0.0.5 之前的）
5. 使用 egf build 进行编译发布

### 总结一下框架有什么特性

- 轻量级模块化机制
- 模块生命周期
  - 让模块的初始化有序，依赖可控
- 可面向接口编程
  - 方便具体实现细节可替换，模块可动态替换
- 友好的类型声明
  - 点一下就有类型提示，传字符串获取模块也有类型提示，很香的。
- 基于 TypeScript 与引擎无关
- 每个模块库都是一个 npm 包
- 模块库可以导出多种 js 格式，让 laya，ccc，c3d 使用，甚至给 Unity、Unreal 用([xLua 作者的 Puerts](https://github.com/Tencent/puerts/) 了解一下？)

## 这个框架可以做什么？

### 特性

- 基于轻量级无依赖的模块机制，可以为不同项目量身定制框架，可大可小。也可以根据项目的不同阶段进行渐进式扩展。还可以在项目的不同阶段轻易地接入
- 面向接口编程的模块，底层组件可以无感知替换
- 基于模块开发工具，我们可以开发和发布一个单独的对核心零依赖的模块，给不同的项目使用。

  - 方便别的项目引用
  - 方便开源
  - 方便做单元测试

- 基于模块化机制和配套开发工具，大家可以为公司或者个人建立自己的模块库，在不同项目按需复用。

### 架构设想 ▼

<img src="https://coding-pages-bucket-434147-7588795-4574-367535-1255530080.cos.ap-guangzhou.myqcloud.com/img/EgfCoreDesign.png" alt="基于egf-core的架构设想图" style="zoom: 80%;" />

谢谢大家阅读我的文章，希望大家能有所收获。



## 框架开发系列文章



* [框架的诞生-零：为什么写框架？](https://ailhc.github.io/2020/11/17/The-Birth-of-Frames-Zero%EF%BC%9AWhy-write-framework/)

* [框架的诞生-一：我想要的框架](https://ailhc.github.io/2020/11/29/The-Birth-of-a-Framework-One-The-Framework-I-Want/)

* 打破CocosCreator3d不能使用npm包的魔咒!!!

* 不只是 UI 管理:通用显示管理

* 让 fairygui 更好用的插件

* 满足多种需求的通用对象池

* 构建游戏/应用的神器:broadcast

* 满足所有自定义需求的通用 socket 网络模块

* 业务开发总结之状态管理

* 待续。。。



## 最后



欢迎关注我的公众号，更多内容持续更新



公众号搜索:玩转游戏开发



或扫码:<img src="https://coding-pages-bucket-434147-7588795-4574-367535-1255530080.cos.ap-guangzhou.myqcloud.com/img/20201128164023.png" alt="img" style="zoom:50%;" />



QQ 群: 1103157878



博客主页: https://ailhc.github.io/

掘金: https://juejin.cn/user/3069492195769469

github: https://github.com/AILHC