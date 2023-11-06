---
title: "Atlas搭建"
date: 2022-09-14T13:17:40+08:00
draft: true
---

crictl没有提供 save 和 load 命令，如何存储和导入镜像呢？

root@nianyu-virtual-machine:~# crictl -h
NAME:
   crictl - client for CRI

USAGE:
   crictl [global options] command [command options] [arguments...]

VERSION:
   v1.24.0

COMMANDS:
   attach              Attach to a running container
   create              Create a new container
   exec                Run a command in a running container
   version             Display runtime version information
   images, image, img  List images
   inspect             Display the status of one or more containers
   inspecti            Return the status of one or more images
   imagefsinfo         Return image filesystem info
   inspectp            Display the status of one or more pods
   logs                Fetch the logs of a container
   port-forward        Forward local port to a pod
   ps                  List containers
   pull                Pull an image from a registry
   run                 Run a new container inside a sandbox
   runp                Run a new pod
   rm                  Remove one or more containers
   rmi                 Remove one or more images
   rmp                 Remove one or more pods
   pods                List pods
   start               Start one or more created containers
   info                Display information of the container runtime
   stop                Stop one or more running containers
   stopp               Stop one or more running pods
   update              Update one or more running containers
   config              Get and set crictl client configuration options
   stats               List container(s) resource usage statistics
   statsp              List pod resource usage statistics
   completion          Output shell completion code
   help, h             Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --config value, -c value            Location of the client config file. If not specified and the default does not exist, the program's directory is searched as well (default: "/etc/crictl.yaml") [$CRI_CONFIG_FILE]
   --debug, -D                         Enable debug mode (default: false)
   --image-endpoint value, -i value    Endpoint of CRI image manager service (default: uses 'runtime-endpoint' setting) [$IMAGE_SERVICE_ENDPOINT]
   --runtime-endpoint value, -r value  Endpoint of CRI container runtime service (default: uses in order the first successful one of [unix:///var/run/dockershim.sock unix:///run/containerd/containerd.sock unix:///run/crio/crio.sock unix:///var/run/cri-dockerd.sock]). Default is now deprecated and the endpoint should be set instead. [$CONTAINER_RUNTIME_ENDPOINT]
   --timeout value, -t value           Timeout of connecting to the server in seconds (e.g. 2s, 20s.). 0 or less is set to default (default: 2s)
   --help, -h                          show help (default: false)
   --version, -v                       print the version (default: false)
解决：使用 ctr 提供命令来操作

root@nianyu-virtual-machine:~# ctr image -h
NAME:
   ctr images - manage images

USAGE:
   ctr images command [command options] [arguments...]

COMMANDS:
   check                    check existing images to ensure all content is available locally
   export                   export images
   import                   import images
   list, ls                 list images known to containerd
   mount                    mount an image to a target path
   unmount                  unmount the image from the target
   pull                     pull an image from a remote
   push                     push an image to a remote
   delete, del, remove, rm  remove one or more images by reference
   tag                      tag an image
   label                    set and clear labels for an image
   convert                  convert an image

OPTIONS:
   --help, -h  show help

作者：87d6dc4b11a7
链接：https://www.jianshu.com/p/0671f094117b
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



ctr image import dashboard.tar

#查询镜像，为什么没有刚才导入的镜像？
crictl images

ctr是containerd自带的工具，有命名空间的概念，若是k8s相关的镜像，都默认在k8s.io这个命名空间，所以导入镜像时需要指定命令空间为k8s.io

#使用ctr命令指定命名空间导入镜像
ctr -n=k8s.io image import dashboard.tar

#查看镜像，可以看到可以查询到了
crictl image
————————————————
版权声明：本文为CSDN博主「平凡似水的人生」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_37837432/article/details/123929774