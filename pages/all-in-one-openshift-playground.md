# All-in-One K8S Playground新增OpenShift支持

---

> All-in-One K8S Playground v1.2发布，喜迎OpenShift入驻！

<!-- Place this tag in your head or just before your close body tag. -->
<script async defer src="https://buttons.github.io/buttons.js"></script>

> 开源项目[lab-k8s-playground](https://github.com/morningspace/lab-k8s-playground) v1.2发布啦！除了支持标准Kubernetes集群，OpenShift现在也可以跑在All-in-One K8S Playground里啦！如果您喜欢这个项目，欢迎点击下面的按钮，关注，加星，以及贡献代码^_^

<a class="github-button" href="https://github.com/morningspace/lab-k8s-playground/subscription" data-icon="octicon-eye" data-size="large" aria-label="Watch morningspace/lab-k8s-playground on GitHub">Watch</a> <a class="github-button" href="https://github.com/morningspace/lab-k8s-playground" data-icon="octicon-star" data-size="large" aria-label="Star morningspace/lab-k8s-playground on GitHub">Star</a> <a class="github-button" href="https://github.com/morningspace/lab-k8s-playground/fork" data-icon="octicon-repo-forked" data-size="large" aria-label="Fork morningspace/lab-k8s-playground on GitHub">Fork</a>

## OpenShift

![](https://morningspace.github.io/assets/images/lab/k8s/openshift.png)

[OpenShift](https://www.openshift.com/)是Red Hat开发的构筑于Kubernetes之上的企业级容器应用平台，为企业应用与服务的开发，管理，以及部署提供了全面的支持，是当前炙手可热的一款Kubernetes发行版本。其社区版称为[OKD](https://www.okd.io/)（OpenShift Kubernetes Distribution的缩写，原名Origin）。

本文将告诉你，利用[All-in-One Kubernetes Playground](https://github.com/morningspace/lab-k8s-playground/)，我们不仅可以在单机环境下部署多节点的标准Kubernetes集群，还可以部署一个支持OKD v3.11版本的单节点集群。不仅如此，像[Istio](https://istio.io)和[API Connect](https://developer.ibm.com/apiconnect)这些应用，也已经被成功迁移到了OpenShift集群上。只需要简单几条命令，就可以完成部署，整个过程是完全自动化进行的，并且可以无限次重复！

## 准备环境

在[All-in-One K8S Playground 中文使用指南](http://localhost:4000/tech/all-in-one-k8s-playground/)一文里，我们已经学会了如何在单机环境下快速自动部署起一个支持多节点的标准Kubernetes集群。如果要想让Playground支持OpenShift，我们只需要在此基础上，再指定一个额外的环境变量`K8S_PROVIDER`就可以了。目前，该环境变量支持的合法值包括：
* `dind`，代表基于DIND技术（即Docker-in-Docker）实现的，可在单机环境里运行的，多节点标准Kubernetes集群；
* `oc`，代表基于OpenShift的`oc cluster up`命令实现的，可在单机环境里运行的，单节点OpenShift集群；

为了让环境变量的设置持久生效，我们可以编辑`~/.bashrc`文件：
```shell
# The IP of your host, default is 127.0.0.1
export HOST_IP=<your_host_ip>
# The Kubernetes provider, default is dind
export K8S_PROVIDER=oc
# The Kubernetes version, default is v1.14
export K8S_VERSION=
# The number of worker nodes, default is 2
export NUM_NODES=
```

> 如果我们选择部署OpenShift，那么`K8S_VERSION`和`NUM_NODES`这两个环境变量会被忽略。

为了在当前登录终端里让上面的改动生效，我们需要执行下面的命令重新加载`.bashrc`：
```shell
$ . ~/.bashrc
```

执行下面的命令，可以验证当前环境变量的设置是否正确：
```shell
$ launch env
Targets to be launched: [env]
####################################
# Launch target env...
####################################
LAB_HOME=/root/lab-playgrounds/lab-k8s-playground
HOST_IP=192.168.0.10
K8S_PROVIDER=oc
K8S_VERSION=
NUM_NODES=
Total elapsed time: 0 seconds
```

## 启动OpenShift

启动OpenShift集群所使用的命令，和启动标准Kubernetes集群是一模一样的：
```shell
$ launch kubernetes
```

等命令执行完毕以后，Playground会为我们在当前主机上启动起一个单节点的OpenShift集群。并且，只要机器性能足够好，我们甚至可以在一台机器上同时跑两个集群，分别代表标准的Kubernetes集群和OpenShift集群：
```shell
$ K8S_PROVIDER=dind launch kubernetes
$ K8S_PROVIDER=oc launch kubernetes
```

执行如下命令，我们可以同时看到跑在标准Kubernetes集群上的Dashboard和跑在OpenShift集群上的Web Console的访问地址：
```shell
$ launch endpoints
Targets to be launched: [endpoints]
####################################
# Launch target endpoints...
####################################
» common endpoints...
✔      Web Terminal: https://192.168.0.10:4200 
✔         Dashboard: http://192.168.0.10:32774/api/v1/namespaces/kube-system/services/http:kubernetes-dashboard:/proxy 
✔ OpenShift Console: https://192.168.0.10:8443/console 
Total elapsed time: 0 seconds
```

### 动图演示

下面这个动图所演示的，就是如何在一台8核32G内存的机器上，先启动一个多节点的标准Kubernetes集群，然后再启动一个单节点的OpenShift集群。利用[kubectx](https://github.com/ahmetb/kubectx)，我们可以实现在这两个集群上下文之间的自如切换（点击图片可放大观看哦）：

[![](https://morningspace.github.io/lab-k8s-playground/docs/demo-oc.gif)](https://morningspace.github.io/lab-k8s-playground/docs/demo-oc.gif)

## 使用OpenShift

根据`launch endpoints`返回的结果，我们可以登录到OpenShift所提供的Web Console里，对应用和集群进行管理。

> 默认的管理员账户/密码为：admin/admin；开发人员账户/密码为：developer/developer。

如果希望通过命令行来管理应用和集群，则可以利用OpenShift提供的`oc`命令进行登录。以管理员身份登录：
```shell
$ oc login -u system:admin
```

或者以开发人员身份登录：
```shell
$ oc login -u developer
```

## 在OpenShift上运行Istio和APIC

在All-in-One K8S Playground里基于OpenShift运行Istio或APIC只需要一条命令就能实现。而且，这条命令和基于标准Kubernetes集群运行Istio或APIC是一模一样的！关于如何在标准Kubernetes集群里运行Istio，请参见[All-in-One K8S Playground 中文使用指南](http://localhost:4000/tech/all-in-one-k8s-playground/)一文；API Connect，则参见[把API Connect关进All-in-One K8S Playground里](http://localhost:4000/tech/all-in-one-apic-playground/)一文。

等Kubernetes集群启动起来以后，执行如下命令启动Istio：
```shell
$ launch istio istio-bookinfo
```

执行如下命令启动APIC：
```shell
$ launch apic
```

在标准的Kubernetes集群里部署Istio和API Connect时，最后需要通过端口转发（port forwarding）把应用的访问地址暴露出来，从而可以在集群外面对应用进行访问。对于OpenShift而言，因为是直接在宿主机上部署的集群和应用，所以就不需要这一个步骤了。

另外，对于应用的访问地址，由于使用了[`nip.io`](https://nip.io/)提供的DNS服务，所以也不需要把主机IP和主机名的映射定义添加到本地的`/etc/hosts`文件里。

总之一句话，等集群启动起来以后，无需额外的步骤，我们就可以直接访问集群中的应用了。至于这些应用的访问地址具体长啥样，可以通过运行`launch endpoints`命令查看到。
```shell
$ launch endpoints
Targets to be launched: [endpoints]
####################################
# Launch target endpoints...
####################################
» apic endpoints...
✔   Gateway Management Endpoint: https://gwd.192.168.0.10.nip.io 
?     Gateway API Endpoint Base: https://gw.192.168.0.10.nip.io 
✔    Portal Management Endpoint: https://padmin.192.168.0.10.nip.io 
✔            Portal Website URL: https://portal.192.168.0.10.nip.io 
✔ Analytics Management Endpoint: https://ac.192.168.0.10.nip.io 
✔              Cloud Manager UI: https://cm.192.168.0.10.nip.io/admin (default usr/pwd: admin/7iron-hide)
» common endpoints...
✔      Web Terminal: https://192.168.0.10:4200 
✔         Dashboard: http://192.168.0.10:32774/api/v1/namespaces/kube-system/services/http:kubernetes-dashboard:/proxy 
✔ OpenShift Console: https://192.168.0.10:8443/console 
» istio endpoints...
✔        Grafana: http://192.168.0.10:3000 
✔          Kiali: http://192.168.0.10:20001 
✔         Jaeger: http://192.168.0.10:15032 
✔     Prometheus: http://192.168.0.10:9090 
✔ Istio Bookinfo: http://192.168.0.10:31380/productpage 
Total elapsed time: 7 seconds
```
