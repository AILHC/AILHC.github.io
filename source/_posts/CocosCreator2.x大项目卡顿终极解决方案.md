---
title: CocosCreator2.x大项目卡顿终极解决方案
categories:
  - EasyGameFramework
tags:
  - CocosCreator
comments: false
abbrlink: 17439
date: 2021-6-13 12:56:34
img: https://cdn.jsdelivr.net/gh/ailhc/picture/img/燕子.jpg

---

## 前言

论坛上有很多帖子、很多人说Creator大项目卡顿，也在苦苦寻找解决方案。

这对于每一个想用`Creator`做个大项目或者正在做着项目的小伙伴来说都是很难受的。

这可能会让他们割舍对Creator的喜爱，而另作选择。

或者在上了车之后，没法填这个坑导致项目黄了，那就更加难过了。

怎么办呢？我过年时就萌生了一个想法：我想摸一下Creator的上限。看看到底能不能解决大项目卡顿的问题，也是替各位小伙伴们探探路。

前段时间的探索，成功了。

那让我知道Creator开发大项目是**完全没有问题**的。

可以让我们用Creator爽，一直用一直爽的。![](https://img.soogif.com/OWr6i3dqLhhTFEjIEceyDipoZB48FlAy.gif?scope=mdnice)


文章: [`我想摸一下Creator的上限`](https://mp.weixin.qq.com/s/t7664GuXvEWz2BKnbpPDCw)

上个星期有个深陷水深火热的小伙伴找到我，让我帮帮忙
    ![](https://cdn.jsdelivr.net/gh/ailhc/picture/img/大项目-聊天记录1.png)
    ![](https://cdn.jsdelivr.net/gh/ailhc/picture/img/大项目-聊天记录2.png)
    
    

能够帮到他，非常开心。![](https://img.soogif.com/QhP0T3SNuoF58FaB68eekoSLzMTJX0kM.gif?scope=mdnice)


最近项目赶版本，比较忙，所以解决方案完善的进度慢了些。

虽迟但到~

![](https://cdn.jsdelivr.net/gh/ailhc/picture/img/表情-来了来了.gif)

视频演示:

**https://www.bilibili.com/video/BV1hh411e7p5/**

插件地址:

**https://store.cocos.com/app/resources/search?name=Aswallow**

## 我们感到难受的是什么？

先说开发过程中让人感觉很舒服的一种情况

`我改了一行代码，立刻就能看到运行情况`

让人感觉难受的是什么？

我改了一行代码，等了好久都没能运行起来

**反馈的时间越短，越舒服** <img src="https://cdn.jsdelivr.net/gh/ailhc/picture/img/表情-舒服.jpg" style="zoom:33%;" />

其实这里有一个很有趣的概念，叫`即时反馈` ，它是进入心流状态的条件之一（就是那种很爽很沉醉的状态）
有兴趣的可以查一下。

`CocosCreator`让我们在开发游戏过程中，能快速得到反馈

  1. 编辑场景，保存一下就可以刷新看结果
  2. 改一下代码，立刻编译看结果
  3. 修改shader立刻也能看到效果
  4. 遇到难点、bug翻一下论坛就能找到解决方案

这个就很舒服

但项目资源越多，这个改了一行代码，卡的时间就越长，越来越难受。

而项目后期大部分时间可能都是在改代码，也就意味着，一直卡。像吃屎一样。![](https://cdn.jsdelivr.net/gh/ailhc/picture/img/表情-吃屎.jpg)

### 其实。。。

其实很多引擎在项目资源多的时候都会出现卡顿、反馈时间长等问题，这个是通病。

有问题就解决嘛。我们先找找原因

另外:`CocosCreator`已经做得很好了，不是吗？

## 卡顿的根源:Creator编辑器的资源管理机制

Creator编辑器会对所有资源进行分析，记录它们之间的依赖关系。

`.meta`文件就是记录资源的信息和依赖信息

资源的修改、增加、删除，都可能会导致依赖信息变化。

所以Creator会监视所有assets目录下的文件，当某个资源变动，编辑器都会遍历检查一下变动的影响。

这是为了保证依赖的准确性，如果依赖缺失了就会立刻提示你。

因此，当资源量多了，这个处理耗时就会变得越来越长。

这也是以下情况的缘由

1. 保存一下prefab，卡顿好久
2. 保存一下代码，切回来卡了好久

正所谓成也萧何败也萧何


## 解决思路：拆
论坛上其实就有这个思路的方案，比如

1. 将图片预先打成图集=>减少图片资源量
2. 将资源放到CDN

Creator官方在`Creator2.4.x`版本也给出了他们的解决方案:AssetBundle

我觉得，在项目内分AssetBundle，卡顿问题还是没法解决的。因为资源量没变，处理耗时也也没有减少

但如果在另外一个项目制作资源的`AssetBundle`，然后导出。这个是可以解决问题的。

这些方案是有用，但就是有些麻烦，怎么才能做到更加方便和无感知呢？

而且还有一个问题，怎么加载解析图集、龙骨、spine甚至fgui发布的资源和tilemap呢？

因为官方的接口中，加载远程资源：只能加载简单的图片、音频、文本

 文档传送门🚪:https://docs.cocos.com/creator/manual/zh/scripting/dynamic-load-resources.html

## 我的解决方案

### 拆分资源的解决方案
#### 第一种:**自定义网页预览**

不知道大家知不知道这个的存在。

文档传送门:https://docs.cocos.com/creator/manual/zh/advanced-topics/custom-preview-template.html

在项目根目录创建`preview-templates`，然后编辑器的预览功能就会以这个目录为入口

你将外部资源放到这个文件夹，使用远程加载接口，就可以加载到这里的资源

```ts
//直接将someres.png放到preview-templates
var remoteUrl = "someres.png";
cc.assetManager.loadRemote(remoteUrl, function (err, texture) {
    // Use texture to create sprite frame
});
```
但这样需要自己在构建时处理资源，复制到发布目录


#### 第二种:**插件 aswallow 如燕(谐音:爱上我咯)**

1. 开发预览支持
    我通过hook creator的预览服务器逻辑，让你可以访问assets文件夹外的目录。对原来的预览服务器逻辑无影响
    你只需将需要拆分出去的资源放到assets文件夹外的ext-res文件夹内即可
    这个ext-res也可以通过修改local/aswallow-config.json来修改
2. 发布构建支持
    构建时，会自动将资源拷贝到构建输出目录，构建配置中的MD5Cache打开可以给文件名加md5，生成路径映射version.json文件
    也可以自己实现自定义的构建处理逻辑,具体可见custom-build-scripts/custom-build.js
    构建功能支持Creator2.3.x和2.4.x，以及发布构建支持：微信小游戏(其他小游戏没测，应该可以)、安卓原生、web
    当然，如果配合下面的这个运行时会更完美
    ps:听jare大佬说Assetbundle可以单独发布，我没找到。不过将资源项目发布到ext-res文件夹，就可以远程加载其中的bundle资源了

### 复杂资源加载解析方案

这个其实挺苦恼的，官方没有接口。

我在论坛找到了龙骨的加载解析逻辑，然后断点运行看源码完善了。图集、spine、tilemap也是断点运行看源码实现的。

找到了解析方法后，基于此实现了 `aswallow-asset-manager`

它可以让你更加简单的加载外部、解析和管理`外部资源`(图集、龙骨、spine、tilemap、fgui发布的资源)

通过加载version文件，可以实现轻易加载解析加了md5后缀的资源（加载逻辑不变的情况下）

使用示例：

1. 加载图集

   ```ts
   aswallow.extAssetMgr.load([{ url: "atlas/emoji", assetType: "plist" }], (err, result) => {
               console.log(result);
               const atlas =  aswallow.extAssetMgr.get("atlas/emoji.plist") as cc.SpriteAtlas;
               console.log(atlas);
               this.emojiSp.spriteFrame = atlas.getSpriteFrame("emoji1")
           });
   }
   ```

   

2. 加载图片

   ```ts
   let asset: cc.Asset;
   let index = 0;
   this._scheduleCallback = () => {
       asset = aswallow.extAssetMgr.get(`${iconRoot}/i${index}.png`);
       index = Math.floor(Math.random() * 10);
       this.sp.spriteFrame = null;
   
       this.sp.spriteFrame = new cc.SpriteFrame(asset as cc.Texture2D);
   }
   let iconRoot = "fgui-res/Icons";
   let resPaths = [];
   for (let i = 0; i < 10; i++) {
       resPaths.push(iconRoot + "/i" + i + ".png");
   }
   // cc.assetManager.preloadAny()
   aswallow.extAssetMgr.load(resPaths, (err, result: aswallow.ILoadResult) => {
       if (!err) {
           console.log(`加载成功`)
           console.log(result);
   
           this.schedule(this._scheduleCallback, 1, cc.macro.REPEAT_FOREVER);
   
   
           // this.sp.spriteFrame.ensureLoadTexture();
   
       }
   
   });
   ```

   

3. 加载龙骨

   ```ts
   const extAssetMgr = aswallow.extAssetMgr; 
   extAssetMgr.load([
        { url: "dragonbones/dragon/texture.json", assetType: "DragonBonesAtlasAsset" },
        { url: "dragonbones/dragon/NewDragonTest.json", assetType: "DragonBonesAsset" },
        "dragonbones/dragon/texture.png"], (err, items) => {
        console.log(items)
        this.dragonBone_json.dragonAsset = extAssetMgr.get("dragonbones/dragon/NewDragonTest.json") as any;
        this.dragonBone_json.dragonAtlasAsset = extAssetMgr.get("dragonbones/dragon/texture.json") as any;
        this.dragonBone_json.armatureName = 'armatureName';
        this.dragonBone_json.playAnimation('stand', 0);
    });
   //加载二进制
   
   extAssetMgr.load({ url: "dragonbones/sword-man/SwordsMan", assetType: "DragonBonesAsset", ext: ".dbbin" }, (err, items) => {
       this.dragonBone_bin.dragonAsset = extAssetMgr.get("dragonbones/sword-man/SwordsMan_ske.dbbin") as any;
       this.dragonBone_bin.dragonAtlasAsset = extAssetMgr.get("dragonbones/sword-man/SwordsMan_tex.json") as any;
       this.dragonBone_bin.armatureName = 'Swordsman-NestArmature';
       this.dragonBone_bin.playAnimation('walk', 0);
   })
   ```

   

4. 加载spine

   ```ts
   const extAssetMgr = aswallow.extAssetMgr;
   
   extAssetMgr.load([{ url: "spines/spineboy/spineboy.json", assetType: "SpineAsset" },
                     "spines/spineboy/spineboy.txt",
                     "spines/spineboy/spineboy.png"], (err, items) => {
       console.log(items)
       this.spine_json.skeletonData = extAssetMgr.get("spines/spineboy/spineboy.json") as any;
       this.spine_json.animation = 'run';
   });
   //加载二进制
   extAssetMgr.load([{ url: "spines/spineRatorBin/raptor-pro.skel", assetType: "SpineAsset" },
                     "spines/spineRatorBin/raptor-pro.atlas",
                     "spines/spineRatorBin/raptor-pro.png"], (err, items) => {
       console.log(items)
       this.spine_bin.skeletonData = extAssetMgr.get("spines/spineRatorBin/raptor-pro.skel") as any;
       this.spine_bin.animation = 'walk';
       // this.spine._updateSkeletonData
   });
   ```

   

5. 加载fgui

   ```ts
   //正常使用即可，接口没有变化 
   ```

6. 资源释放(以释放spine资源为例)

   ```ts
   aswallow.extAssetMgr.release([
       { url: "spines/spineboy/spineboy.json", assetType: "SpineAsset" },
       "spines/spineboy/spineboy.txt",
       "spines/spineboy/spineboy.png",
       { url: "spines/spineRatorBin/raptor-pro.skel", assetType: "SpineAsset" },
       "spines/spineRatorBin/raptor-pro.atlas",
       "spines/spineRatorBin/raptor-pro.png"
   ])
   ```

   

暂时支持2.4.x，发布构建测试通过的平台：Android原生、web、微信小游戏(其他小游戏平台应该也可以)

## 最后

CocosCreator其实是很强大的，不是吗？

卡顿问题也是可以轻易解决的，不是吗？

`希望每个喜爱Cocos的小伙伴不用纠结要不要用Creator开发大项目`

`希望在开发大项目的小伙伴不再受卡顿之苦`

`希望每个CocosCreator项目都有所成~`

能帮到你们真的很开心。

如果喜欢我的解决方案，想一起玩转游戏开发

欢迎关注我的公众号 

公众号搜索:玩转游戏开发

或扫码:<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/abd0c14c9c954e56af20adb71fa00da9~tplv-k3u1fbpfcp-zoom-1.image" alt="img" style="zoom:50%;" />

QQ 群: 1103157878

博客主页: https://ailhc.github.io/

掘金: https://juejin.cn/user/3069492195769469

github: https://github.com/AILHC
