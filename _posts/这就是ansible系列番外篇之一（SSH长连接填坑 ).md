---
layout: post
title: 这就是ansible系列番外篇之一（SSH长连接填坑)
categories: [ssh]
description: 本文介绍了ansible实践时遇到的ssh异常问题的排错过程。
keywords: ssh, 排错
---

​    本来计划着手开始实践ansible操作的， 准备了3台Centos7u5的机器， 系统环境为python 3.7.4， ansible版本为2.10.3,  哪知道出师不利， 配置好了ansible.cfg及inventory后， 执行一下ping就异常了。下面将尽量还原具体填坑的过程。

  先来看一下折腾了几次之后的配置文件，如下：

![img](https://mmbiz.qpic.cn/mmbiz_png/tzwqLkEsh8PO1GIE6n4AFPicPBtIYA0cmITDAakxYfIngLTCIxn9J8wicBWVZ6rURhEsa7MkV4EBQeDliaFmQyy4g/640?wx_fmt=png)

  由于是pip安装的ansible, 缺省不会自动生成ansible.cfg， 就手动创建了。另外， 还创建的主机清单文件，如下：

![img](https://mmbiz.qpic.cn/mmbiz_png/tzwqLkEsh8PO1GIE6n4AFPicPBtIYA0cmwHrhNibstXBZxkDAoM378uUxVmUJ6p3jyB7AOulz9kUI3fF86r3ehHQ/640?wx_fmt=png)

 一开始的时候没有配置ssh连接的具体选项， 使用的是帐户加密码的认证方式，   执行ping命令 

```
ansible nginx_cluster -m ping 
```

时， 出错了， 错误如下：

![img](https://mmbiz.qpic.cn/mmbiz_png/tzwqLkEsh8PO1GIE6n4AFPicPBtIYA0cmhknQmlZVL828B9WYUzsdyUia77dW0TcvzQF1NEVhfVHBl34ZRoGAtCA/640?wx_fmt=png)

具体的出错信息就是： 

```
command-line line 0: Bad yes/no/ask argument.
```

看到yes/no， 灵机一动， 想起了ssh连接新机器时，也会有一个提示，要输入yes/no的， 大概如下：

![img](https://mmbiz.qpic.cn/mmbiz_png/tzwqLkEsh8PO1GIE6n4AFPicPBtIYA0cmVicjeTsHjoIwgUIgg4B0M51nlfoGhhLTqZTbsibeqHH19YvpicAObHlLA/640?wx_fmt=png)

于是就顺着如何跳过首次连接时的命令提示的方向去定位和解决问题了， 成功地把自己带到了坑里。

  首先尝试了采用ssh公钥的方式， 相关的配置如下：

```
ssh-keygen  ssh-copy-id -i ~/.ssh/id_rsa.pub <remote-host-name>
```

  然后在shell里面测试了一下，确实可以免钥直接登录成功。一阵开心， 心想这就问题应该O了， 把ansible.cfg及inventory调整了一下， 再次执行ping命令， 发现错误依然存在的。

  然后就去折腾ssh自身的连接配置，包括在ansible.cfg里面指定ssh_conection的相关选项， 最主要的就是首次连接不要检查主机指纹StrictHostKeyChecking设置为no， 再次测试ping命令，发现错误依然存在。

  拿着返回的错误提示， 去google了一下， 没有找到该错误直接关联的有效信息。找到了线索， 加上-vvv， 进行具体的debug。 交互中的重要信息如下：

![img](https://mmbiz.qpic.cn/mmbiz_png/tzwqLkEsh8PO1GIE6n4AFPicPBtIYA0cmdWrgDu4JNZOI9ug5dv9eDZWFJenY1xrneibuM4EOpog3fEwdbYnKFOA/640?wx_fmt=png)

```
<host-name> SSH: EXEC ssh -C -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking=no -o KbdInteractiveAuthentication=no -o PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey -o PasswordAuthentication=no -o ConnectTimeout=10 -o ControlPath=/home/xx/.ansible/cp/472fc29db8 <host-name> '/bin/sh -c '"'"'echo ~ && sleep 0'"'"''<xx-hostname> (255, b'', b'command-line line 0: Bad yes/no/ask argument.\r\n')
```

可以看到， ssh连接的时候是带了一些选项的： 

```
ssh -C -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking=no
```

于是在shell里面单独加上这些选项执行ssh时， 果然复现了同样的错误。

  接下来就是逐个选项测试， 看一下哪些选项影响到了。最后试下来发现是ControlMaster及ControlPersist选项， 查了一下资料， 这个选项是提供了SSH长链接的功能， 该功能可以在断开终端连接时实现类似TCP长连接那样在一段时间内保持会话，下次连接的时候可以复用，从而达到节省时间的效果。

  至此，问题总算有些眉目了。但是为什么ControlMaster等选项不生效呢？又继续挖下去中，发现是openssh版本的原因， ControlPersist选项（估计包括ControlMaster auto）都是在openssh 5.6之后才有的。而当前机器ssh使用的openssh版本为：

```
(base) bash-4.1$ ssh -VOpenSSH_3.9p1, OpenSSL 1.0.0-fips 29 Mar 2010
```

  所以自然就出现了上面的问题。 

  总结一下，发现openssh不同版本的差异还是蛮大的， 包括ControlMaster的取值等等， 下次遇到一定要非常留意。 问题定位到了，下一步就是升级openssh版本了， 这里暂时不再详述了。