---
title: 通用游戏UI框架的设计与实现
categories:
  - 框架设计
  - 分类2
tags:
  - EasyGameFramework
  - 框架设计
comments: true
abbrlink: 53689
date: 2020-12-19 22:06:23
img:
---
@[TOC](通用游戏UI框架的设计与实现)

## 前言

这个月月初，我发了几篇文章分享了我写框架的心路历程和一些自己的想法。
感兴趣的可以通过文末链接回顾。

同时我发布了我的第一个开源渐进式H5游戏开发框架: [EasyGameFrameworkOpen](https://github.com/AILHC/EasyGameFrameworkOpen)

提供强大轻量级的核心库:模块管理库 [@ailhc/egf-core](https://www.npmjs.com/package/@ailhc/egf-core)。可以无缝接入任意引擎游戏项目

以及提供一个基于rollup的typescript库构建工具: [@ailhc/egf-cli](https://www.npmjs.com/package/@ailhc/egf-cli)，可以构建出适合任意引擎项目的js库和单一.d.ts声明文件 类似cocos.d.ts声明文件。

在之前的文章中，我也预告了，还有后续的分享。后续的分享也是关于框架其他库的设计与实现的。

虽然文章还没出，但一直在更新。

大家可以关注框架的github仓库，如果可以，给个star哦。
![](https://coding-pages-bucket-434147-7588795-4574-367535-1255530080.cos.ap-guangzhou.myqcloud.com/img/比心.jpg)
### 为什么鸽了这么久才更新呢？
本来预期是上个星期更新的。

但是涅~(￣▽￣)~*
1. 首先工作比较忙，家里又有事情需要处理
2. 同时，接下来要讲的这个方面我觉得比较难，我想要讲清楚讲明白(原谅我稚嫩的笔触)
3. 我想添加尽可能多的单元测试和demo例子(毕竟要往更加规范的工程化靠拢)

鸽子🕊 : 别哔哔，鸽了就是鸽了，赶紧进入正题~

我: 好嘞~

## 背景
做游戏，绝大部分，UI交互界面是必不可少的。

当项目的UI界面多起来的时候，而且策划们的需求千奇百怪，迭代频繁的时候，就会让我们非常头疼。

接下来我将以我稚嫩的笔触分享我是如何分析和解决这个头疼问题的。

这个议题偏向主观和开放，每个人都有自己的业务开发经历和想法，而我就在此抛砖引玉，献丑了。

如有不足，请各位多多指教🙏( •̀ ω •́ )✧

## 本质需求
我们来看两种情景

1. 策划提了一个需求:一个有五彩斑斓的黑背景的文本提示界面B->程序员去实现界面B的逻辑

2. 点击A按钮->显示B界面

这两个情景就是我们开发中最最最常见，最最最本质的两个需求：

1. 实现UI界面逻辑

2. 调用界面

那么我们头疼也就是这两个需求的实现，我分别讲一下

### UI逻辑的实现

UI逻辑的实现是各种各样，需求也是千奇百怪。

而且这个实现可能不是一步就位的，可能还需要迭代，重做等。

那么在这种情况下，我作为UI逻辑的开发者，我希望


1. 不用关注别的业务逻辑在什么地方怎么调用我，只需要通过告诉外界如何调用我
2. 拥有足够高的自由度
    1. 我不一定要加载prefab，我希望可以加载一个图片来动态new一个node来添加组件显示
    2. 也不一定要实例化prefab，作为显示节点，我希望可以通过绘图api自己画界面显示
    3. 我不一定要用cocoscreator的显示渲染，我希望可以通过写html显示，可以接fgui显示，可以调用安卓和ios的原生界面显示
    4. 我不一定使用通用的加载释放逻辑，我希望可以自定义加载和资源释放逻辑
    5. 可以自己控制节点添加到那个父节点。
    6. 可以依赖多个不同类型的资源，动态生成
3. 在不改变函数接口的情况下，拥有足够高的扩展性

这样高可扩展，高自由度，对外界透明可以让开发者更加专注高效地实现和迭代UI逻辑。
* 不用因为别人调用不规范导致出bug而头疼
* 不用因为束手束脚而头疼
* 不用为想扩展又不想改接口而头疼

### UI的调用
我们在项目开发中可能会有各种UI调用的需求：

UI调用最复杂也是最常用的是**显示调用**，比如
1. 显示A界面
2. 显示B界面并按照B界面的显示数据接口传递数据给B界面，B按照传递的数据渲染，甚至传递各种回调
3. 显示C界面，想C显示接口调用后回调显示完成回调，即想在界面显示完成后执行逻辑

其他常用的是
1. 更新
2. 隐藏
3. 销毁

还有一些特殊的
比如：预加载指定界面（不显示），获取指定界面依赖的资源（用于批量加载多个界面依赖的资源）

除了对功能的需求，还有对接口扩展性的需求

因为在不同项目中，或者不同项目阶段中，可能会增加一些需求，需要扩展接口参数的。
但如果接口参数太多，调用就可能会很长很麻烦。

所以会希望，接口有更强的扩展性，而且在不改动底层的情况增加接口就能实现需求。

回头一看好像是自己给自己挖坑啊。不过造轮子的，不都是自己找坑挖坑吗？

分析完需求，我们就可以着手去设计接口了

## 接口设计


一个简单的流程图
![框架流程图](https://coding-pages-bucket-434147-7588795-4574-367535-1255530080.cos.ap-guangzhou.myqcloud.com/img/display-ctrl-framework.jpg)



再看一张UML图
![框架UML](https://coding-pages-bucket-434147-7588795-4574-367535-1255530080.cos.ap-guangzhou.myqcloud.com/img/display-ctrl-umljpg.jpg)


### UI控制器的接口设计
UI控制器的职责就是：**实现UI逻辑**
```ts
 /**
 * 显示配置
 */
interface IShowConfig<
      TypeKey extends keyof any = any,
      InitDataTypeMapType = any,
      ShowDataTypeMapType = any,
      > {
    typeKey?: TypeKey,
    /**
     * 透传初始化数据
     */
    onInitData?: InitDataTypeMapType[ToAnyIndexKey<TypeKey, InitDataTypeMapType>]
    /**
     * 强制重新加载
     */
    forceLoad?: boolean
    /**
     * 显示数据
     */
    onShowData?: ShowDataTypeMapType[ToAnyIndexKey<TypeKey, ShowDataTypeMapType>],
    /**在调用控制器实例onShow后回调 */
    showedCb?: CtrlInsCb;
    /**控制器显示完成后回调 */
    showEndCb?: VoidFunction;
    /**显示被取消了 */
    onCancel?: VoidFunction,
    /**加载后onLoad参数 */
    onLoadData?: any,
    /**加载完成回调,返回实例为空则加载失败，返回实例则成功 */
    loadCb?: CtrlInsCb
}
interface ICtrl<NodeType = any> {
    key?: string | any;
    /**正在加载 */
    isLoading?: boolean;
    /**已经加载 */
    isLoaded?: boolean;
    /**已经初始化 */
    isInited?: boolean;
    /**已经显示 */
    isShowed?: boolean;
    /**需要显示 */
    needShow?: boolean
    /**需要加载 */
    needLoad?: boolean
    /**正在显示 */
    isShowing?: boolean

    /**
     * 透传给加载处理的数据,
     * 会和调用显示接口showDpc中传来的onLoadData合并,
     * 以接口传入的为主
     * Object.assign(ins.onLoadData,cfg.onLoadData); 
     * */
    onLoadData?: any;
    /**获取资源 */
    getRess?(): string[] | any[];
    /**
     * 初始化
     * @param initData 初始化数据
     */
    onInit(config?: displayCtrl.IInitConfig): void;
    /**
     * 当显示时
     * @param showData 显示数据
     */
    onShow(config?: displayCtrl.IShowConfig): void;
    /**
     * 当更新时
     * @param updateData 更新数据
     * @param endCb 结束回调
     */
    onUpdate(updateData: any): void;
    /**
     * 获取控制器
     */
    getFace<T>(): ReturnCtrlType<T>;
    /**
     * 当隐藏时
     */
    onHide(): void;
    /**
     * 强制隐藏
     */
    forceHide(): void;
    /**
     * 当销毁时
     * @param destroyRes 
     */
    onDestroy(destroyRes?: boolean): void;
    /**
     * 获取显示节点
     */
    getNode(): NodeType;
}
```
这个接口不依赖任何引擎的接口和类。每个引擎项目实现对应的基类

管理器和业务不需要关注UI逻辑是怎么实现的，只需要调用接口并按照对外的数据接口传数据对象就可以了

0. onLoadData 这个是通用加载透传数据，比如：告诉通用加载逻辑显示什么加载等待界面
1. 初始化接口onInit ，主要是用于实例化显示节点，监听UI交互事件等
2. 显示接口onShow，主要是将UI显示并根据传入的数据进行渲染用
3. 更新接口onUpdate，主要是根据传入的数据进行渲染更新
4. 获取资源依赖接口 getRess，主要是通用资源处理逻辑需要获取UI界面依赖的资源信息进行加载/释放


其他的接口就很简单了，省略
我重点讲一下，onShow这个接口的设计
第一版的设计是 
```ts
/**
 * @param onShowData 调用时的透传数据
 */
onShow(onShowData?:any): void;
```
这样的设计，只能实现自定义透传显示数据的，但是要想实现管理逻辑的扩展而不改动onShow接口，就会非常麻烦

比如：
我想扩展管理器，给UI控制传一个显示完成的回调：派发UI显示完成事件，让UI逻辑中动画播放完，或者其他延迟操作结束后调用。

那按照第一版的设计，那么只能加个参数：onShow(onShowData?:any,showEnd?:VoidFunction)

这个时候就是得修改ICtrl这个接口设计了。一个还可以，那如果变成了两个三个四个参数呢？就会变得复杂。
现在的设计就可以解决这个问题，扩展透传的config参数的接口,不会增加接口参数，不改变接口


这个灵感来自[axios](https://github.com/axios/axios)这个库,一个易用、简洁且高效的http库

**关于自定义资源处理实现**

有自定义资源处理需求的UI控制器，需要实现这个接口
```ts
/**
 * 资源处理器
 */
interface IResHandler {
    /**
     * 加载资源
     * @param config 
     */
    loadRes?(config: displayCtrl.IResLoadConfig): void;
    /**
     * 释放资源
     * @param ctrlIns 
     */
    releaseRes?(ctrlIns?: ICtrl): void;
}
```
那么管理器就会调用这个自定义的资源处理接口，而不是走通用的资源处理了

这样的接口设计可以说是在UI被管理的同时，给予了UI逻辑实现者极大的自由度，可以专注高效地实现和迭代，以及任意扩展

极大的自由度是有代价的，就是不能开箱即用。需要去实现接口，其实也很简单，所以这个是值得的。

具体的demo示例可以看:[egf-ccc-full](https://github.com/AILHC/EasyGameFrameworkOpen/tree/main/examples/egf-ccc-full)中的实现

继续讲UI管理器的设计

### UI管理器的设计
UI 管理器的职责就是:**提供接口让业务去调用UI**
```ts
interface IMgr<
    CtrlKeyMapType = any,
    InitDataTypeMapType = any,
    ShowDataTypeMapType = any,
    UpdateDataTypeMapType = any> {
    /**控制器key字典 */
    keys: CtrlKeyMapType;
    /**
     * 控制器单例字典
     */
    sigCtrlCache: CtrlInsMap;
    /**
     * 初始化
     * @param resHandler 资源处理
     */
    init(resHandler?: IResHandler): void;
    /**
     * 批量注册控制器类
     * @param classMap 
     */
    registTypes(classes: CtrlClassMap | CtrlClassType[]): void;
    /**
     * 注册控制器类
     * @param ctrlClass 
     * @param typeKey 如果ctrlClass这个类里没有静态属性typeKey则取传入的typeKey
     */
    regist(ctrlClass: CtrlClassType, typeKey?: keyof CtrlKeyMapType): void;
    /**
     * 是否注册了
     * @param typeKey 
     */
    isRegisted<keyType extends keyof CtrlKeyMapType>(typeKey: keyType): boolean;
    /**
     * 获取注册类的资源信息
     * 读取类的静态变量 ress
     * @param typeKey 
     */
    getDpcRessInClass<keyType extends keyof CtrlKeyMapType>(typeKey: keyType): string[] | any[]
    /**
     * 获取单例UI的资源数组
     * @param typeKey 
     */
    getSigDpcRess<keyType extends keyof CtrlKeyMapType>(typeKey: keyType): string[] | any[];
    /**
     * 获取/生成单例显示控制器示例
     * @param typeKey 类型key
     */
    getSigDpcIns<T, keyType extends keyof CtrlKeyMapType = any>(typeKey: keyType): displayCtrl.ReturnCtrlType<T>
    /**
     * 加载Dpc
     * @param typeKey 注册时的typeKey
     * @param loadCfg 透传数据和回调
     */
    loadSigDpc<T, keyType extends keyof CtrlKeyMapType = any>(typeKey: keyType, loadCfg?: displayCtrl.ILoadConfig): displayCtrl.ReturnCtrlType<T>;
    /**
     * 初始化显示控制器
     * @param typeKey 注册类时的 typeKey
     * @param initCfg displayCtrl.IInitConfig
     */
    initSigDpc<T, keyType extends keyof CtrlKeyMapType = any>(
        typeKey: keyType,
        initCfg?: displayCtrl.IInitConfig<keyType, InitDataTypeMapType>
    ): displayCtrl.ReturnCtrlType<T>;
    /**
     * 显示单例显示控制器
     * @param typeKey 类key或者显示配置IShowConfig
     * @param onShowData 显示透传数据
     * @param showedCb 显示完成回调(onShow调用之后)
     * @param onInitData 初始化透传数据
     * @param forceLoad 是否强制重新加载
     * @param onCancel 当取消显示时
     */
    showDpc<T, keyType extends keyof CtrlKeyMapType = any>(
        typeKey: keyType | displayCtrl.IShowConfig<keyType, InitDataTypeMapType, ShowDataTypeMapType>,
        onShowData?: ShowDataTypeMapType[displayCtrl.ToAnyIndexKey<keyType, ShowDataTypeMapType>],
        showedCb?: displayCtrl.CtrlInsCb<T>,
        onInitData?: InitDataTypeMapType[displayCtrl.ToAnyIndexKey<keyType, InitDataTypeMapType>],
        forceLoad?: boolean,
        onLoadData?: any,
        loadCb?: displayCtrl.CtrlInsCb,
        onCancel?: VoidFunction
    ): displayCtrl.ReturnCtrlType<T>;
    /**
     * 更新控制器
     * @param key UIkey
     * @param updateData 更新数据
     */
    updateDpc<keyType extends keyof CtrlKeyMapType>(key: keyType, updateData?: UpdateDataTypeMapType[ToAnyIndexKey<keyType, UpdateDataTypeMapType>]): void;
    /**
     * 隐藏单例控制器
     * @param key 
     */
    hideDpc<keyType extends keyof CtrlKeyMapType>(key: keyType): void;
    /**
     * 销毁单例控制器
     * @param key 
     * @param destroyRes 销毁资源
     */
    destroyDpc<keyType extends keyof CtrlKeyMapType>(key: keyType, destroyRes?: boolean): void;

    /**
     * 实例化显示控制器
     * @param typeKey 类型key
     */
    insDpc<T, keyType extends keyof CtrlKeyMapType = any>(typeKey: keyType): ReturnCtrlType<T>;
    /**
     * 加载显示控制器
     * @param ins 
     * @param loadCfg 
     */
    loadDpcByIns(ins: displayCtrl.ICtrl, loadCfg?: ILoadConfig): void;
    /**
     * 初始化显示控制器
     * @param ins 
     * @param initData 
     */
    initDpcByIns<keyType extends keyof CtrlKeyMapType>(
        ins: displayCtrl.ICtrl,
        initCfg?: displayCtrl.IInitConfig<keyType, InitDataTypeMapType>): void
    /**
     * 显示 显示控制器
     * @param ins 
     * @param showCfg
     */
    showDpcByIns<keyType extends keyof CtrlKeyMapType>(
        ins: displayCtrl.ICtrl,
        showCfg?: displayCtrl.IShowConfig<keyType, InitDataTypeMapType, ShowDataTypeMapType>
    ): void;
    /**
     * 通过实例隐藏
     * @param ins 
     */
    hideDpcByIns<T extends displayCtrl.ICtrl>(ins: T): void;
    /**
     * 通过实例销毁
     * @param ins 
     * @param destroyRes 是否销毁资源
     */
    destroyDpcByIns<T extends displayCtrl.ICtrl>(ins: T, destroyRes?: boolean, endCb?: VoidFunction): void;

    /**
     * 获取单例控制器是否正在
     * @param key 
     */
    isLoading<keyType extends keyof CtrlKeyMapType>(key: keyType): boolean
    /**
     * 获取单例控制器是否加载了
     * @param key 
     */
    isLoaded<keyType extends keyof CtrlKeyMapType>(key: keyType): boolean;
    /**
     * 获取单例控制器是否初始化了
     * @param key 
     */
    isInited<keyType extends keyof CtrlKeyMapType>(key: keyType): boolean;
    /**
     * 获取单例控制器是否显示
     * @param key 
     */
    isShowed<keyType extends keyof CtrlKeyMapType>(key: keyType): boolean;
    /**
     * 获取控制器类
     * @param typeKey 
     */
    getCtrlClass<keyType extends keyof CtrlKeyMapType>(typeKey: keyType): CtrlClassType<ICtrl>;
}
```
这里有一些TypeScript的类型编程魔法。可以实现及其强大的类型提示。看不懂的可以先忽略，先来讲一下接口设计

在设计这个UI管理接口时，一直念着：**职责是管理UI，提供接口调用UI，不做多余的事情。**
为什么？因为脑子里总有很多功能想要塞进去，但其实都只是我想要的不必要的功能

所以设计得很克制，资源处理接口外包出去了，因为**职责是管理UI，提供接口调用UI**

什么栈式UI管理啊，也没有做，因为并不是所有项目都需要，只是特殊需求。

所有项目都需要的是：**管理UI，提供接口调用UI**。

既然讲到管理UI，那游戏开发中都有哪些UI呢？

如果按具体的讲，各有各的说法，什么Dialog啊，Tips啊，Window啊等等,说不完的!

我抽象一下，按照同时存在多少个UI实例来分：只有两种类型的UI
1. 单例UI
    
    比如：通用加载界面，养成界面，角色展示界面，战斗界面等等，同时有且仅有一个实例存在
    ![角色展示](https://coding-pages-bucket-434147-7588795-4574-367535-1255530080.cos.ap-guangzhou.myqcloud.com/img/afk角色展示界面.png)

2. 多实例UI
    比如：奖励获得tips，属性提升tips
    ![属性提升tips](https://coding-pages-bucket-434147-7588795-4574-367535-1255530080.cos.ap-guangzhou.myqcloud.com/img/Snapshot_20201213215918.png)

那业务逻辑通过UI管理器
1. 调用显示UI，就是想如果有就直接显示出来，没有就加载创建一个来显示
2. 调用显示n个UI，就是想创建n个UI同时显示不同的内容

这个抽象逻辑是我能想到的所有游戏项目都通用的逻辑。

那UI管理的接口设计就应该是：提供调用单例UI和多实例UI的接口。仅此

大家估计都看到了很多特殊的类型声明，下面我简单的讲一下这些类型声明的作用

### 类型提示优化

如果想要更加舒适的接口调用体验，那么就必须榨干typescript的类型系统提供的能力（ps:有为什么不用？）
```ts

//displayCtrl.IMgr
isLoading<keyType extends keyof CtrlKeyMapType>(key: keyType): boolean

//实现UI
import { BaseDpCtrl } from "./base-dp-ctrl";
declare global {
    interface ITestCtrlKeyType {
        OnUpdateDpc: "OnUpdateDpc",
        OnShowDpc: "OnShowDpc",
        OnInitDpc: "OnInitDpc"
    }
    interface ITestCtrlShowDataMap {
        OnShowDpc: number
    }
    interface ITestCtrlInitDataMap {
        OnInitDpc: number
    }
    interface ITestCtrlUpdateDataMap {
        OnUpdateDpc: number
    }
}
export class OnUpdateDpc extends BaseDpCtrl {
    public static readonly typeKey: "OnUpdateDpc" = "OnUpdateDpc";
    public updateData: number;

    constructor() {
        super();
    }
    onUpdate(updateData: number) {
        this.updateData = updateData;
    }

}
export class OnShowDpc extends BaseDpCtrl {
    public static readonly typeKey: string = "OnShowDpc";
    public showData: number;

    constructor() {
        super();
    }
    onShow(config: displayCtrl.IShowConfig<"OnShowDpc", ITestCtrlShowDataMap>) {
        this.showData = config.onShowData;
        super.onShow(config)
    }

}
export class OnInitDpc extends BaseDpCtrl {
    public static readonly typeKey: "OnInitDpc" = "OnInitDpc";
    public initData: number;

    constructor() {
        super();
    }
    onInit(config: displayCtrl.IInitConfig<"OnInitDpc", ITestCtrlInitDataMap>) {
        this.initData = config.onInitData;
    }

}
//实例化UI管理器
const dpcMgr = new DpcMgr<ITestCtrlKeyType,ITestCtrlInitDataMap,ITestCtrlShowDataMap,ITestCtrlUpdateDataMap>();//注入类型ITestCtrlKeyType
        dpcMgr.init({
            loadRes: (config) => {
                config.complete();
            }
        });
        dpcMgr.regist(OnUpdateDpc);
        dpcMgr.regist(OnInitDpc);
        dpcMgr.regist(OnShowDpc);
//调用一个简单的
dpcMgr.isLoading("")//当双引号敲出，就会弹出类型提示选择:OnUpdateDpc，OnShowDpc，OnInitDpc

//调用一个复杂的
dpcMgr.showDpc("")//当双引号敲出，就会弹出类型提示选择:OnUpdateDpc，OnShowDpc，OnInitDpc,
//同时，需要传递onShowData时，也会有对应UIkey的onShowData类型提示
dpcMgr.showDpc("OnShowDpc",{})

```
为什么要设计这样的类型提示？

1. 我想让UI逻辑实现者更加专注，只需要在顶部添加声明,业务逻辑调用就知道该传什么数据了。不用去翻找文件找接口
```ts
declare global {
    interface ITestCtrlKeyType {
        OnUpdateDpc: "OnUpdateDpc",
        OnShowDpc: "OnShowDpc",
        OnInitDpc: "OnInitDpc"
    }
    interface ITestCtrlShowDataMap {
        OnShowDpc: number
    }
    interface ITestCtrlInitDataMap {
        OnInitDpc: number
    }
    interface ITestCtrlUpdateDataMap {
        OnUpdateDpc: number
    }
}
```
2. 我想让UI调用者：业务逻辑，调用得更加舒服、无依赖、无import

**关于typescript类型编程参考资料**

* [编写TypeScript工具类型，你需要知道的知识](https://www.cnblogs.com/chanwahfung/p/12229788.html)

* [TypeScript 的工具类型](https://zhuanlan.zhihu.com/p/78180787)

* [深入 TypeScript 的类型系统](https://zhuanlan.zhihu.com/p/38081852)

* [TypeScript系列（三）从编程语言到Conditional Types](https://zhuanlan.zhihu.com/p/60253127)

## 其他可能性
虽然这篇文章讲的是UI管理框架

但其实我代码里和接口设计的方向并不限制只是UI管理，你可以用于管理各种抽象显示单位

UI只是其中一种，这个抽象显示单位可以是
1. 一个小部件
2. 主角
3. 敌人
4. 子弹
5. 等等

它不只是UI框架，它是**通用显示管理框架**

## 总结

关于UI框架的设计，提出UI业务实现中的两个本质的需求
1. 高效、灵活且专注的UI逻辑实现
2. 高效、灵活且高可扩展的UI管理

分析本质需求，提出更加细化的需求

根据需求设计和实现了一套跨引擎零依赖高效灵活高可扩展的UI框架。

适用于所有游戏引擎项目，可以根据自身需求任意扩展

并且通过研究typescript的类型编程，为接口调用提供了超级舒适的类型提示



具体的实现逻辑和CocosCreator2.4.2的demo，大家可以移步框架

GitHub仓库：
[EasyGameFrameworkOpen](https://github.com/AILHC/EasyGameFrameworkOpen)

希望大家可以给个star,谢谢~

谢谢大家阅读我的文章~

祝大家周末愉快~

## 框架开发系列文章



* [框架的诞生-零：为什么写框架？](https://juejin.cn/post/6900546197753167886)

* [框架的诞生-一：我想要的框架](https://juejin.cn/post/6900545484356583431)

* [打破CocosCreator3d不能使用npm包的魔咒!!!](https://juejin.cn/post/6900942099663978510)

* [框架的诞生-二：定位](https://juejin.cn/post/6901906785255620615)

* 通用游戏UI框架的设计与实现

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



QQ 群: 1103157878 (欢迎前来讨论吹水)



博客主页: https://ailhc.github.io/

掘金: https://juejin.cn/user/3069492195769469

github: https://github.com/AILHC