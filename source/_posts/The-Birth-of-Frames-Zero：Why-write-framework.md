---
title: 框架的诞生-零：为什么写框架？
categories:
  - 框架设计
tags:
  - EasyGameFramework
  - 框架设计
comments: false
abbrlink: 6239
date: 2020-11-17 15:53:39
img: "https://coding-pages-bucket-434147-7588795-4574-367535-1255530080.cos.ap-guangzhou.myqcloud.com/img/九十度鞠躬谢谢.jpg"
---
[TOC]



## 题外话

------

大家好，很高兴，能写这篇文章分享给你们看，也很感谢你们能看我的文章。
如果能和你们交流最好了。
做游戏开发3、4年了，我用过这些，Unity，Cocos2dx，CocosCreator，LayaAir，Egret。

用得最久的是LayaAir，因为工作需要嘛。

但最喜欢的还是CocosCreator，因为社区的小伙伴、引擎组的人都很好很可爱，他们分享的东西都让我受益匪浅。

谢谢~
<img src="https://coding-pages-bucket-434147-7588795-4574-367535-1255530080.cos.ap-guangzhou.myqcloud.com/img/九十度鞠躬谢谢.jpg" style="zoom:50%;" />

第一次写文章，如有不当之处请多多见谅。

## 什么是框架？

------

我想有的人心里有答案，有的人也有疑问。

这里我提供别人对框架的见解链接:

* 某乎文章:[什么是框架？为什么要使用框架？我告诉你理由哦！](https://zhuanlan.zhihu.com/p/114465098)

  ```
  框架的英文为Framework意思是框架、机制、准则。
  最早是源于建筑行业，是一个框子——指其约束性，也是一个架子——指其支撑性。
  是一个基本概念上的结构，用于去解决或者处理复杂的问题。
  ```
  
* @金戈的回答: [什么是框架？](https://www.zhihu.com/question/19558524)
  
  ```
  框架（Framework）是一个框子——指其约束性，也是一个架子——指其支撑性。
  IT语境中的框架，特指为解决一个开放性问题而设计的具有一定约束性的支撑结构。在此结构上可以根据具体问题扩展、安插更多的组成部分，从而更迅速和方便地构建完整的解决问题的方案。
  ```

  


我的个人见解：▼

![](https://coding-pages-bucket-434147-7588795-4574-367535-1255530080.cos.ap-guangzhou.myqcloud.com/img/GameApplication.png)

![](https://coding-pages-bucket-434147-7588795-4574-367535-1255530080.cos.ap-guangzhou.myqcloud.com/img/GameApplication-Framework.png)


我们的游戏程序都是基于某个平台的，以及可能会使用现成的渲染框架，来实现我们游戏的玩法和业务。同时针对开发和生产环境部署，我们需要一些工具协助。

框架在渲染框架层和业务层之间，封装部分通用能力供业务层使用。起到支撑业务开发的作用。

我的层级图的灵感来自白玉无冰大佬拍的panda大大讲的一页一个PPT:CocosCreator跨平台的引擎架构。▼

<img src="https://coding-pages-bucket-434147-7588795-4574-367535-1255530080.cos.ap-guangzhou.myqcloud.com/img/CocosCreator跨平台架构.jpg" style="zoom:67%;" />

对于游戏客户端框架来说

- 框架具有一定的约束性，指的是，我们需要在一定范围内按照框架作者的设计和规范去使用和扩展。

  比如：

  在CocosCreator框架中。

  * 如果要加载一个资源，那就需要调用cc.assetManager.loadxxx 或者cc.loader.loadxxx，传规定的参数，传错了就可能出问题。

  * 如果需要加载自定义的资源就需要安装assetManager的规范去扩展。具体可参考CocosCreator的文档，不按规范扩展就可能出问题。

- 框架有部分已经实现的功能，可以直接使用或者稍微扩展就可以用来实现业务逻辑。

- 大部分游戏客户端框架都包含很多功能模块，甚至不是我们项目需要的。

  比如：

  框架中包含UI管理框架，事件通讯框架，网络模块，等等。

  我们可能只需要事件通讯模块或者UI管理模块，只能手动去剔除不需要的。

- 框架所能解决的问题有限，有边界。

  比如：
  
  有的框架只是一个UI管理框架只解决了复杂UI管理的问题

* 框架可能跟底层强相关。

  比如：一个基于CocosCreator的UI管理模块，里面耦合了CocosCreator的加载，prefab文件，resource文件夹的规范等，可能也耦合了cc.Node的使用等等。

## 框架解决什么问题？

------

大家对框架的第一印象可能是

* 我学不动了<img src="https://coding-pages-bucket-434147-7588795-4574-367535-1255530080.cos.ap-guangzhou.myqcloud.com/img/学不动了.jpg" alt="学不动了" style="zoom: 50%;" />

* 提高开发效率，快！！！<img src="https://coding-pages-bucket-434147-7588795-4574-367535-1255530080.cos.ap-guangzhou.myqcloud.com/img/疯狂打字.jpg" style="zoom:50%;" />

我个人觉得好坏还得看框架和看个人

* 好的框架可以统一多个开发人员的编程风格，方便多人协作
* 好的框架可以让开发人员减少维护程序架构心智负担，减少纠结。
* 最重要的是可以大大的提高开发效率，大部分时间专心写业务就可以了。

## 为什么写框架？

------

有现成的框架为什么不拿来直接用，干嘛自己造轮子浪费时间？

可能是现成的框架没法满足业务需求

可能是现成的框架和自己的思想不符

可能是现成的框架过时了

等等

对于我而言

写框架可以督促我去学习其他框架是如何设计的，为什么这样设计，能解决什么问题，为什么能解决这些问题。

然后反过来去思考我工作和开发中遇到的问题，我学着去分析，去尝试找解决方案，以及思考如果我写框架怎么解决这些问题。怎么去设计。

写出来的框架很挫怎么办？怕个毛线，继续学习，继续思考，然后解决问题，大不了推翻重写。

写出来的框架不能用在公司的项目中去实践怎么办？这个想法有点危险，哈哈哈，可以自己写项目来验证。不过还有另外一种解决方案。下一篇会讲。

写框架是一场历练，督促着我去学习和沉淀所学，打磨我的知识体系，让它更加完善。



## 总结

------



- 什么是框架？

  - 看游戏程序架构图
    - 框架层图

  - 他人的见解参考

    - 某乎文章: [什么是框架？为什么要使用框架？我告诉你理由哦！](https://zhuanlan.zhihu.com/p/114465098)

    - 某乎问答: [什么是框架？](https://www.zhihu.com/question/19558524)

  - 我的见解（对于游戏客户端框架）

    - 框架具有一定的约束性，指的是，我们需要在一定范围内按照框架作者的设计和规范去使用和扩展。

    - 框架有部分已经实现的功能，可以直接使用或者稍微扩展就可以用来实现业务逻辑。

    - 大部分游戏客户端框架都包含很多功能模块，甚至不是我们项目需要的

    - 框架所能解决的问题有限，有边界

  - 框架可能跟底层强相关

- 框架解决什么问题？

  - 可以统一多开发人员的风格（框架的风格）

  - 可以让开发人员减少维护程序架构的心智负担，减少纠结（按照框架的思想来就行）

  - 可以大大提高开发效率，大部分时间专心写业务逻辑就可以了

- 为什么写框架？

  - 去学习其他框架怎么设计，怎么解决它们面对的问题。

  - 解决自己工作和开发中遇到的问题

  - 积累和沉淀自己的知识

## 一些游戏客户端框架参考

------



* [CatLib](https://github.com/CatLib) 一个Unity的渐进式框架(ps：我的框架灵感之一)
* [U3d网络游戏架构设计](https://gitbook.cn/gitchat/column/5a3921aec5896e6e1cf1a129 ) 一个大佬的GitChat专栏，需要订阅
* [腾讯学院的手游核心技术实战](http://gameinstitute.qq.com/lore/index/10017) 这是腾讯学院的一个贪吃蛇大作战的一个游戏开发课程，讲到了如何设计和实现基于Unity的游戏框架
* [GameFramework](https://gameframework.cn/) 基于Unity的一个完善的框架
* [QFramework](https://qframework.cn/intro) 基于Unity的一个完善且扩展性非常强的框架，而且作者有很多关于框架设计的理念，非常棒
* [UNITE －Unity项目架构设计与开发管理](https://v.qq.com/x/page/d016340mkcu.html)
  * 文章：[Unity项目架构设计与开发管理](https://zhuanlan.zhihu.com/p/64858713)
* 【ituuz分享-框架】[lightMVC](https://forum.cocos.org/t/ituuz-lightmvc-for-cocos-creator/80678):轻量级游戏开发框架(for cocos creator)
* [GameplayFrameWork for CococsCreator](https://huangx916.github.io/2019/01/01/gameplayframework/) 
* 其他的还有很多，论坛搜 "框架"即可

## 心里话

------

我想和优秀的小伙伴一起开发好玩的游戏

我希望能通过我做的游戏，我的能力获得用于生活和学习的报酬。

我也希望

有人能从我的游戏中获得快乐，或者有所收获

也希望

有人能因我的分享而有所收获

然后我能说一句

![](https://coding-pages-bucket-434147-7588795-4574-367535-1255530080.cos.ap-guangzhou.myqcloud.com/img/《头号玩家》谢谢你玩我的游戏.jpeg)





**谢谢你玩我的游戏**

**谢谢你信任我**

**谢谢你用我的框架**

## 预告

------

框架系列文章

- [框架的诞生-零：为什么写框架？](https://ailhc.github.io/2020/11/17/The-Birth-of-Frames-Zero%EF%BC%9AWhy-write-framework/)
- 框架的诞生-一：我的框架
- 框架的诞生-二:  承载万千的核心
- 不只是UI管理:通用显示管理
- 让fairygui更好用的插件
- 满足多种需求的通用对象池
- 构建游戏/应用的神器:broadcast
- 满足所有自定义需求的通用socket网络模块
- 业务开发总结之状态管理
- 。。。

## 最后

------

欢迎关注，更多内容持续更新

公众号搜索:玩转游戏开发

或扫码:<img src="https://coding-pages-bucket-434147-7588795-4574-367535-1255530080.cos.ap-guangzhou.myqcloud.com/img/20201128164023.png" alt="img" style="zoom:50%;" />

QQ群: 1103157878

博客主页: https://ailhc.github.io/

github: https://github.com/AILHC