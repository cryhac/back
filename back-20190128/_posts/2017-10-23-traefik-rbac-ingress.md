---
layout: post
title: Traefik ingress rbac
categories: kubernetes
description: Traefik ingress rbac
keywords: kubernetes
catalog:    true
header-img: "img/pexels/triangular.jpeg"
---


# Traefik Ingress

> 官方文档 https://docs.traefik.io/user-guide/kubernetes/


## 下载 yaml 文件

```
# 基于 kubernetes 1.8 rbac

wget https://raw.githubusercontent.com/containous/traefik/master/examples/k8s/traefik-rbac.yaml

wget https://raw.githubusercontent.com/containous/traefik/master/examples/k8s/traefik-deployment.yaml

```

## 编译 yaml 文件

```
# 这里只需要编辑  deployment 文件

# 首先来 查看一下 node 标签

kubectl get nodes --show-labels

# 我这里增加了 ingress=proxy 标签 到其中两台node 种

```




```
# 编辑 deployment 文件


---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
---
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
  labels:
    k8s-app: traefik-ingress-lb
spec:
  replicas: 2
  selector:
    matchLabels:
      k8s-app: traefik-ingress-lb
  template:
    metadata:
      labels:
        k8s-app: traefik-ingress-lb
        name: traefik-ingress-lb
    spec:
      serviceAccountName: traefik-ingress-controller
      hostNetwork: true
      nodeSelector:
        ingress: proxy
      terminationGracePeriodSeconds: 60
      containers:
      - image: traefik
        name: traefik-ingress-lb
        ports:
        - name: web
          containerPort: 80
          hostPort: 80
        - name: admin
          containerPort: 8888
        args:
        - --web
        - --web.address=:8888   # 修改端口必须配置绑定,否则仍然绑定为 8080 端口
        - --kubernetes
  
```



## 导入 yaml 文件


```

kubectl apply -f .
serviceaccount "traefik-ingress-controller" created
deployment "traefik-ingress-controller" created
service "traefik-ingress-service" created
clusterrole "traefik-ingress-controller" configured
clusterrolebinding "traefik-ingress-controller" configured

```


## 部署 Traefik web ui

```
# 编辑一个 service 服务

vi traefik-web-service.yaml


apiVersion: v1
kind: Service
metadata:
  name: traefik-web-ui
  namespace: kube-system
spec:
  selector:
    k8s-app: traefik-ingress-lb
  ports:
  - port: 80
    targetPort: 8888
    
```





## 编辑一个 Traefik admin-web-ing 文件

```
vi traefik-ing.yaml


apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: traefik-web-ui
  namespace: kube-system
  annotations:
    kubernetes.io/ingress.class: traefik
spec:
  rules:
  - host: traefik.jicki.me
    http:
      paths:
      - backend:
          serviceName: traefik-web-ui
          servicePort: 80

```




## 测试其他 ing


```
# 查看 svc

kubectl get svc nginx
NAME      TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
nginx     ClusterIP   None         <none>        80/TCP    5d




# 编辑 ing

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nginx-ingress
spec:
  rules:
  - host: nginx.jicki.me
    http:
      paths:
      - backend:
          serviceName: nginx
          servicePort: 80




# 查看 ing

kubectl get ing
NAME            HOSTS            ADDRESS   PORTS     AGE
nginx-ingress   nginx.jicki.me             80        6s



# 查看访问


curl nginx.jicki.me
hello word!

```



## 测试访问 web ui


```
curl traefik.jicki.me   

<a href="/dashboard/">Found</a>.

```

![2.png-56.3kB][1]


  [1]: https://jicki.me/img/posts/traefik/2.png
