---
title: "K8s内部网络调试"
date: 2023-03-10T10:23:59+08:00
draft: true
---

```
kubectl run tmp-shell --rm -i --tty --image busybox
 kubectl run tmp-shell --restart=Never --rm -i --tty --image busybox -n lili -- /bin/sh 

  kubectl run tmp-shell --restart=Never --rm -i --tty --image ubuntu -n lili -- /bin/sh 
```

我首先通过nslookup 检查是否可以查询


随后验证pod是否过载


kubectl top pod -l k8s-app=kube-dns -n kube-system
或者
kubectl get nodes --selector=kubernetes.io/role!=master -o jsonpath={.items[*].status.addresses[?\(@.type==\"InternalIP\"\)].address}

```
安装metrics-server的步骤如下：

下载metrics-server的yaml文件：wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml -O metrics-server-components.yaml
更改yaml文件中的镜像地址为阿里云镜像地址：sed -i 's/k8s.gcr.io/registry.cn-hangzhou.aliyuncs.com\/google_containers/g' metrics-server-components.yaml
在k8s集群中部署metrics-server：kubectl apply -f metrics-server-components.yaml
检查metrics-server是否正常运行：kubectl get pods -n kube-system | grep metrics
```