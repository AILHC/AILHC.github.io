---
title: 打破CocosCreator3d不能使用npm包的魔咒!!!
categories:
  - EasyGameFramework使用
tags:
  - EasyGameFramework
  - CocosCreator3d
comments: false
abbrlink: 22225
date: 2020-11-30 22:36:20
img: https://coding-pages-bucket-434147-7588795-4574-367535-1255530080.cos.ap-guangzhou.myqcloud.com/img/破碎玻璃窗口.webp
---
## 背景

- 我在开发EasyGameFramework这个框架的时候，目标之一是构建出来的模块包可以在CocosCreator3D上使用，使用体验和正常使用具有类型声明的Npm包一致。实在不行再用发布源码的方式。

- 在js生态中npm有很多很强大很有用的模块包

- 论坛上也有小伙伴对在CocosCreator3d上使用npm包有需求。

- 但CocosCreator3D的模块机制和CocosCreator不同，所以暂不支持直接使用npm包。

  那怎么办呢？首先了解一下CocosCreator3d模块机制

## CocosCreator3d模块机制

这个时候就得祭出调试的杀手级工具:Chrome Devtools

### 调试运行CocosCreator3d空项目

**查看index.html**

![index.html](https://api2.mubu.com/v3/document_image/4fcfb451-ef2e-4510-858a-510dd3cdaf6d-416062.jpg)

CocosCreator3d中使用了system.js，并且在index.html注册了一个叫 code-quality:cr的模块

而且加载了一个.import-map.json的东西

**搜索.import-map.json看看是什么**

![.import-map.json](https://api2.mubu.com/v3/document_image/9c8ed51c-43b1-4e5c-a61e-f77aab96f3d5-416062.jpg)

通过以上的调试我们大概知道，CocosCreator3d使用了一个叫systemjs的库，然后呢，用来注册一些模块。这个systemjs可能是关键的东西，得了解一下。

> SystemJS的官方github:https://github.com/systemjs/systemjs
>
> 别人的使用:[SystemJS使用记录](https://www.jianshu.com/p/42c695316d56)

了解过后，我们能知道它就是一个模块加载器，以及它的注册方式

1. 通过一个map注册一堆模块
2. 通过register接口注册一个模块

但是怎么处理引用的呢？

### 断点调试systemjs

**找到systemjs，搜索register**

![image](https://api2.mubu.com/v3/document_image/eb5b31a3-2602-4832-8566-f8bf16809162-416062.jpg)

**搜索importMap之类的关键词**

![image](https://api2.mubu.com/v3/document_image/e74c451c-2395-4acd-a8f6-5fc22f505452-416062.jpg)

通过以上调试验证了我们的猜想：SystemJs，用来注册和加载模块的，是一种模块规范，需要做特殊处理才能注册进去和使用。但我们需要来写一个简单的systemjs规范的模块来验证一下。

## 手写一个简单的SystemJs的模块

* **simple-sysjs.js**

  ```js
  System.register('simple-sysjs', [], function (exports) {
  'use strict';
      return {
      execute: function () {
          var SimpleSysJs = exports('SimpleSysJs', /** @class */ (function () {
              function SimpleSysJs() {
                  console.log("我是简单的systemjs模块")
              }
              return SimpleSysJs;
                  }()));
              }
      };
  });
  
  ```

  

* 声明文件:**simple-sysjs.d.ts**

  ```ts
  declare module 'simple-sysjs' {
  export class SimpleSysJs {
      }
  }
  ```

* 为了保证这个脚本优先于其他脚本加载，将simple-sysjs.js设置为插件

* 然后在编辑器新建脚本，写下

  ```ts
  import { SimpleSysJs } from "simple-sysjs"
  new SimpleSysJs()
  ```

  

* 运行后可以看到输出:我是简单的systemjs模块

这个很简单的，大家都可以新建一个空C3d项目尝试一下。

经过验证尝试，我们知道，只要将npm包转成systemjs规范的文件就可以被我们的代码引用，以及正常使用了。但手写太麻烦了，有什么简单的办法呢？

## 如何快速地转换npm包为systemjs规范库

### 使用egf-cli

这个工具是我为EasyGameFramework打造的，目的是可以发布框架库为任意规范模块，给不同的引擎和项目使用。

有兴趣可以看看我之前写的文章

* [框架的诞生-一：我想要的框架](https://ailhc.github.io/2020/11/29/The-Birth-of-a-Framework-One-The-Framework-I-Want/)

以及框架仓库:https://github.com/AILHC/EasyGameClientFramework.git

**使用这个工具对npm包有一定的要求**

1. 符合commonjs规范
2. 最好有类型声明文件
3. 最好能是typescript开发的

**我以[xstate](https://xstate.js.org/)（一个高star的状态机库）为例子**

- c3d项目下安装 xstate ,npm i xstate

- 然后复制 EasyGameFrameOpen的package-template项目

- 修改包名为

  ```json
  "name": "@ailhc/xstate2c3d",
  ```

  

- 修改构建命令为

   ```json
  "build:system": "egf build -f system:@ailhc/xstate2c3d",
   ```

  

- 删除其他ts文件，留下index.ts

- 然后写一句:

  ```ts
  export * from "xstate"
  ```

  

- 最后执行 yarn run build:system

- 最后的最后，因为xstate里引用了node的变量，所以在构建出来的index.js要加一句

  ```js
  sendParent: sendParent,
  sendUpdate: sendUpdate,
  spawn: spawn
        });
  var process = { env: { NODE_ENV: "production" } }
  ```

  

- 复制lib和types到c3d项目，将index.js设置为插件

- 在任意脚本引用就可以

  ```ts
  import { createMachine, interpret } from "@ailhc/xstate2c3d"
  // Stateless machine definition
  // machine.transition(...) is a pure function used by the interpreter.
  const toggleMachine = createMachine({
    id: 'toggle',
    initial: 'inactive',
    states: {
      inactive: { on: { TOGGLE: 'active' } },
      active: { on: { TOGGLE: 'inactive' } }
    }
  });
  
  // Machine instance with internal state
  const toggleService = interpret(toggleMachine)
    .onTransition((state) => console.log(state.value))
    .start();
  // => 'inactive'
  
  toggleService.send('TOGGLE');
  // => 'active'
  
  toggleService.send('TOGGLE');
  // => 'inactive'
  ```

  **xstate2c3d**的工程已经更新到EasyGameFrameOpen：https://github.com/AILHC/EasyGameFrameworkOpen.git

### 其他方式

1. 如果npm包有构建systemjs，直接使用
2. 如果没有构建systemjs，看是否有typescript源码，copy源码，使用egf-cli构建一个systemjs规范的js
3. 如果是全局变量的库，直接设置为插件就可以了。

## CocosCreator3.0?
它也许会支持npm包，但等待它的到来可能还有一段时间

而且研究一下CocosCreator3d的模块加载机制说不定，在遇到看不懂的问题时有所头绪呢。

谢谢大家阅读我的文章，希望大家能有所收获。



## 框架开发系列文章



* [框架的诞生-零：为什么写框架？](https://ailhc.github.io/2020/11/17/The-Birth-of-Frames-Zero%EF%BC%9AWhy-write-framework/)

* [框架的诞生-一：我想要的框架](https://ailhc.github.io/2020/11/29/The-Birth-of-a-Framework-One-The-Framework-I-Want/)

* [打破CocosCreator3d不能使用npm包的魔咒!!!](https://ailhc.github.io/2020/11/30/Break-the-spell-that-CocosCreator3d-cannot-use-NPM-packages/)

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