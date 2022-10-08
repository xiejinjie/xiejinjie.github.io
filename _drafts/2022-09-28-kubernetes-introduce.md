---
layout: post
title: "Kubernetes简介"
date: 2022-09-27 09:22:00 +0800
categories: tech
---

## 系统部署演变历史
![](https://raw.githubusercontent.com/xiejinjie/xiejinjie.github.io/gh-pages/assets/img/20220928103541.png)

## kubernetes核心功能
Kubernetes提供了一个可弹性运行分布式系统的框架。Kubernetes能够弹性扩容、故障转移、提供部署模式等。 

## 基本概念

## kubernetes API
[API参考文档](https://kubernetes.io/zh-cn/docs/reference/kubernetes-api/)

## 使用minikuber创建第一个集群
Minikube是一种轻量级的 Kubernetes 实现，可在本地计算机上创建 VM 并部署仅包含一个节点的简单集群。
minikube version 确认minikuber已正确安装配置
minikube start 启动kubernetes本地集群minikube

- kuberctl
kubectl 是一个kubernetes的命令行终端，通过它可以与集群交互。
kubectl version 确认kubectl已正确安装配置
kubectl cluster-info 查询集群信息
kubectl get nodes 查询节点信息
- deployment
kubectl create deployment 创建部署对象
```
kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1
```
kubectl get deployments 查看部署对象
kubectl proxy 启动一个代理，这个代理会转发会话到集群内部
- pod
kubectl get pods 获取pods信息
kubectl describe pods 查看pods详细信息

应用输出到标准输出STDOUT的任何信息都会成为pod中容器的日志
kubectl logs $POD_NAME 查看pod日志

pod启动之后可以向pod中的容器提交命令，只有一个容器时可以省略其名称
kubectl exec $POD_NAME -- $cmd 执行命令
kubectl exec -ti $POD_NAME -- bash 启用命令行


## pod
Pod 是 Kubernetes 抽象出来的，表示一组一个或多个应用程序容器（如 Docker），以及这些容器的一些共享资源。
Pod 为特定于应用程序的“逻辑主机”建模，并且可以包含相对紧耦合的不同应用容器。例如，Pod 可能既包含带有 Node.js 应用的容器，也包含另一个不同的容器，用于提供 Node.js 网络服务器要发布的数据。Pod 中的容器共享 IP 地址和端口，始终位于同一位置并且共同调度，并在同一工作节点上的共享上下文中运行。
Pod是 Kubernetes 平台上的原子单元。 当我们在 Kubernetes 上创建 Deployment 时，该 Deployment 会在其中创建包含容器的 Pod （而不是直接创建容器）。每个 Pod 都与调度它的工作节点绑定，并保持在那里直到终止（根据重启策略）或删除。 如果工作节点发生故障，则会在集群中的其他可用工作节点上调度相同的 Pod。

## 工作节点
一个 pod 总是运行在 工作节点。工作节点是 Kubernetes 中的参与计算的机器，可以是虚拟机或物理计算机，具体取决于集群。每个工作节点由主节点管理。工作节点可以有多个 pod ，Kubernetes 主节点会自动处理在集群中的工作节点上调度 pod 。 主节点的自动调度考量了每个工作节点上的可用资源。
每个 Kubernetes 工作节点至少运行:
- Kubelet，负责 Kubernetes 主节点和工作节点之间通信的过程; 它管理 Pod 和机器上运行的容器。
- 容器运行时（如 Docker）负责从仓库中提取容器镜像，解压缩容器以及运行应用程序。

