---
title: weightedRoute
date: 2020-08-20 20:13:45
tags:
---
## 2.3 权重路由（灰度发布）
![图2-3-1-权重路由概览图](../../images/releaseAndObserve/2-3-1.svg)
背景：随着网站流量的增加，网站开始有了广告投放的需求，广告投放需要在商品页面增加广告位。网站的开发人员新开发了product服务的v2版本，以product v2的deployment的形式提供，并希望对product-v2版本做灰度发布。

先将product v2版本的deployment部署至广州集群：
```yaml
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-v2
  namespace: base
  labels:
    app: product
    version: v2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: product
      version: v2
  template:
    metadata:
      labels:
        app: product
        version: v2
    spec:
      containers:
        - name: product
          image: ccr.ccs.tencentyun.com/zhulei/testproduct2:v1
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
EOF
```

通过DR定义服务版本+通过VS定义权重路由来完成灰度发布的第一步，将部分流量（50%）路由至product v2 subset以验证新版本，剩余部分（50%）的流量仍然路由至product v1版本。以上设定可通过复制以下yaml通过“YAML创建资源”完成。

```yaml
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: product-vs
  namespace: base
spec:
  hosts:
    - "product.base.svc.cluster.local"
  http:
    - match:
        - uri:
            exact: /product
      route:
        - destination:
            host: product.base.svc.cluster.local
            subset: v1
            port:
              number: 7000
          weight: 50
        - destination:
            host: product.base.svc.cluster.local
            subset: v2
            port:
              number: 7000
          weight: 50
---

apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: product
  namespace: base
spec:
  host: product
  subsets:
    - name: v1
      labels:
        version: v1
    - name: v2
      labels:
        version: v2
EOF
```

配置完成后，访问product服务的流量将有50%被路由至v1版本，50%被路由至v2版本，刷新网站商品页面即可验证。
![图2-3-2-权重路由](../../images/releaseAndObserve/2-3-1.svg)

![图2-3-3-50%的请求路由到product v2版本](../../images/releaseAndObserve/2-3-2.png)

product v2版本验证通过后，即可修改关联product的VirtualService中路由规则目的端的权重，设置访问product服务的所有流量（100%）至v2版本，设置完成后可刷新商品列表页面验证。
![图2-3-4-基于virtual Service更改权重](../../images/releaseAndObserve/2-3-3.png)

![图2-3-5-灰度发布完成](../../images/releaseAndObserve/2-3-4.svg)


