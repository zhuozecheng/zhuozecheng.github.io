---
layout: post
title: template page
categories: [cate1, cate2]
description: some word here
keywords: keyword1, keyword2
---

  ansible属于自动化配置工具的领域， 已经被Redhat收购(官网https://www.ansible.com/）。 

  自动化配置工具解决的是如何将软件服务高效可靠地部署到数量众多的服务器上的问题， 简而言之就是批量部署软件服务。 如果不使用自动配置工具， 很自然地，我们通常会先手动在单机上安装一遍软件， 再写成自动化地脚本， 再一台机器一台机器的初始化环境及安装相关的依赖，再把相关的软件包上传到每台服务器上， 在上面执行前面写好的自动化脚本。 服务器数量在3台以内这个工作量看起来还是可以接受的， 但如果有十几台、甚至几十、上百、上千台就几乎不大可能了。自动化配置工具的出现就是来解决这样的问题。

  当前在自动化配置工具领域业界有几款软件都是比较常用的，如下图所示：

![img](https://mmbiz.qlogo.cn/mmbiz_png/tzwqLkEsh8Phk2bJ84xic1fHYkibGMjZL2nME455dSJ5BZ0nBM5TD9ibVcuujoFnOavtsFJuIAyJ5oDq3nJqM8ZqA/0?wx_fmt=png)

  最近想基于k8s集群搭建大数据实验平台， 规划了5台物理机， 想着使用自动化配置工具来完成批量的软件部署， 所以简单地调研了一下， 最好决定使用ansible。 相关软件选型的考虑请看下文分析。

  

![img](https://mmbiz.qlogo.cn/mmbiz_png/tzwqLkEsh8Phk2bJ84xic1fHYkibGMjZL2vPIwMdCEhNJ7pWyPNib39dV6obpB77y5D8gNlica1F2Nia1rcne3VJ46Q/0?wx_fmt=png)

  首先从架构来看， 无client端的会简化一点， 少一些麻烦， 从这个角度上来看， puppet及chef减1分； 从直接的协议来看， ssh协议支持通过密码和密钥认证， 也是非常方便地实现从一台服务器远程操作其它服务器的， 协议上来看都是可以提以接受的； 从实现语言来看， ruby远没有python友好， 这在二次开发相关功能插件时， 会有比较明显的效率上的差异， puppet及chef再减1分。

  那在ansible及saltstack之间如何选择呢？ 看相关的分析数据， saltstack部署效率会更高、机器数量比老版本的ansible也支持得多， 但从github上各自项目的star及贡献者及引用的项目来看， ansible明显是优于saltstack，意味着社区支持ansible会更好，saltstack减1分； 然后在网上也看到有网友吐槽

saltstack 文档复杂， 概念很多， 细微差别指令多， 酌情再减0.5分（自己还没有深度试用)。 其它的必要的项目的稳定性、可扩展性等， 都是差不多的。

  另外， 本次主要解决的是使用ansible来搭建k8s集群， 发现github上人已经贡献了类似的项目https://github.com/easzlab/kubeasz， 有利于我快速上手解决当前的问题， ansible加1分。

  综合上述， 在自动化配置工具的选型上最终选用了ansible。

