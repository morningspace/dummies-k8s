# All-in-One K8S Playground支持OpenShift v4

---

> All-in-One K8S Playground v1.3发布，OpenShift v3+v4一勺烩

<!-- Place this tag in your head or just before your close body tag. -->
<script async defer src="https://buttons.github.io/buttons.js"></script>

> 开源项目[lab-k8s-playground](https://github.com/morningspace/lab-k8s-playground) v1.3发布，All-in-One K8S Playground现在可以支持最新的OpenShift v4啦！如果您喜欢这个项目，欢迎点击下面的按钮，关注，加星，以及贡献代码^_^

<a class="github-button" href="https://github.com/morningspace/lab-k8s-playground/subscription" data-icon="octicon-eye" data-size="large" aria-label="Watch morningspace/lab-k8s-playground on GitHub">Watch</a> <a class="github-button" href="https://github.com/morningspace/lab-k8s-playground" data-icon="octicon-star" data-size="large" aria-label="Star morningspace/lab-k8s-playground on GitHub">Star</a> <a class="github-button" href="https://github.com/morningspace/lab-k8s-playground/fork" data-icon="octicon-repo-forked" data-size="large" aria-label="Fork morningspace/lab-k8s-playground on GitHub">Fork</a>

## OpenShift v4

![](https://morningspace.github.io/assets/images/lab/k8s/openshift.png)

[OpenShift](https://www.openshift.com/)是Red Hat开发的构筑于Kubernetes之上的企业级容器应用平台，为企业应用与服务的开发，管理，以及部署提供了全面的支持，是当前炙手可热的一款Kubernetes发行版本。其社区版称为[OKD](https://www.okd.io/)（OpenShift Kubernetes Distribution的缩写，原名Origin）。

OpenShift v4是目前OpenShift的最新版本，和v3相比，新版本从功能，架构，到实现，都有非常巨大的变化，当然这也包括安装方式的变化。从v4开始，OpenShift不再支持`oc cluster up`方式的单节点集群部署，而是开发了专门的[Installer](https://github.com/openshift/installer)，并在此基础上提供了一种在本机部署的单节点集群安装方式，叫做[CodeReady Containers](https://github.com/code-ready/crc)（简称为`crc`）。

尽管v3和v4有着天壤之别，通过本文将会看到，利用[All-in-One Kubernetes Playground](https://github.com/morningspace/lab-k8s-playground/)，我们依然可以沿用和之前同样的命令行命令，启动OpenShift v4的单节点集群。这是因为，Playground为我们屏蔽了差异，使我们可以用统一的方式，在单机环境下部署：多节点的标准Kubernetes集群，以及单节点的OpenShift v3或v4集群。

不仅如此，像[Istio](https://istio.io)这样的应用，也已经在这一版本的Playground被成功迁移到了OpenShift v4集群上。只需要简单几条命令，就可以完成部署，整个过程是完全自动化进行的，并且可以无限次重复！

## 准备环境

在[All-in-One K8S Playground 中文使用指南](http://localhost:4000/tech/all-in-one-k8s-playground/)以及[All-in-One K8S Playground新增OpenShift支持](/tech/all-in-one-openshift-playground/)一文里，我们已经学会了如何在单机环境下快速自动部署起一个支持多节点的标准Kubernetes集群，以及支持单节点的OpenShift v3集群。如果要想让Playground支持OpenShift v4，只要把`K8S_PROVIDER`这个环境变量的值设置成`crc`就可以了。

```shell
export K8S_PROVIDER=crc
```

目前，该环境变量支持的合法值包括：
* `dind`，代表基于DIND技术（即Docker-in-Docker）实现的，可在单机环境里运行的，多节点标准Kubernetes集群；
* `oc`，代表基于OpenShift的`oc cluster up`命令实现的，可在单机环境里运行的，单节点OpenShift 3.x集群；
* `crc`，代表基于OpenShift的[CodeReady Containers](https://github.com/code-ready/crc/)实现的，可在单机环境里运行的，单节点OpenShift v4集群；

执行下面的命令，可以验证当前环境变量的设置是否正确：
```shell
$ launch env
Targets to be launched: [env]
####################################
# Launch target env...
####################################
Common:
  LAB_HOME=/root/lab-playgrounds/lab-k8s-playground
  HOST_IP     : 127.0.0.1
  K8S_PROVIDER: crc

Specific to crc:
  CRC_VERSION          : 1.0.0
  CRC_MEMORY           : 10240
  CRC_CPUS             : 4
  CRC_USE_VIRTUALBOX   : 

Total elapsed time: 0 seconds
```

通过相关环境变量的设置，我们还可以选择特定的crc版本，集群所占主机内存与CPU资源的大小，以及是否选择VirtualBox方式部署。

crc会在宿主机上首先启动一个虚拟机，然后再在虚拟机里跑OpenShift。通常，它会在不同的操作系统环境下自动选择相应的虚拟机技术。比如：
* 在MacOS上，使用Hyperkit；
* 在Windows上，使用Hyper-V；
* 在Linux上，使用KVM；

或者，通过把`CRC_USE_VIRTUALBOX`设置为1，我们也可以主动选择VirtualBox，这种跨操作系统的虚拟机技术。

> 目前crc可以支持在Linux，MacOS，Windows上运行。

## 启动OpenShift v4

启动OpenShift v4集群所用的命令和启动OpenShift v3以及标准Kubernetes集群是一模一样的：
```shell
$ launch kubernetes
```

等命令执行完毕以后，Playground会为我们在当前主机上启动起一个单节点的OpenShift v4集群。

> 在OpenShift启动过程中，可能会提示输入Pull Secret。我们可以从[这里](https://cloud.redhat.com/openshift/install/crc/installer-provisioned)找到自己的Pull Secret。如果把Pull Secret下载下来，并以`pull-secret.txt`为文件名保存到当前用户主目录下的`.crc`子目录，那么Playground会自动检测到并读取它，这样就不会在OpenShift启动过程中再提示用户输入了。

执行如下命令，我们可以看到OpenShift Web Console的访问地址：
```shell
$ launch endpoints
Targets to be launched: [endpoints]
####################################
# Launch target endpoints...
####################################
Common:
  ✗ Web Terminal     : https://127.0.0.1:4200 
  ✔ OpenShift Console: https://console-openshift-console.apps-crc.testing (admin usr/pwd: kubeadmin/F44En-Xau6V-jQuyb-yuMXB)

Total elapsed time: 0 seconds
```

## 使用OpenShift v4

根据`launch endpoints`返回结果中给出的管理员用户名和密码，我们可以登录到OpenShift所提供的Web Console里，对应用和集群进行管理。

如果希望通过命令行来管理应用和集群，则可以利用OpenShift提供的`oc`命令进行登录。以管理员身份登录：
```shell
$ oc login -u kubeadmin -p <password>
```

或者以开发人员身份登录：
```shell
$ oc login -u developer -p developer
```

## 在OpenShift v4上运行Istio

在All-in-One K8S Playground里基于OpenShift v4运行Istio只需要一条命令就能实现。而且，这条命令和基于标准Kubernetes集群，以及OpenShift v3运行Istio是一模一样的！关于如何在标准Kubernetes集群里运行Istio，请参见[All-in-One K8S Playground 中文使用指南](http://localhost:4000/tech/all-in-one-k8s-playground/)一文；在OpenShift v3上执行，则参见[All-in-One K8S Playground新增OpenShift支持](/tech/all-in-one-openshift-playground/)一文。

等Kubernetes集群启动起来以后，执行如下命令启动Istio，以及它的Bookinfo示例应用：
```shell
$ launch istio istio-bookinfo
```

由于crc是在宿主机上跑着的一个虚拟机里运行OpenShift的，它所使用的网络是一个只在当前宿主机范围内可见的本地局域网，所以通常我们只能在本机对OpenShift集群，以及集群里的应用进行访问。如果希望从另一台机器上远程访问这个集群，可以使用反向代理。Playground里已经为我们预装了nginx，如果要远程访问Istio的应用，可以执行如下命令：
```shell
$ launch istio::expose istio-bookinfo::expose
```

它会自动帮我们更新nginx的配置，加入对Istio各个应用的请求转发规则，从而把Istio在本机的各个应用服务暴露给其他机器。

另外，如果要远程访问OpenShift v4的Web Console，则依然使用和本地访问相同的URL地址。为此，需要在本机的`/etc/hosts`文件里添加相应配置，比如：
```shell
192.168.0.10 api.crc.testing oauth-openshift.apps-crc.testing console-openshift-console.apps-crc.testing
```

这里的`192.168.0.10`就是OpenShift集群所在机器的IP地址。对于其他应用的访问地址，由于使用了[`nip.io`](https://nip.io/)提供的DNS服务，所以不需要把主机IP和主机名的映射定义添加到本地的`/etc/hosts`文件里了。

如果想得到包括Web Console在内的当前集群里所有应用的访问地址，可以通过运行`launch endpoints`命令查看到。
```shell
$ launch endpoints 
Targets to be launched: [endpoints]
####################################
# Launch target endpoints...
####################################
Common:
  ✗ Web Terminal     : https://127.0.0.1:4200 
  ✔ OpenShift Console: https://console-openshift-console.apps-crc.testing (admin usr/pwd: kubeadmin/F44En-Xau6V-jQuyb-yuMXB)

Istio:
  ✔ Grafana       : http://grafana-istio-system.apps-crc.testing 
  ✔ Kiali         : http://kiali-istio-system.apps-crc.testing 
  ✔ Jaeger        : http://jaeger-query-istio-system.apps-crc.testing 
  ✔ Prometheus    : http://prometheus-istio-system.apps-crc.testing 
  ✔ Istio Bookinfo: http://istio-ingressgateway-istio-system.apps-crc.testing/productpage 

Istio proxied:
  ✔ Grafana       : http://grafana-istio-system.192.168.0.10.nip.io
  ✔ Kiali         : http://kiali-istio-system.192.168.0.10.nip.io 
  ✔ Jaeger        : http://jaeger-query-istio-system.192.168.0.10.nip.io 
  ✔ Prometheus    : http://prometheus-istio-system.192.168.0.10.nip.io 
  ✔ Istio Bookinfo: http://istio-ingressgateway-istio-system.192.168.0.10.nip.io/productpage 

Total elapsed time: 2 seconds
```
