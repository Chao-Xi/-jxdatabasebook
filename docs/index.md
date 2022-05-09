# **手摸手Databases全家桶技术与实战教程**

> Started at 15th Jan.2021 By Jacob Xi
> 
> MySQL, Aurora, MongoDB, PostgresSQL, AWS Neptune, AWS DynamoDB

## **JX's chit-chat**

> Hi, this part is fucingk new and freshing🤔

Well, believe it or not, today is 9th May 2022. I have been already locked down for stupid COVID-19 quarantine and isolation for the fucking 52 days from mid-March.  Saw too many unbelievable and fucking magical realism in Shanghai these days. Dont tell me "C'est la vie". Life should not be falling and yielding like this. I'm a peaceful man but I'm already outrageous for such a long time, really tired, really...

> 说点开心的🤠

* 封闭的将近两个月，每天可以做到长跑15km, 似乎减肥了，似乎😑
* 看了几部不错的创业剧，有讲 Travis Kalanick Uber CEO 的《Super Pumped: The Battle For Uber》，故事重现Uber过山车式的兴衰、引起诸多后果的内外交战，体现硅谷的起起伏伏。讲Adam Neumann Wework CEO 的《WeCrashed: The Rise and Fall of WeWork》，讲述共享办公室公司WeWork的崛起以及衰落。在不到十年内，WeWork从一个共享办公空间发展成为一个470亿美元的全球品牌。然后又在短短的一年之内，损失了400亿美元。 还有讲 Theranos CEO Elizabeth Holmes《The Dropout》利欲熏心下Elizabeth开始不停造假，一度推出了根本没法提供正确数据的验血机；最终一切被拆穿后，Elizabeth现正面临刑事指控，而不认罪的她可能被判长达20年的刑期。 不得不说 Super Pumped 和 WeCrashed 的质量更高， nice shows 🤠
* 另外 《月光骑士》不错， 《slow horse》不知道在讲啥， 《HALO》垃圾，白瞎了那么好的制作，白左去死。 The last but not least, 《大侦探波洛》真的是传世经典
* 两本书，最近看书有点少 东野圭吾的《长长的回廊》，尼尔史蒂芬森的《雪崩》, 超源域真是好东西
* 又开始看最新的 《巫师8》，我们的猎魔人又和女术士搞到一起了
* 大事情要发生了，感谢自己的这一年的努力

## 内容简介

本书经过了漫长的，超过一年半的编写。终于在2022，五月中旬完成了阶段性的收尾工作。 本书共9章（Temp) 。 其中五章介绍MySQL, 两章介绍MongoDB, 一章PostgresSQL, 一章Aurora, 一章AWS DBs(Neptune+ DynamoDB)

* MySQL 基础篇: 介绍了MySQL的底层逻辑 隔离，索引，并发控制，锁,B+，树，事务，MVCC
* MySQL 实战篇(面试用): 从实战角度优化MYSQL, 数据类型, 表设计，索引优化，查询优化，高可用架构，分布式架构，实战面试
* AWS Aurora: 介绍了AWS aurora的基础知识，底层存储和基本操作
* MySQL on K8S
* MySQL 基础操作篇(操作指南)： 操作实例，数据约束，索引，用户管理，日志，备份管理，主从复制，GTID
* MongoDB操作实战：安装，文档操作，聚合操作，聚合高级操作，视图，索引
* MongoDB高级实战：**MongoDB 再入门， 从熟练到精通的开发之路， 片集群与高级运维之道， 集中实战掌握框架构建之法**
* PostgresSQL 12.6运维(实战)：链接管理&权限管理，常用命令，系统架构，备份恢复，MS复制，系统函数
* AWS DB others:  Amazon Neptune 图数据库技术,  分布式键值存储 DynamoDB 

![Alt Image Text](./images/bg1_1.jpeg "body image")

### **Previous on my Technolog book**

> [手摸手 Jenkins 战术教程 (大师版）](https://chao-xi.github.io/jxjenkinsbook/)
> 
> [手摸手 Elasticsearch7 技术与实战教程](https://chao-xi.github.io/jxes7book/)
> 
> [手摸手 Redis 技术与实战教程](https://chao-xi.github.io/jxredisbook/)
> 
> [手摸手 Chef & Ansible 技术与实战教程](https://chao-xi.github.io/jxchefbook/)
> 
> [手摸手 分布式与流式系统 (In progress)](https://chao-xi.github.io/jxdmsbook/)
> 
> [Azure 103&900 Tutorial (In progress)](https://chao-xi.github.io/jxazurebook/)
> 
> [手摸手 Linux Performance & 面试实战教程](https://chao-xi.github.io/jxperfbook/)
>
> [手摸手 Databases 全教程](https://chao-xi.github.io/jxdatabasebook/)
> 
>  [AWS Certified Data Analytics Tutorial](https://chao-xi.github.io/jxawscbdbook/)
> 
> [Istio & Service Mesh 战术教程](https://chao-xi.github.io/jxistiobook/)
> 
> [AWS Certification Solutions Architect Book](https://chao-xi.github.io/jxawscsaabook/)
> 
> [Distributed Message System Book(Kafka)](https://chao-xi.github.io/jxdmsbook/)

## **Salut! C'est Moi**

> The man is not old as long as he is seeking something, A man is not old until regrets take the place of dreams.

Hello, this is me, Jacob. Currently, I'm working as DevOps and Cloud Engineer in SAP, and I'm the certified AWS Solution Architect and Certified Azure Administrator, Kubernetes Specialist, Jenkins CI/CD and ElasticStack enthusiast. 

I was working as Backend Engineer in New York City and achieved my CS master degree in SIT, America. Believe it or not, I'll keep writing, more and more books will come out at such dramatic and unprecedented 2021. 

If you have anything want to talk to me directly, you can reach out for via email xichao2015@outlook.com。


Salute, c'est moi, Jacob. Actuellement, je travaille en tant qu'ingénieur DevOps et Cloud dans SAP, et je suis architecte de solution AWS certifié et administrateur Azure certifié, spécialiste Kubernetes et passionné de CI/CD.

Je travaillais en tant qu'ingénieur backend à New York et j'ai obtenu mon master CS à SIT, en Amérique. Croyez-le ou non, je continuerai à écrire, de plus en plus de livres sortiront cette année.



![Alt Image Text](./images/bg1_2.png "body image")

## **To be continue**

I will start working on Prometheus book later on and in future, I will put more effort do finish "Distributed Message System Book".🙂