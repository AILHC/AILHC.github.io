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