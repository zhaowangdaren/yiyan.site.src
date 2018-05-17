---
title: Go语言的5大开源库
date: 2018-05-13 12:07:07
tags:
---
Go语言可以说是一个开箱即用的编程语言，其性能出众、支持分布式等特性深得程序员的喜爱。小编之前在毫无Go语言开发经验的情况下，只花了一个星期便掌握了Go语言的基本特性，后来也没花几天便搭建了一个数据库为mongoDB的服务器，并实现了相当复（chong）杂（fu）的功能，可见其上手非常简单。

下面介绍5个在GitHub上比较火热的Go语言开源工具库。
## Go kit：微服务工具包
Go kit微服务工具包为微服务提供了一系列的功能，使搭建微服务特别方便，开发人员只需要关注业务逻辑即可。
限于篇幅，这里列出Go kit提供的几项功能：
- jwt(JSON web token):一种认证通信双方的协议
- RPC（远程过程调用）基础支持
- grpc：Google开源RPC框架
- http服务
- LB负载均衡
- ZK（ZooKeeper）支持：分布式服务框架
- Graphite:支持监控系统
- InfluxDB: 时间序列数据库

## GORM
ORM是什么？对象关系映射（Object Relational Mapping）,是一种程序技术，用于实现面向对象编程语言里不同类型系统的数据之间的转换。通俗一点讲：操作数据库。
GORM具有很多功能，具其官网介绍，具备一下功能：
- 全功能ORM（无限接近）
- 关联（包含一个，包含多个，属于，多对多，多态）
- 钩子 (在创建/保存/更新/删除/查找之前或之后)
- 预加载
- 事务
- 复合主键
- SQL 生成器
- 数据库自动迁移
- 自定义日志
- 可扩展性, 可基于 GORM 回调编写插件
- 所有功能都被测试覆盖
- 开发者友好

## cli
[cli](https://github.com/urfave/cli)，可以用来快速的创建go语言命令行工具，目前在GitHub上的star数为8k+。

## vegeta
[vegeta](https://github.com/tsenart/vegeta)http负载测试工具。

## fuzzy
[fuzzy](https://github.com/sahilm/fuzzy),go语言实现的字符串模糊匹配工具。

## build-web-application-with-golang
除了上面提到的几个工具外，在GitHub上还有一个比较火热的项目，star数量达到了2万多。其实，称之为项目并不准确，应该说是博客，它从零开始，详细的介绍了如何一步步搭建Go语言开发环境、搭建web服务器、连接数据、部署等等，几乎一整套的Go语言建站方法，它就是:[build-web-application-with-golang](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/preface.md)。
它不仅有中文版，还有英文、法文、日文版等等多种语言。从介绍页面上，其包含浓浓热情、求赞助的支付二维码来看，这是国内开发者贡献的，点赞！