title: Linux Shell执行远程命令
date: "2014-07-30 18:06:10"
tags: Linux
---
转自[http://www.cnblogs.com/ilfmonday/p/ShellRemote.html](http://www.cnblogs.com/ilfmonday/p/ShellRemote.html)

###shell远程执行：

　　经常需要远程到其他节点上执行一些shell命令，如果分别ssh到每台主机上再去执行很麻烦，因此能有个集中管理的方式就好了。一下介绍两种shell命令远程执行的方法。

###前提条件：

　　配置ssh免密码登陆参考 [Linux/UNIX下使用ssh-keygen设置SSH无密码登录](http://blog.csdn.net/leexide/article/details/17252369) ( 若没配置，则在执行脚本时需要输入密码 )

###对于简单的命令：

　　如果是简单执行几个命令，则：

```
ssh user@remoteNode "cd /home ; ls"
```

　　基本能完成常用的对于远程节点的管理了，几个注意的点：

* 双引号，必须有。如果不加双引号，第二个ls命令在本地执行
* 分号，两个命令之间用分号隔开

###对于脚本的方式：

　　有些远程执行的命令内容较多，单一命令无法完成，考虑脚本方式实现：

```
#!/bin/bash
ssh user@remoteNode > /dev/null 2>&1 << eeooff
cd /home
touch abcdefg.txt
exit
eeooff
echo done!
```

　　远程执行的内容在“<< eeooff ” 至“ eeooff ”之间，在远程机器上的操作就位于其中，注意的点：

* << eeooff，ssh后直到遇到eeooff这样的内容结束，eeooff可以随便修改成其他形式。
* 重定向目的在于不显示远程的输出了
* 在结束前，加exit退出远程节点

