---
title: 解决CocosCreator2.4.x的ABundle不能使用npm包的问题
categories:
  - EasyGameFramework使用
  - CocosCreator
tags:
  - npm
  - CocosCreator
  - EasyGameFramework
comments: false
abbrlink: 55699
date: 2020-12-21 20:00:09
img:
---
## 前言
今天逛论坛时，我看论坛上有个小伙伴问cocoscreator能不能用npm包？

帖子:[cocos creator 工程里面能装npm包吗？](https://forum.cocos.org/t/topic/102196)

我在想怎么可能不支持呢?

于是点击去看到了大佬的回答，也仔细看了大佬写的了一篇AssetBundle的讲解文章
> [Creator | Asset Bundle 全解析](https://forum.cocos.org/t/creator-asset-bundle/99886)

这里说npm包都打进了main这个bundle了，分包里没法引用

这句话让我不禁想了一下，[EasyGameFramework](https://github.com/AILHC/EasyGameFrameworkOpen/)的所有库都是以npm包的方式安装使用的。

如果不能在分包引用，那不就很烦？

我用egf-ccc-full的项目做了一个测试，发现预览可以，构建出来真的不行，加载bundle就直接报错了。

后来我尝试挂载到全局，可以正常运行，但编码体验下降一百倍，过程繁琐。

我忍不了了

之前就发现一个跟npm包相关的问题：不支持引用项目文件夹外的npm包，编辑器编辑和构建会报错（但预览不会）
那个问题我已经解决了:[CocosCreator2.4.2/2.4.3无法编译引用了项目文件夹外部的npm模块](https://forum.cocos.org/t/topic/101523)

所以我觉得这两个问题应该是差不多的。

今天我就让ABundle支持引用npm包！！！

@一下引擎组

## 问题重要吗?
我觉得很重要!

虽然说可以将npm包的东西挂载到全局，然后在分包里的脚本，去全局取用。

但这样不安全。控制台也可以访问到了。

而且最重要的是，很别扭，你知道吧？这编码体验很不爽了。

EasyGameFramework的出发点就是要让编码更加Easy，更加爽。

所以
今天我就让ABundle支持引用npm包！！！
今天我就让ABundle支持引用npm包！！！
让EasyGameFramework无缝支持分包使用

## 复现问题
flag已经立了，说明我已经解决了。哈哈哈哈哈哈~ 它不可能倒下的~

我们先把问题复现出来，看看是怎么回事。

只有构建之后才能看出问题，所以先构建一波，这里我使用egf-ccc-full来测试。

> https://github.com/AILHC/EasyGameFrameworkOpen/tree/main/examples/egf-ccc-full

大家可以克隆下来看看

### 我简单讲一下我的测试逻辑
1. 加载abtest这个包：test/display-ctrl/abtest，然后将这个包里的ABTestModule注入模块管理器
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/315428c0dee449afb608613d99bc763f~tplv-k3u1fbpfcp-watermark.image)
2. 通过abtest模块的逻辑，显示abtest里的界面
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/78a3c0f4bb0642f29e216bd480bae818~tplv-k3u1fbpfcp-watermark.image)
3. ABTestModule注入全局
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/38cbc87385f24ca9aa502cce06cc88c3~tplv-k3u1fbpfcp-watermark.image)
4. ABTestView继承@ailhc/dpctrl-ccc包的NodeCtrl
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8d794174bea04e46baaf1514ddfe97b5~tplv-k3u1fbpfcp-watermark.image)
### 运行构建后的看看报错
找不到模块，这个报错的模块名有点奇怪啊，不是我的包名@ailhc/dpctrl-ccc,被切了
![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7b90797c4d214f54afd1c135fb9cf14b~tplv-k3u1fbpfcp-watermark.image)
### 万能的断点调试
1. 我们会发现，在分包的index.js找不到，只有两个包内的脚本模块对象
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/70d998f46ac245c08b04b63473ef061b~tplv-k3u1fbpfcp-watermark.image)
2. 下一步。
![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1d93829656fc4045af67d75051e99f73~tplv-k3u1fbpfcp-watermark.image)
	
    然后它把包名切割了！！！！就这样就切割了？？？！！！为什么？
    给不懂的科普一下npm包名
    	· npm包名是没有./或者../的，大多是只有一个名字 比如xstate
		· 有一些特殊的包名比如:@babel/core，比如@ailhc/egf-core	

3. 继续跟踪，它进了一个require函数，然后跳转过去，发现这个index.js是main bundle里的
通过切割后的名字dpctrl-ccc到main包里也没找到
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a20d688063e4240899687d136e879e4~tplv-k3u1fbpfcp-watermark.image)
其实main包里应该是有这个包的，只是key不对。
我通过调试main index.js中对@ailhc/dpctrl-ccc的引用其实是可以找到的，只是它的key 转化为4

### 调试总结
通过这系列调试，我们可以知道
* 主包main里的index.js有npm包模块，也可以引用
* 分包里的index.js如果引用了包外的脚本模块，则会向上main包查找，查不到就报错
* 而这个分包向主包查找模块的逻辑里，没考虑到npm包名的情况
* 主包和分包里的index.js的开头那部分都很像，像是模板

总结完之后，我们就去根据我们调试的成果去尝试解决问题

## 如何来解决这个问题呢？
### 首先得找到模板index.js开头
通过进入编辑器目录，搜索文本__require找到了
CocosDashboard\resources\.editors\Creator\2.4.2\resources\static\_prelude.js
```ts
(function e(t, n, r) {
    function s(o, u) {
        if (!n[o]) {
            if (!t[o]) {
                var b = o.split('/');
                b = b[b.length - 1];
                if (!t[b]) {
                    var a = "function" == typeof __require && __require;
                    if (!u && a) return a(b, !0);
                    if (i) return i(b, !0);
                    throw new Error("Cannot find module '" + o + "'");
                }
                o = b;
            }
            var f = n[o] = {
                exports: {}
            };
            t[o][0].call(f.exports, function (e) {
                var n = t[o][1][e];
                return s(n || e);
            }, f, f.exports, e, t, n, r);
        }
        return n[o].exports;
    }
    var i = "function" == typeof __require && __require;
    for (var o = 0; o < r.length; o++) s(r[o]);
    return s;
})

```
很像啊哈

简直就是🙄

然后我简单的加了一句log，发现重新构建之后输出出来了，说明改这个有效

### 然后呢，解决那个切掉我npm包名的逻辑
先判断路径是什么类型的，再决定切不切

```ts
var b = o;
if (o.includes("./")) {
//内部代码
b = o.split("/");
b = b[b.length - 1];
} else {
	//npm包代码
}

```

### 最后呢，因为主包中，npm包包名和模块对象的对应关系被转换了
所以呢

我们在第一次引用的时候，判断是不是npm包，如果是，就以包名为key，记录一下，下次好用包名查找

```ts
(function e(t, n, r) {
    function s(o, u, npmPkgName) {
        if (!n[o]) {
            if (!t[o]) {
                var b = o;
                if (o.includes("./")) {
                    //内部代码
                    b = o.split("/");
                    b = b[b.length - 1];
                } else {
                    //npm包代码
                }
                if (!t[b]) {
                    var a = "function" == typeof __require && __require;
                    if (!u && a) return a(b, !0);
                    if (i) return i(b, !0);
                    throw new Error("Cannot find module '" + o + "'");
                }
                o = b;
            }
            var f = n[o] = {
                exports: {}
            };
            t[o][0].call(f.exports, function (e) {
                var n = t[o][1][e];
                //判断是不是npm包，是就传包名
                return s(n || e, undefined, !e.includes("./") ? e : undefined);
            }, f, f.exports, e, t, n, r);
        }
        //判断是不是npm包，是就用包名作为key存一下模块引用
        if (npmPkgName && n[o] && !n[npmPkgName]) {
            n[npmPkgName] = n[o];
        }
        return n[o].exports;
    }
    var i = "function" == typeof __require && __require;
    for (var o = 0; o < r.length; o++) s(r[o]);
    return s;
})

```
重新构建，运行，没有报错~
搞定散花~
![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cc67890c8ab042f0ac8dd6df3b5c73a5~tplv-k3u1fbpfcp-watermark.image)

## 重要提示
虽然说搞定了，但它还是有局限性的

比如，**你主包没有引用过npm包，那么你分包也引用不了**

## 最后



欢迎关注我的公众号，更多内容持续更新



公众号搜索:玩转游戏开发



或扫码:<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/abd0c14c9c954e56af20adb71fa00da9~tplv-k3u1fbpfcp-zoom-1.image" alt="img" style="zoom:50%;" />



QQ 群: 1103157878



博客主页: https://ailhc.github.io/

掘金: https://juejin.cn/user/3069492195769469

github: https://github.com/AILHC