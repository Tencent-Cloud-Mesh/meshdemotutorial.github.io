---
title: serviceLocality
date: 2020-08-20 20:20:09
tags:
---
## 3.5.2 地域感知

背景：电商网站业务规模继续增长，上海的网站用户继续增加。业务团队需要在上海集群也部署一套电商网站服务，来自上海地域的请求通过地域感知就近访问上海的服务，广州地域的请求就近访问广州的服务。

首先将网站全套服务也部署至另一地域（上海）集群，将以下yaml文件提交至集群-上海：
```yaml
kubectl apply -f - <<EOF
apiVersion: v1
kind: Namespace
metadata:
  name: base
  labels:
    istio-injection: enabled
spec:
  finalizers:
    - kubernetes
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: base
  labels:
    app: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend
          image: ccr.ccs.tencentyun.com/chloeyhuang/demo:v202007101540
          imagePullPolicy: Always
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: REGION
              value: "shanghai"
          ports:
            - containerPort: 80
---

apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: base
  labels:
    app: frontend
spec:
  ports:
    - port: 80
      name: http
  selector:
    app: frontend
---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: product-v1
  namespace: base
  labels:
    app: product
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: product
      version: v1
  template:
    metadata:
      labels:
        app: product
        version: v1
    spec:
      containers:
        - name: product
          image: ccr.ccs.tencentyun.com/zhulei/testproduct1:v1
          imagePullPolicy: Always
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: REGION
              value: "shanghai"
          ports:
            - containerPort: 7000
---

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
              value: "shanghai"
          ports:
            - containerPort: 7000
---

apiVersion: v1
kind: Service
metadata:
  name: product
  namespace: base
  labels:
    app: product
spec:
  ports:
    - port: 7000
      name: http
  selector:
    app: product
---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: user
  namespace: base
  labels:
    app: user
spec:
  replicas: 1
  selector:
    matchLabels:
      app: user
  template:
    metadata:
      labels:
        app: user
    spec:
      containers:
        - name: user
          image: ccr.ccs.tencentyun.com/zhulei/testuser:v1
          imagePullPolicy: Always
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: REGION
              value: "shanghai"
          ports:
            - containerPort: 7000
---

apiVersion: v1
kind: Service
metadata:
  name: user
  namespace: base
  labels:
    app: user
spec:
  ports:
    - port: 7000
      name: http
  selector:
    app: user
---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: stock
  namespace: base
  labels:
    app: stock
spec:
  replicas: 1
  selector:
    matchLabels:
      app: stock
  template:
    metadata:
      labels:
        app: stock
    spec:
      containers:
        - name: stock
          image: ccr.ccs.tencentyun.com/zhulei/teststock:v1
          imagePullPolicy: Always
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: REGION
              value: "shanghai"
          ports:
            - containerPort: 7000
---

apiVersion: v1
kind: Service
metadata:
  name: stock
  namespace: base
  labels:
    app: stock
spec:
  ports:
    - port: 7000
      name: http
  selector:
    app: stock
---


apiVersion: apps/v1
kind: Deployment
metadata:
  name: cart
  namespace: base
  labels:
    app: cart
spec:
  replicas: 3
  selector:
    matchLabels:
      app: cart
  template:
    metadata:
      labels:
        app: cart
    spec:
      containers:
        - name: cart
          image: ccr.ccs.tencentyun.com/zhulei/testcart:v1
          imagePullPolicy: Always
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: REGION
              value: "shanghai"
          ports:
            - containerPort: 7000
              protocol: TCP
---

apiVersion: v1
kind: Service
metadata:
  name: cart
  namespace: base
  labels:
    app: cart
spec:
  ports:
    - name: http
      port: 7000
      protocol: TCP
  selector:
    app: cart
  type: ClusterIP
---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-v1
  namespace: base
  labels:
    app: order
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: order
      version: v1
  template:
    metadata:
      labels:
        app: order
        version: v1
    spec:
      containers:
        - name: order
          image: ccr.ccs.tencentyun.com/zhulei/testorder1:v1
          imagePullPolicy: Always
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: REGION
              value: "shanghai"
          ports:
            - containerPort: 7000
              protocol: TCP
---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-v2
  namespace: base
  labels:
    app: order
    version: v2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: order
      version: v2
  template:
    metadata:
      labels:
        app: order
        version: v2
    spec:
      containers:
        - name: order
          image: ccr.ccs.tencentyun.com/zhulei/testorder2:v1
          imagePullPolicy: Always
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: REGION
              value: "shanghai"
          ports:
            - containerPort: 7000
              protocol: TCP
---

apiVersion: v1
kind: Service
metadata:
  name: order
  namespace: base
  labels:
    app: order
spec:
  ports:
    - name: http
      port: 7000
      protocol: TCP
  selector:
    app: order
  type: ClusterIP
EOF
```

部署完成后，需要配置所有服务的健康检查功能，通过将以下yaml文件提交至主集群-广州实现

```yaml
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: cart
  namespace: base
spec:
  host: cart
  trafficPolicy:
    loadBalancer:
      consistentHash:
        httpHeaderName: UserID
    outlierDetection:
      consecutiveErrors: 5
      interval: 10000ms
      baseEjectionTime: 30000ms
      maxEjectionPercent: 10
      minHealthPercent: 50
  exportTo:
    - '*'

---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: frontend
  namespace: base
spec:
  host: frontend
  trafficPolicy:
    outlierDetection:
      consecutiveErrors: 5
      interval: 10000ms
      baseEjectionTime: 30000ms
      maxEjectionPercent: 10
      minHealthPercent: 50
  exportTo:
    - '*'

---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: order
  namespace: base
spec:
  host: order
  trafficPolicy:
    outlierDetection:
      consecutiveErrors: 5
      interval: 10000ms
      baseEjectionTime: 30000ms
      maxEjectionPercent: 10
      minHealthPercent: 50
  subsets:
    - name: v1
      labels:
        version: v1
    - name: v2
      labels:
        version: v2
  exportTo:
    - '*'

---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: product
  namespace: base
spec:
  host: product
  trafficPolicy:
    outlierDetection:
      consecutiveErrors: 5
      interval: 10000ms
      baseEjectionTime: 30000ms
      maxEjectionPercent: 10
      minHealthPercent: 50
  subsets:
    - name: v1
      labels:
        version: v1
    - name: v2
      labels:
        version: v2

---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: stock
  namespace: base
spec:
  host: stock
  trafficPolicy:
    outlierDetection:
      consecutiveErrors: 5
      interval: 10000ms
      baseEjectionTime: 30000ms
      maxEjectionPercent: 10
      minHealthPercent: 50
  exportTo:
    - '*'

---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: user
  namespace: base
spec:
  host: user
  trafficPolicy:
    outlierDetection:
      consecutiveErrors: 5
      interval: 10000ms
      baseEjectionTime: 30000ms
      maxEjectionPercent: 10
      minHealthPercent: 50
  exportTo:
    - '*'
EOF
```

健康检查配置完成后，广州的用户通过广州的边缘代理网关访问网站服务，浏览商品页面，添加购物车，下单等操作，广州的边缘代理网关会将流量路由至广州的前端服务，广州的前端服务也会地域感知就近调用广州的user、cart、order、stock服务；上海的用户通过上海的边缘代理网关访问网站业务，请求路由至上海的前端服务，上海的服务也会就近调用上海的服务。通过Demo页面左下角悬浮窗可以查看当前调用服务的地域。

![图3-5-2-1-地域感知](../../../images/netCommunication/3-5-3.png)
