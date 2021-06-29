---
title: Kubernetes-On-MacOS-minikube
tags:
  - k8s
toc: true
date: 2021-06-29 15:18:15
categories:
---

## 安装软件

```zsh
brew install docker
brew install virtualbox
brew install kubectl
brew install minikube
```

## minikube 命令

### start

```zsh
## minikube 支持三种HyperKit、VirtualBox、Parallels Desktop、VMware Fusion
## 设置默认 driver
minikube config set driver virtualbox

## 启动集群
## 简单版本
minikube start
## 详细参数版本
minikube start --v=0 --kubernetes-version v1.20.7 --cpus 4 --memory=8192 --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers --driver=virtualbox

## 参数说明
--v 日志等级
  0 INFO level logs
  1 WARNING level logs
  2 ERROR level logs
  3 libmachine logging
  7 libmachine --debug level logging
--kubernetes-version 指定版本
--cpus 4 --memory=8192 指定资源
--image-repository 指定镜像代理
--driver=virtualbox 指定 driver, 如果设置过默认 driver 这个可以忽略
```

### dashboard

```zsh
## 在新的终端运行命令
## 打开仪表盘，默认会打开一个浏览器页面
minikube dashboard

## 打开仪表盘，但是不打开浏览器
minikube dashboard --url
```


### addons
```zsh
## addons 相关命令
## 启用 ingress
minikube addons enable ingress
## 禁用 ingress
minikube addons disable ingress

minikube addons list
```

### service

```zsh
## 运行 kubectl 创建好的服务
minikube service hello-node
```

### stop delete

```zsh
## 关闭 minikube virtual machine
minikube stop
## 删除 minikube virtual machine
minikube delete
```

## kubectl 命令

### deployment

```zsh
## 创建 deployment
kubectl create deployment hello-node --image=k8s.gcr.io/echoserver:1.4
## 查看 deployment
kubectl get deployments
```

### event
```zsh
## 查看 cluster events
kubectl get events
```

### configuration
```zsh
## 查看 cubectl configuration
kubectl config view
```

### pods
```zsh
## 查看 pod
#### 查看 namespace 为 default 的 pod
kubectl get pods
#### 查看所有 namespace 的 pod
kubectl get pods --all-namespaces -o wide
#### 查看指定 namespace 为 kube-system 的 pod
kubectl get pods -n kube-system -o wide

## 查询详细
kubectl describe pod PodName
```

### services
```zsh
## 查看你创建的 pod,service
kubectl get pod,svc -n kube-system

## 查看 service
kubectl get services --all-namespaces
## 创建 service
kubectl expose deployment hello-node --type=LoadBalancer --port=8080
## 查看 service
kubectl get services --all-namespaces
## 注: 对比第一次查看 service, 此时多了一个刚刚 expose 的 hello-node 服务
## 原因: 默认 pod都是只在集群内部可用，只有expose之后，外部网络才可以访问到

## minikube 运行服务
minikube service hello-node
```

### cleanup
```zsh
## 删除 service deploy, 即 清理资源
kubectl delete service hello-node
kubectl delete deployment hello-node
```


## 参考资料
[在MacOS上安装kubernetes](https://www.jianshu.com/p/93e4d1508a52)  
[在MAC上安装K8S (kubernets) for Docker Desktop](https://zhuanlan.zhihu.com/p/65559363)  
[手摸手教你从开发到部署(CI/CD)GO微服务系列](https://learnku.com/docs/go-micro-build/1.0/kubernetes-mac-os-based-installation-tutorial/8878)  
[minikube docs](https://minikube.sigs.k8s.io/docs/)  
[k8s tutorials Hello Minikube](https://kubernetes.io/docs/tutorials/hello-minikube/)