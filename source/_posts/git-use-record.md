---
title: Git使用记录
categories:
  - 经验之谈
  - 资料收集
tags:
  - Git
  - 学习
  - 记录
comments: true
abbrlink: 52800
date: 2020-02-27 21:04:35
img:
---
# Git 使用记录

## Git学习资源


## Git问题记录

### SSHKey
1. 怎么生成？
    ```
    ssh-keygen -t rsa -C "youremail"
    默认会生成 名字 id_rsa 的sshkey 
    可以自己自定义,但再使用时可能会有问题
    ```
2. 关于错误:ssh: Could not resolve hostname github.com: Name or service not known.fatal: Could not read from remote repository.
[参考解决方案:](https://www.cnblogs.com/niuniui/p/8783273.html)
3. 在Github中使用 
    ```
    Settings->SSHKey -> NewSSHKey
    将 id_rsa.pub(公钥) 文件的内容复制进去
    
    然后在git bash 执行:ssh -T git@github.com
    
    ```
4. "执行:ssh -T git@github.com" 出现Permission denied (publickey)[参考解决方案](https://www.cnblogs.com/lxwphp/p/7884700.html)

5. 如果在创建sshkey时使用自定义文件名, 执行 ssh-add ~/.ssh/<自定义sshkey名> 可能会出现：Could not open a connection to your authentication agent [参考解决方案](https://www.cnblogs.com/Security-Darren/p/4106328.html)

### 文件名大小问题
1. 缘由：原来的文件名是小写的文件，复制了大写的文件名文件，显示覆盖，但覆盖后，文件名还是小写的
2. 解决方案：
    1. 参考：[git 大小写问题 踩坑笔记](https://blog.csdn.net/u013707249/article/details/79135639)
    2. 在项目目录下执行命令：
        ```
        touch .gitconfig
        git config core.ignorecase false
        git config --global core.ignorecase false
        ```
### Git二进制文件管理以及如何git瘦身
#### 缘由
在游戏开发过程中，会使用用到很多二进制类资源，比如 图片，配置压缩文件，3d资产文件等，由于git的工作原理，不断的commit会不断的累加。整个仓库会变得非常庞大。
#### 二进制文件管理
1. 参考资料
  a. [Git 管理实战（五）：二进制大文件的版本控制](http://www.uml.org.cn/pzgl/201901233.asp)
  b. [Git LFS的使用](https://www.jianshu.com/p/493b81544f80)
  c. 百度 git lfs
#### git 瘦身实践
1. 参考资料
  a. [Git清理删除历史提交文件]()https://www.jianshu.com/p/7ace3767986a
  b. [使用BFG移除git库中的大文件或污点提交](https://www.awaimai.com/2202.html)
  c. [使用BFG清除git仓库中的隐私文件或大文件](https://www.cnblogs.com/huipengly/p/8424096.html)
  d. [为Git仓库瘦身](http://www.huamo.online/2017/11/22/%E4%B8%BAGit%E4%BB%93%E5%BA%93%E7%98%A6%E8%BA%AB/)
  e. [git rebase有哪些用法？](https://www.zhihu.com/question/25072850?sort=created) 这个可以对项目进行瘦身
2. 可以使用 gitlfs 来迁移仓库的管理方式而瘦身。但是这个会相对麻烦，需要远程仓库也支持。如果是使用gitee或者github等免费远程仓库则需要付费
3. 将二进制或者所有资源文件使用svn进行管理
  a. 先备份资源文件
  b. 忽略管理对应资源文件
  c. 删除资源文件的版本记录
  d. 同步操作到远程仓库
4. 定期对资源文件的版本记录进行清理
