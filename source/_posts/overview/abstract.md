---
title: overview/abstract
date: 2020-08-20 20:09:17
tags:
---

## 1.2 Demo概览

Demo是一个电商网站，由6个服务组成：
- frontend：网站前端，调用user、product、cart、order服务
- product：商品服务，提供商品信息
两个版本，版本一没有顶部广告banner；版本二有顶部广告banner
- user：用户登陆服务，提供登陆功能
- cart：购物车服务，提供添加、查看购物车功能，调用库存服务提供库存告警功能，需要登陆才可以下单
- order：订单结算服务，登陆后点击checkout后可发起结算，结算时需要调用stock库存服务查询库存情况，库存不足会下单失败
两个版本，版本一无积分抵扣运费的功能，版本二有积分抵扣运费的功能
- stock：库存服务，为order购物车服务的库存告警功能和order订单结算服务的库存查询提供库存信息

![图1-2-1-demo结构](../../images/overview/1-2-1.svg)

![图1-2-2-demo首页](../../images/overview/1-2-2.png)
