---
title: "Atlas搭建"
date: 2022-09-14T13:17:40+08:00
draft: true
---

<!-- ```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add - && \
  echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list && \
  sudo apt-get update -q && \
  sudo apt-get install -qy kubelet=<version> kubectl=<version> kubeadm=<version>
```

where available <version> is:
```bash
curl -s https://packages.cloud.google.com/apt/dists/kubernetes-xenial/main/binary-amd64/Packages | grep Version | awk '{print $2}'
``` -->

## 配置科学网络

## 安装rke服务
参照: https://docs.rke2.io/zh/install/quickstart
```bash
curl -sfL https://get.rke2.io | sh -

curl -sfL https://rancher-mirror.rancher.cn/rke2/install.sh | INSTALL_RKE2_MIRROR=cn sh -
```


## 卸载旧集群信息

两个清理脚本 rke2-killall.sh 和 rke2-uninstall.sh 将安装到以下路径：
如果是常规文件系统，则是 /usr/local/bin
如果是只读和 brtfs 文件系统，则是 /opt/rke2/bin
如果设置了 INSTALL_RKE2_TAR_PREFIX，则是 INSTALL_RKE2_TAR_PREFIX/bin

rm -rf /etc/ceph \
       /etc/cni \
       /etc/rancher \
       /etc/kubernetes \
       /opt/cni \
       /opt/rke \
       /run/secrets/kubernetes.io \
       /run/calico \
       /run/flannel \
       /var/lib/calico \
       /var/lib/etcd \
       /var/lib/cni \
       /var/lib/kubelet \
       /var/lib/rancher\
       /var/log/containers \
       /var/log/kube-audit \
       /var/log/pods \
       /var/run/calico

# 启动rke2服务
2. 启用 rke2-server 服务
systemctl enable rke2-server.service

3. 启动服务
systemctl start rke2-server.service

4. 如有需要，可以查看日志
journalctl -u rke2-server -f

## worker
Linux Agent（Worker）节点安装
本节中的步骤需要 root 级别访问权限或 sudo 才能执行。

1. 运行安装程序
curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE="agent" sh -

备注
中国用户，可以使用以下方法加速安装：

curl -sfL https://rancher-mirror.rancher.cn/rke2/install.sh | INSTALL_RKE2_MIRROR=cn INSTALL_RKE2_TYPE="agent" sh -

这会将 rke2-agent 服务和 rke2 二进制文件安装到你的主机上。除非以 root 用户或通过 sudo 运行，否则它将失败。

2. 启用 rke2-agent 服务
systemctl enable rke2-agent.service

3. 配置 rke2-agent 服务
mkdir -p /etc/rancher/rke2/
vim /etc/rancher/rke2/config.yaml

config.yaml 的内容：

server: https://<server>:9345
token: <token from server node>

备注
rke2 server 进程在端口 9345 上侦听新节点注册。Kubernetes API 仍然照常在端口 6443 上提供服务。

4. 启动服务
systemctl start rke2-agent.service

如有需要，可以查看日志

journalctl -u rke2-agent -f

注意：每台主机必须具有唯一的主机名。如果你的主机没有唯一的主机名，请在 config.yaml 文件中设置 node-name 参数，并为每个节点提供一个有效且唯一的主机名。

有关 config.yaml 文件的更多信息，请参阅安装选项文档。



# 启动rancher

1. 添加 Helm Chart 仓库#
使用helm repo add命令添加含有 Rancher Chart 的 Helm Chart 仓库。

请将命令中的<CHART_REPO>，替换为latest，stable或alpha。更多信息，请查看选择 Rancher 版本来选择最适合你的仓库。

latest: 建议在尝试新功能时使用。
stable: 建议在生产环境中使用。（推荐）
alpha: 未来版本的实验性预览。
注意：不支持从 Alpha 到 Alpha 之间的升级。

helm repo add rancher-<CHART_REPO> https://releases.rancher.com/server-charts/<CHART_REPO>
提示
国内用户，可以使用放在国内的 Rancher Chart 加速安装：

helm repo add rancher-<CHART_REPO> https://rancher-mirror.rancher.cn/server-charts/<CHART_REPO>
2. 为 Rancher 创建 Namespace#
我们需要定义一个 Kubernetes Namespace，在 Namespace 中安装由 Chart 创建的资源。这个命名空间的名称为 cattle-system：

kubectl create namespace cattle-system
3. 选择你的 SSL 选项#
Rancher Server 默认需要 SSL/TLS 配置来保证访问的安全性。

提示
如果你想在外部终止 SSL/TLS，请参考：使用外部 TLS 负载均衡器。

你可以从以下三种证书来源中选择一种，证书将用来在 Rancher Server 中终止 TLS：

Rancher 生成的 TLS 证书： 在这种情况下，你需要在集群中安装 cert-manager。 Rancher 利用 cert-manager 签发并维护证书。Rancher 将生成自己的 CA 证书，并使用该 CA 签署证书。然后 cert-manager 负责管理该证书。
Let's Encrypt： Let's Encrypt 选项也需要使用 cert-manager。但是，在这种情况下，cert-manager 与 Let's Encrypt 的特殊颁发者相结合，该颁发者执行获取 Let's Encrypt 颁发的证书所需的所有操作（包括请求和验证）。此配置使用 HTTP 验证（HTTP-01），因此负载均衡器必须具有可以从公网访问的公共 DNS 记录。
使用你已有的证书： 此选项使你可以使用自己的权威 CA 颁发的证书或自签名 CA 证书。 Rancher 将使用该证书来保护 WebSocket 和 HTTPS 流量。在这种情况下，你必须上传名称分别为tls.crt和tls.key的 PEM 格式的证书以及相关的密钥。如果使用私有 CA，则还必须上传该 CA 证书。这是由于你的节点可能不信任此私有 CA。 Rancher 将获取该 CA 证书，并从中生成一个校验和，各种 Rancher 组件将使用该校验和来验证其与 Rancher 的连接。
设置	Chart 选项	描述	是否需要 cert-manager
Rancher 生成的证书（默认）	ingress.tls.source=rancher	使用 Rancher 生成的 CA 签发的自签名证书此项为默认选项	是
Let’s Encrypt	ingress.tls.source=letsEncrypt	使用Let's Encrypt颁发的证书	是
你已有的证书	ingress.tls.source=secret	使用你的自己的证书（Kubernetes 密文）	否
重要
Rancher 中国技术支持团队建议你使用“你已有的证书” ingress.tls.source=secret 这种方式，从而减少对 cert-manager 的运维成本。

4. 安装 cert-manager#
提示
如果你使用自己的证书文件 ingress.tls.source=secret或者使用外部 TLS 负载均衡器可以跳过此步骤。

仅在使用 Rancher 生成的证书 ingress.tls.source=rancher 或 Let's Encrypt 颁发的证书 ingress.tls.source=letsEncrypt时才需要 cert-manager。

这些说明来自官方的 cert-manager 文档。


# 如果你手动安装了CRD，而不是在Helm安装命令中添加了`--set installCRDs=true`选项，你应该在升级Helm chart之前升级CRD资源。
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.5.1/cert-manager.crds.yaml

# 添加 Jetstack Helm 仓库

helm repo add jetstack https://charts.jetstack.io

# 更新本地 Helm chart 仓库缓存

helm repo update

# 安装 cert-manager Helm chart

helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.5.1

安装完 cert-manager 后，你可以通过检查 cert-manager 命名空间中正在运行的 Pod 来验证它是否已正确部署：

kubectl get pods --namespace cert-manager

NAME                                       READY   STATUS    RESTARTS   AGE
cert-manager-5c6866597-zw7kh               1/1     Running   0          2m
cert-manager-cainjector-577f6d9fd7-tr77l   1/1     Running   0          2m
cert-manager-webhook-787858fcdb-nlzsq      1/1     Running   0          2m
5. 根据你选择的 SSL 选项，通过 Helm 安装 Rancher#
Rancher 生成的证书
Let's Encrypt 证书
已有的证书
默认为 Rancher 生成自签名 CA，用于 cert-manager 颁发访问 Rancher Server 接口的证书。

因为 rancher 是 ingress.tls.source 的默认选项，所以在运行 helm install 命令时我们没有指定 ingress.tls.source。

将 hostname 设置为解析到你的负载均衡器的 DNS 记录。Rancher HA 安装成功后，你需要通过这个域名来访问 Rancher Server。
将 replicas 设置为 Rancher 部署所使用的副本数量。默认为 3；如果集群中的节点少于 3 个，你应该相应地减少副本数量。
要安装一个特定的 Rancher 版本，使用 --version 标志，例如：--version 2.3.6。
如果你安装的是 alpha 版本，Helm 要求在命令中加入 --devel 选项。
helm install rancher rancher-<CHART_REPO>/rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org \
  --set replicas=3
等待 Rancher 运行：

kubectl -n cattle-system rollout status deploy/rancher
Waiting for deployment "rancher" rollout to finish: 0 of 3 updated replicas are available...
deployment "rancher" successfully rolled out
Rancher Chart 有许多自定义选项来自定义你的安装环境。以下是一些常见的高级方案：

HTTP 代理
私有镜像仓库
外部负载均衡器上的 TLS 终止
有关选项的完整列表，请参见Chart 选项。

1. 验证 Rancher Server 是否已成功部署#
添加 secret 后，检查 Rancher Server 是否运行成功：

kubectl -n cattle-system rollout status deploy/rancher
Waiting for deployment "rancher" rollout to finish: 0 of 3 updated replicas are available...
deployment "rancher" successfully rolled out
如果看到以下错误： error: deployment "rancher" exceeded its progress deadline, 你可以通过运行以下命令来检查 deployment 的状态：

kubectl -n cattle-system get deploy rancher
NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
rancher   3         3         3            3           3m
DESIRED和AVAILABLE应该显示相同的个数。


https://docs.rancher.cn/docs/rancher2.5/installation/resources/tls-secrets/_index/

如果您使用的是私有 CA，Rancher 需要您提供 CA 证书的副本，用来校验 Rancher Agent 与 Server 的连接。

拷贝 CA 证书到名为 cacerts.pem 的文件，使用 kubectl 命令在 cattle-system 命名空间中创建名为 tls-ca 的密文。

kubectl -n cattle-system create secret generic tls-ca \
  --from-file=cacerts.pem=./cacerts.pem

```bash
helm install rancher rancher-stable/rancher \
  --namespace cattle-system \
  --set hostname=test.rancher \
  --set ingress.tls.source=secret \
  --set privateCA=true
```

## 忘记密码
```bash
kubectl exec -n cattle-system deploy/rancher -- reset-password
```

```
kubectl -n cattle-system exec $(kubectl  -n cattle-system get pods -l app=rancher | grep '1/1' | head -1 | awk '{ print $1 }') -- reset-password
```

## 网络插件
https://www.bladewan.com/2022/01/01/rke2/
使用RKE2部署Kubernetes使用其他网络插件
默认情况下rke2部署使用的是canal做为网络插件，还支持calico和cilium网络插件，若想使用其他网络插件只需要进行配置即可。

如cilium
cilium依赖内核bfp特性，在启用前需要先进行挂载。
检查是否有进行挂载

1
mount | grep /sys/fs/bpf
进行挂载

1
2
3
4
sudo mount bpffs -t bpf /sys/fs/bpf
sudo bash -c 'cat <<EOF >> /etc/fstab
none /sys/fs/bpf bpf rw,relatime 0 0
EOF'
在次检查

1
2
3
mount | grep /sys/fs/bpf
bpffs on /sys/fs/bpf type bpf (rw,relatime)
bpffs on /sys/fs/bpf type bpf (rw,relatime)
在start rke2-server和agent服务前先配置config.yaml

1
2
mkdir -p /etc/rancher/rke2/
vim /etc/rancher/rke2/config.yaml
添加以下参数

1
cni: cilium
启动rke2-server

1
systemctl start rke2-server.service
查看是否部署成功


组件参数配置
在/etc/rancher/rke2/config.yaml 文件中，按照对应组件，添加对应的参数，如apiserver对应为kube-apiserver-arg，组件对应参数为etcd-arg。kube-controller-manager-arg、kube-scheduler-arg、kubelet-arg、kube-proxy-arg。

1
2
3
4
5
6
7
etcd-arg:
  - "quota-backend-bytes=858993459"
  - "max-request-bytes=33554432"
kube-apiserver-arg:
  - "watch-cache=true"
kubelet-arg:
  - "system-reserved=cpu=1,memory=2048Mi"
配置完成后启动rke2-server。agent节点要同步时配置，否则kubelet和kube-proxy参数将不生效

检查参数是否生效
如：

1
ps aux|grep system-reserved
集群备份和还原
rke2备份文件保存在每个拥有etcd角色的节点的/var/lib/rancher/rke2/server/db/snapshots目录内，拥有多副本保存。
默认每隔12小时备份一次，保留5份。

注：目前版本只能通过定时备份，没有立刻备份的选型。
将

指定备份文件恢复

关闭rke2-server进程

1
systemctl stop rke2-server
指定文件恢复

1
2
3
rke2 server \
  --cluster-reset \
  --cluster-reset-restore-path=<PATH-TO-SNAPSHOT>
若是HA集群，还原成功后在其他server节点将执行rm -rf /var/lib/rancher/rke2/server/db然后重新启动server，加入集群。

rke2跟rke1一样也支持将备份文件在一个新集群进行还原。

etcd操作
etcdctl check perf

1
for etcdpod in $(kubectl -n kube-system get pod -l component=etcd --no-headers -o custom-columns=NAME:.metadata.name); do kubectl -n kube-system exec $etcdpod -- sh -c "ETCDCTL_ENDPOINTS='https://127.0.0.1:2379' ETCDCTL_CACERT='/var/lib/rancher/rke2/server/tls/etcd/server-ca.crt' ETCDCTL_CERT='/var/lib/rancher/rke2/server/tls/etcd/server-client.crt' ETCDCTL_KEY='/var/lib/rancher/rke2/server/tls/etcd/server-client.key' ETCDCTL_API=3 etcdctl check perf"; done
etcdctl endpoint status

1
for etcdpod in $(kubectl -n kube-system get pod -l component=etcd --no-headers -o custom-columns=NAME:.metadata.name); do kubectl -n kube-system exec $etcdpod -- sh -c "ETCDCTL_ENDPOINTS='https://127.0.0.1:2379' ETCDCTL_CACERT='/var/lib/rancher/rke2/server/tls/etcd/server-ca.crt' ETCDCTL_CERT='/var/lib/rancher/rke2/server/tls/etcd/server-client.crt' ETCDCTL_KEY='/var/lib/rancher/rke2/server/tls/etcd/server-client.key' ETCDCTL_API=3 etcdctl endpoint status"; done
etcdctl endpoint health

1
for etcdpod in $(kubectl -n kube-system get pod -l component=etcd --no-headers -o custom-columns=NAME:.metadata.name); do kubectl -n kube-system exec $etcdpod -- sh -c "ETCDCTL_ENDPOINTS='https://127.0.0.1:2379' ETCDCTL_CACERT='/var/lib/rancher/rke2/server/tls/etcd/server-ca.crt' ETCDCTL_CERT='/var/lib/rancher/rke2/server/tls/etcd/server-client.crt' ETCDCTL_KEY='/var/lib/rancher/rke2/server/tls/etcd/server-client.key' ETCDCTL_API=3 etcdctl endpoint health"; done
etcdctl compact

1
2
rev=$(kubectl -n kube-system exec $(kubectl -n kube-system get pod -l component=etcd --no-headers -o custom-columns=NAME:.metadata.name | head -1) -- sh -c "ETCDCTL_ENDPOINTS='https://127.0.0.1:2379' ETCDCTL_CACERT='/var/lib/rancher/rke2/server/tls/etcd/server-ca.crt' ETCDCTL_CERT='/var/lib/rancher/rke2/server/tls/etcd/server-client.crt' ETCDCTL_KEY='/var/lib/rancher/rke2/server/tls/etcd/server-client.key' ETCDCTL_API=3 etcdctl endpoint status --write-out fields | grep Revision | cut -d: -f2")
kubectl -n kube-system exec $(kubectl -n kube-system get pod -l component=etcd --no-headers -o custom-columns=NAME:.metadata.name | head -1) -- sh -c "ETCDCTL_ENDPOINTS='https://127.0.0.1:2379' ETCDCTL_CACERT='/var/lib/rancher/rke2/server/tls/etcd/server-ca.crt' ETCDCTL_CERT='/var/lib/rancher/rke2/server/tls/etcd/server-client.crt' ETCDCTL_KEY='/var/lib/rancher/rke2/server/tls/etcd/server-client.key' ETCDCTL_API=3 etcdctl compact \"$(echo $rev)\""
etcdctl defrag

1
kubectl -n kube-system exec $(kubectl -n kube-system get pod -l component=etcd --no-headers -o custom-columns=NAME:.metadata.name | head -1) -- sh -c "ETCDCTL_ENDPOINTS='https://127.0.0.1:2379' ETCDCTL_CACERT='/var/lib/rancher/rke2/server/tls/etcd/server-ca.crt' ETCDCTL_CERT='/var/lib/rancher/rke2/server/tls/etcd/server-client.crt' ETCDCTL_KEY='/var/lib/rancher/rke2/server/tls/etcd/server-client.key' ETCDCTL_API=3 etcdctl defrag --cluster"
对应的，直接操作etcdctl
参考：https://gist.github.com/superseb/3b78f47989e0dbc1295486c186e944bf




/usr/share/rancher/ui-dashboard/dashboard rancher 容器

zh-hans-yaml.964e0d7e.js 
zh-hans-yaml.964e0d7e.js.map 

zypper install wget
cd /usr/share/rancher/ui-dashboard/dashboard
rm zh-hans-yaml.964e0d7e.js
rm zh-hans-yaml.964e0d7e.js.amp
wget http://192.168.1.31:80/zh-hans-yaml.964e0d7e.js
wget http://192.168.1.31:80/zh-hans-yaml.964e0d7e.js.map

mysql外网访问: https://blog.csdn.net/liliuqing/article/details/88723409

echo 'Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch' | sudo tee /etc/pacman.d/mirrorlist