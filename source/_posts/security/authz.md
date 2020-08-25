---
title: auth
date: 2020-08-20 20:30:38
tags:
---
## 4.2 授权Authorization

背景：生产环境（base namespace）的服务已经稳定运行，电商网站业务团队希望对网格中的服务做权限控制，限制生产环境（base namespace）下的服务不能被测试环境（test namesapce）下的服务访问。

对网格中的服务进行权限控制可通过配置AuthorizationPolicy实现，配置以下AuthorizationPolicy策略限制base namespace下所有服务不能被test namespace下的服务访问。

![图4-2-1-配置授权规则](../../images/safeLink/4-2-1.png)

配置完成后通过TKE集群控制台查看test namespace下client服务的pod日志，发现client服务访问base namespace的user服务失败。授权策略生效。

![图4-2-2-授权规则限制后，访问失败](../../images/safeLink/4-2-2.png)