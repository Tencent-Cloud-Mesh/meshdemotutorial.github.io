---
title: circleBreaking
date: 2020-08-20 20:19:09
tags:
---
## 3.4 circuit breaking断路器

背景：随着电商网站业务规模的增大，对网站的访问请求并发量开始增加，网站业务人员计划为服务配置断路器，限制服务最大并发数，避免服务进一步降级，保证服务运行健壮性。

为模拟“高并发”请求场景，首先通过提交以下yaml部署client服务（10 pods），模拟对user服务的高并发请求。
```yaml
kubectl apply -f - <<EOF
apiVersion: v1
kind: Namespace
metadata:
  name: test
  labels:
    istio-injection: enabled
spec:
  finalizers:
    - kubernetes
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: client
  namespace: test
  labels:
    app: client
spec:
  replicas: 10
  selector:
    matchLabels:
      app: client
  template:
    metadata:
      labels:
        app: client
    spec:
      containers:
        - name: client
          image: ccr.ccs.tencentyun.com/zhulei/testclient:v1
          imagePullPolicy: Always
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: REGION
              value: "guangzhou"
          ports:
            - containerPort: 7000
              protocol: TCP
---

apiVersion: v1
kind: Service
metadata:
  name: client
  namespace: test
  labels:
    app: client
spec:
  ports:
    - name: http
      port: 7000
      protocol: TCP
  selector:
    app: client
  type: ClusterIP
EOF
```

此时对于访问user服务没有最大并发数限制，所有请求均可访问成功。通过TKE集群控制台client deployment查看client pod日志，所有的请求均返回了用户名Kevin，证明访问请求成功。

![图3-4-1-高并发请求](../../images/netCommunication/3-4-1.svg)

![图3-4-2-所有请求均可访问成功](../../images/netCommunication/3-4-2.png )

通过配置user服务的Destination Rule限制最大并发数为1：

```yaml
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: user
  namespace: base
spec:
  host: user
  trafficPolicy:
    connectionPool:
      http:
        http1MaxPendingRequests: 1
        http2MaxRequests: 1
        maxRequestsPerConnection: 1
  exportTo:
    - '*'
EOF
```

此时查看client pod日志，部分请求开始出现异常，未返回用户名，请求失败，断路器起到了限制访问服务最大并发数的作用。

![图3-4-3-部分请求访问失败](../../images/netCommunication/3-4-3.png)

断路器测试完成后，在user服务的详情页面删除断路器相关流量策略配置。
![图3-4-4-删除流量策略相关配置](../../images/netCommunication/3-4-4.png)