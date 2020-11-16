---
layout: post
title: 这就是ansible系列之二（基础知识)
categories: [anisble, 自动化配置， 系统技术]
description: 本文介绍了ansible是什么以及如何实现批量系统配置、 批量程序部署、批量运行命令功能。
keywords: anisble, 自动化配
---

  从ansible系列之一的文章中， 我们知道ansible主要是实现了批量系统配置、 批量程序部署、 批量运行命令功能。 很多情况下， ansible一个命令， 就可以完成一系列的操作， 真正做到了高效、自动化。那么， ansible是如何做到这些的呢？

  首先让我们认识一下ansible的软件架构。

![img](https://mmbiz.qpic.cn/mmbiz_png/tzwqLkEsh8Mmqvib00zpqLhN4gh49NIHJicQubeSCribCq2xS4n90qgdTyjmic1lH0JR7oDIsf25eyb1dyFIcMSjhQ/640?wx_fmt=png)



  相关的模块的功能介绍如下：

| 模块名            | 功能                                   | 备注 |
| ----------------- | -------------------------------------- | ---- |
| HostInventory     | 主机清单                               |      |
| Playbooks         | 配置文件， 包含多个任务， 可自动化执行 |      |
| CoreModules       | 核心模块， 实现整个框架                |      |
| CustomModules     | 定制模块                               |      |
| Plugins           | 补充模块的功能                         |      |
| ConnectionPlugins | 连接器，用于连接主机                   |      |

  实际上， ansible自身是个框架， 相关的功能都是通过 CoreModules/CustomModules来实现的，而HostInventory/Playbooks/ConnectionPlugins等是用来描述远程执行的主机信息、连接信息及编排要执行任务的相关组件。 上图还描述了ansible不仅适用于传统的IDC里面的主机/集群，也适用于云计算时候的公有云/私有云的场景。

  那么， 实现了具体功能的CoreModules/CustomModules都有哪些呢？下面列举了一些常用的：

  

| 模块名  | 功能                                | 示例                                                         | 备注                                           |
| ------- | ----------------------------------- | ------------------------------------------------------------ | ---------------------------------------------- |
| command | 缺省模块，可以不用-m指定， 执行命令 | ansible <host> -a "chdir=/tmp ls" 在<host>上运程执行跳转到/tmp目录并执行ls命令 | 不能直接执行管道等特殊命令， 需要通过shell模块 |
| shell   | 执行shell命令                       | ansible <host> -m shell  -a "chdir=/tmp ls \|grep a" 在<host>上运程执行跳转到/tmp目录并执行ls命令并通过管道过滤出包含a的目录或文件 |                                                |
| user    | 执行用户？                          | -m user -a "name=xx state=xx password=xx"                    |                                                |
| cron    | 定期任务                            | -m cron -a "name=xx minute=xx job=xx"                        | job取值要加引号                                |
| copy    | 复制                                | -m copy -a "src= xx dest=xx"                                 | 可以使用相对路径或者绝对路径                   |
| yum     | 包管理器                            | -m yum -a "name=xx state=xx"                                 |                                                |
| fetch   | 从远程主机获取文件到本地            | -m fetch -a "src=xx dest=xx"                                 | 参数含义跟copy相反， dest是指本地路径          |
| file    | 创建或者查询目录或者文件            | -m file -a "path=xx mode=xx state={directory\|link\|present\|absent} src= " | directory会递归创建                            |
| ping    | 网络连通测试                        | -m ping                                                      | 不用参数                                       |
| service | 服务操作                            | -m service -a "name=xx state={started\|stopped\|restarted} enabled=xx" |                                                |



  具体的， 可以通过ansible-docl -l命令来查看可用模块的列表， 而模块的功能通用可以这样调用： 

ansible <host-pattern> <host-list>  -m module-name  -a args



  从整体上来看， ansible执行自动化部署的过程大概包括如下步骤：

a. 读取剧本playbook, 获取解析要执行的任务详情

b. 通过主机清单HostInventory 获取要执行的主机， 根据具体的任务调用相应的模块来执行任务

c. 通过连接器ConnectonPlugins连接对应的主机并推送待远程执行的任务列表

d. 远程执行任务中具体包含的命令

  



  ansible的基础知识我们就大概介绍这么多， 下一篇将从实践的角度上来认识下ansible。