---
title: "solana结构"
date: 2022-12-28T11:21:05+08:00
draft: true
---

minikube start --driver=docker --container-runtime=containerd --image-mirror-country=cn

https://gist.github.com/islishude/231659cec0305ace090b933ce851994a

docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.26.2  k8s.gcr.io/kube-apiserver:v1.26.2 
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.26.2   k8s.gcr.io/kube-controller-manager:v1.26.2  
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.26.2  k8s.gcr.io/kube-scheduler:v1.26.2 
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.26.2  k8s.gcr.io/kube-proxy:v1.26.2 
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.9   k8s.gcr.io/pause:3.9  
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.5.6-0  k8s.gcr.io/etcd:3.5.6-0 
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:v1.9.3  k8s.gcr.io/coredns:v1.9.3 