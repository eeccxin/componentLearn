# 课程介绍

> 极客时间课程：[《Elasticsearch 核心技术与实战》](https://time.geekbang.org/course/intro/100030501)
>
> 视频课程

## 作者简介

阮一鸣，eBay Pronto 平台技术负责人。

Pronto 平台目前管理了 eBay 内部上百个 Elasticsearch 集群，包含了4000 多个数据节点。这些集群目前被广泛使用在 eBay 的生产环境之中。涵盖了网站搜索，商品推荐，日志管理，风险控制，IT 运维，安全监控等多个领域。



毕业于浙江大学电子工程专业，拥有近20年的开发经验，除了在惠普等大型企业的从业经历外，还有丰富的创业经验，包括手机游戏，手机游戏模拟器 WebPod，个性化音乐推荐与分享社区 8box.com 等。

![image-20240930171959455](Elasticsearch核心技术与实战.assets/image-20240930171959455.png)



## 你将获得

1. 掌握 Elasticsearch 核心技能；
2. 熟练进行生产环境中的部署与优化；
3. 灵活运用 ELK 进行搜索与大数据分析；
4. 具备通过 Elastic 官方认证的能力。

## 课程介绍

Elasticsearch 是一款非常强大的开源搜索及分析引擎。

在 DB-Engines Ranking 的数据库评测中，Elasticsearch 在 [Search Engine 分类](https://db-engines.com/en/ranking/search+engine)中长期位列第一。

除了搜索，结合 Kibana、Logstash 和 Beats，Elasticsearch 还被广泛运用在大数据近实时分析，包括日志分析、指标监控、信息安全等多个领域。

在国内，阿里巴巴、腾讯、滴滴、今日头条、饿了么、360 安全、小米、vivo 等诸多知名公司都在使用 Elasticsearch。

这门课将带你全面掌握 Elasticsearch 在生产环境中的核心实战技能。学完后，你可以在工作中快速构建出符合自身业务的分布式搜索和数据分析系统。

- 由浅入深：从基础概念到进阶用法，再到集群管理和大数据分析，学完即可应用到实际生产环境中；
- 实战演练：通过两个 Elasticsearch 实战项目，手把手带你进行实战服务搭建，巩固所学知识点；
- 认证备考：课程内容涵盖 Elastic 认证的全部考点，有助于你顺利通过认证考试。

## 课程目录

![img](Elasticsearch核心技术与实战.assets/366cda6bf58fe0cf0783e9f3f60859bb.jpg)



# 第一章：概述

## 01 | 课程介绍

1、es能做什么？

1）搜索：

订单搜索、商品推荐、日志管理

2）大数据近实时分析：

风险控制、IT运维、安全监控

<img src="Elasticsearch核心技术与实战.assets/image-20240930172951095.png" alt="image-20240930172951095" style="zoom:25%;" />

2、为什么要学es

DBRanking排名（2019年5月）：https://db-engines.com/en/ranking

![image-20240930173129546](Elasticsearch核心技术与实战.assets/image-20240930173129546.png)

最新：2024.09 =》第8

<img src="Elasticsearch核心技术与实战.assets/image-20240930173757699.png" alt="image-20240930173757699" style="zoom: 50%;" />





用的公司多

elastic工程师认证：

![image-20240930173225061](Elasticsearch核心技术与实战.assets/image-20240930173225061.png)



课程使用es 7.1。





## 02 | 内容综述及学习建议

### 为什么要学es

![image-20240930173537901](Elasticsearch核心技术与实战.assets/image-20240930173537901.png)





03 | Elasticsearch简介及其发展历史

04 | Elastic Stack家族成员及其应用场景