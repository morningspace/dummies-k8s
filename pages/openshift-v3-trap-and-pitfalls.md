# OpenShift v3的填坑之旅

---

> 记录All-in-One K8S Playground支持OpenShift v3沿途踩过的坑

<!-- Place this tag in your head or just before your close body tag. -->
<script async defer src="https://buttons.github.io/buttons.js"></script>

> 伴随着开源项目[lab-k8s-playground](https://github.com/morningspace/lab-k8s-playground) v1.2的发布，All-in-One K8S Playground现在可以同时支持标准Kubernetes和OpenShift啦！本文总结了在支持OpenShift v3.11过程中遇到的一系列问题。如果您喜欢这个项目，欢迎点击下面的按钮，关注，加星，以及贡献代码^_^

<a class="github-button" href="https://github.com/morningspace/lab-k8s-playground/subscription" data-icon="octicon-eye" data-size="large" aria-label="Watch morningspace/lab-k8s-playground on GitHub">Watch</a> <a class="github-button" href="https://github.com/morningspace/lab-k8s-playground" data-icon="octicon-star" data-size="large" aria-label="Star morningspace/lab-k8s-playground on GitHub">Star</a> <a class="github-button" href="https://github.com/morningspace/lab-k8s-playground/fork" data-icon="octicon-repo-forked" data-size="large" aria-label="Fork morningspace/lab-k8s-playground on GitHub">Fork</a>

## OpenShift

![](https://morningspace.github.io/assets/images/lab/k8s/openshift.png)

[OpenShift](https://www.openshift.com/)是Red Hat开发的构筑于Kubernetes之上的企业级容器应用平台，为企业应用与服务的开发，管理，以及部署提供了全面的支持，是当前炙手可热的一款Kubernetes发行版本。其社区版称为[OKD](https://www.okd.io/)（OpenShift Kubernetes Distribution的缩写，原名Origin）。

在[All-in-One K8S Playground新增OpenShift支持](/tech/all-in-one-openshift-playground/)一文中，我们已经了解了，如何利用[All-in-One Kubernetes Playground](https://github.com/morningspace/lab-k8s-playground/)在单机环境下部署一个支持OKD v3.11版本的单节点集群，以及如何在上面部署[Istio](https://istio.io)和[API Connect](https://developer.ibm.com/apiconnect)。

本文将告诉你，All-in-One K8S Playground在支持OpenShift v3.11过程中遇到的一些问题，以及相应的解决办法。这其中，大部分问题的解决办法都已经通过脚本的形式固化在了All-in-One K8S Playground里，这也是该Playground的价值所在！

## 坑#1：Web Console登录总是重定向到127.0.0.1

在本机启动OpenShift集群时，服务器的访问地址基本都是`127.0.0.1`。但有时候，我们也会在远程机器上部署OpenShift集群，这个时候的服务器访问地址通常就是某个具体的IP地址或者hostname了。

在利用`oc cluster up`命令启动OpenShift的时候，OpenShift为我们提供了一个参数`--public-hostname`，用来指定集群启动起来以后服务器的访问地址。比如：
```shell
$ oc cluster up --public-hostname=$HOST_IP
```

但即便如此，我们也经常会遇到OpenShift在登录Web Console时自动重定向到`127.0.0.1`这样的情况。这往往是由于，我们在上一次启动OpenShift时使用了`127.0.0.1`，而在本次启动OpenShift时又没有清除之前启动时留在本地的配置文件目录。这个时候，可以先清除本地OpenShift的配置文件目录，然后再启动OpenShift。

> 在Playground里，配置文件目录在`~/openshift.local.clusterup`下，执行`launch kubernetes::clean`时会为我们清除掉这个目录。

有关这一问题的详细讨论，参见：

| 网络资源
| ----
| [Access fail with 'oc cluster up --public-hostname=public ip' and redirect to 127.0.0.1](https://github.com/openshift/origin/issues/20726)

## 坑#2：Admission Webhook默认没有开启

在部署类似Istio这样的应用时，需要我们的Kubernetes集群开启[Admission Webhook](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/)。Kubernetes从v1.11开始默认就开启了这项功能。但是，OpenShift直到v3.11为止，默认都还没有开启它。

事实上，OpenShift的[GitHub库](https://github.com/openshift/origin)里是有专门的[PR](https://github.com/openshift/origin/pull/20953)用来解决这一问题的。但是由于种种原因，最后并没有进入v3.11。

在v3.11上解决这一问题的方法，可以参考[Minishift](https://github.com/minishift/minishift)的这个[PR](https://github.com/minishift/minishift/pull/3044)。

> 在Playground里，执行`launch istio`时，会自动完成对OpenShift配置的修改以开启Admission Webhook，并自动重启OpenShift的API服务，以使配置修改生效。

有关这一问题的详细讨论，参见：

| 网络资源
| ----
| [Consider enabling admission webhooks by default with oc cluster up](https://github.com/openshift/origin/issues/20530)
| [Enable mutating and validating webhooks](https://github.com/minishift/minishift/issues/2676)

## 坑#3：API服务重启后不能立刻对外提供服务

为了启动Admission Webhook，我们需要对OpenShift的配置进行修改，并重启它的API服务。关于重启API服务的方法，可以参考[Minishift](https://github.com/minishift/minishift)的这个[PR](https://github.com/minishift/minishift/pull/3044)。

做为自动化脚本的一部分，我们需要在检测到API服务成功重启以后，继续后面的部署工作。为此，参照Minishift的做法，我们可以通过访问`/healthz`来判断OpenShift的API服务是否已经成功启动：
```shell
$ curl -k https://$HOST_IP:8443/healthz
```

通常，当`/healthz`返回`ok`的时候，就表示API服务可以对外提供服务了。但是，在实际测试的过程中发现，即使`/healthz`返回`ok`，某些OpenShift的操作仍然会失败。比如：
```shell
$ oc login -u system:admin
Error from server (InternalError): Internal error occurred: unexpected response: 400
```

> 在Playground里，当重启API服务时，除了检测`/healthz`，还增加了额外的检测机制，以保证重启后的API服务能够真正对外提供服务。

有关这一问题的详细讨论，参见：

| 网络资源
| ----
| [How to detect if apiserver is fully restarted and ready for use](https://github.com/openshift/origin/issues/23864)

## 坑#4：HostPath PV报Permission denied

部署API Connect时，我们使用了HostPath类型的Persistent Volume。但很多Pod在启动过程中都会遇到创建目录或文件失败的情况，从而导致Pod启动失败。比如：
```shell
$ klo r31f4a26f5e-apiconnect-cc-0 -n apiconnect
mkdir: cannot create directory '/var/db/data': Permission denied
mkdir: cannot create directory '/var/db/logs': Permission denied
```

这是因为大多数API Connect的Pod都以非root用户运行，而在宿主机上创建的Persistent Volume目录，默认权限都是`drwxr-xr-x`，即：`group`和`other`对目录都没有写的权限，导致Pod在这些目录下无法创建子目录或文件。

根据实际测试，除了按照API Connect的文档，把`anyuid`这个Security Context Constraint赋给`apiconnect`名字空间下的所有service account之外：
```shell
$ oc adm policy add-scc-to-group anyuid system:serviceaccounts:apiconnect
```

我们还需要为Persistent Volume目录及其子目录和文件赋予更大的权限：
```shell
$ chmod ugo+rwx -R /path/to/apic/pv
```

> 在Playground里，`launch apic`会帮我们自动创建API Connect运行所需的Persistent Volume目录及其子目录，并为它们赋予足够的权限。

有关这一问题的详细讨论，参见：

| 网络资源
| ----
| [Directories provisioned by hostPath provisioner are only writeable by root](https://github.com/kubernetes/minikube/issues/1990)
| [Permission denied in pod](https://github.com/openshift/origin/issues/9131)
| [OpenShift: insufficient permission inside the containers](https://github.com/rook/rook/issues/1314)

## 坑#5：OpenShift的DNS服务问题

API Connect在运行期间需要通过指定的域名来访问它所提供的服务，这就要用到DNS服务了。标准的Kubernetes从v1.13开始就默认使用[CoreDNS](https://coredns.io/)来替代原来的Kube-DNS实现了。基于CoreDNS，我们可以利用它的[hosts](https://coredns.io/plugins/hosts/)插件为API Connect指定静态DNS规则，就像在`/etc/hosts`里配置`IP-Host`映射一样。

而OpenShift的DNS服务，到v3.11为止，一直都是基于一个叫做[SkyDNS](https://github.com/skynetservices/skydns)的项目实现的。通过尝试发现，它并没有提供类似CoreDNS的hosts插件那样的方法对DNS进行定制。因此，我们需要为OpenShift配置额外的DNS服务器，甚至可能要搭建自己的DNS服务器。对于只是配置静态DNS规则这样的简单需求而言，这个方案就有点过于复杂了。

> 目前，Playground采用的解决办法是基于[nip.io](https://nip.io/)的公共DNS服务，实现域名到IP的自动转换。事实上，OpenShift通过route定义暴露集群服务默认也是用的nip.io。

根据OpenShift 4.1的[Release Notes](https://docs.openshift.com/container-platform/4.1/release_notes/ocp-4-1-release-notes.html)，似乎从v4.x开始，OpenShift用CoreDNS取代了原来的SkyDNS。从相应的[PR](https://github.com/openshift/origin/pull/22270)里可以看到，所有和SkyDNS相关的逻辑也都被清除掉了。

有关这一问题的详细讨论，参见：

| 网络资源
| ----
| [What's the simplest way to setup IP host mappings for OpenShift ... cluster?](https://github.com/openshift/origin/issues/23769)
| [Why OKD 3.11 with Kubernetes 1.11,but not use CoreDNS?](https://github.com/openshift/origin/issues/21422)

## 坑#6：openshift.local.volumes目录删除失败

当我们用`oc cluster down`把OpenShift集群停掉时，有时也希望把OpenShift留在本地的配置文件目录一并删除掉。但在CentOS和RHEL上，删除这个目录时会报错：
```shell
$ rm -rf ~/openshift.local.clusterup
rm: cannot remove ‘openshift.local.clusterup/openshift.local.volumes/pods/916be4fd-e32f-11e9-b688-005056bcdc55/volumes/kubernetes.io~secret/serving-cert’: Device or resource busy
...
```

这是由于OpenShift集群在运行期间会为Pod在宿主机上设置mount点，而停掉集群的时候，这些mount点由于某些原因，并没有被正确地unmount。解决这一问题的方法是手工对它们进行unmount，具体做法可以参考这个[PR](https://github.com/openshift/origin/pull/4982)。

> 在Playground里，执行`launch kubernetes::clean`时会自动帮我们寻找当前还没有被unmount的与OpenShift有关的目录，并逐一执行unmount。

有关这一问题的详细讨论，参见：

| 网络资源
| ----
| [Umount secret volumes before trying to remove](https://github.com/openshift/origin/pull/2629)

## 坑#7：Docker Desktop在Mac上的代理问题

在Mac上启动Openshift时，会在集群启动结束时见到下面的警告信息：
```shell
WARNING: An HTTP proxy (gateway.docker.internal:3128) is configured for the Docker daemon, but you did not specify one for cluster up
WARNING: An HTTPS proxy (gateway.docker.internal:3129) is configured for the Docker daemon, but you did not specify one for cluster up
WARNING: A proxy is configured for Docker, however 172.30.1.1 is not included in its NO_PROXY list.
   172.30.1.1 needs to be included in the Docker daemon's NO_PROXY environment variable so pushes to the local OpenShift registry can succeed.
```

这是由于Docker Desktop for Mac从某个版本开始引入了一种内部代理机制，叫做“transparent proxy”。如果执行`docker info`命令，我们可以看到，`HTTP Proxy`和`HTTPS Proxy`这两项被设置成了相应的值：
```shell
$ docker info|grep -i proxy
 HTTP Proxy: gateway.docker.internal:3128
 HTTPS Proxy: gateway.docker.internal:3129
```

即便运行Docker时不需要代理，这两个值也是始终存在的。OpenShift在启动时提供了相应的命令行参数：`--http-proxy`，`--https-proxy`，可以帮我们把这两个值传给OpenShift。

另一方面，做为OpenShift的内部Docker Registry，IP地址为`172.30.1.11`，是不需要经过代理的，因此它应该出现在`NO_PROXY`环境变量里。

但问题是，我们通过Docker Desktop提供的界面对`NO_PROXY`的设置并不起作用（Preferences... > Proxies > Manual proxy configuration > Bypass proxy settings for these hosts & domains）。

目前为止，Docker Desktop的最新版本还没有修复这个问题。不过，据称它的17.09.1-ce-mac42版本是可以工作的。因此，现在解决这一问题的唯一方法是安装低版本的Docker Desktop。

有关这一问题的详细讨论，参见：

| 网络资源
| ----
| [Docker for Mac `no_proxy` not honored](https://github.com/docker/for-mac/issues/2732)
| [Unwanted Proxy](https://github.com/docker/for-mac/issues/2467)
| [Application deployment failing when cluster setup using oc cluster up on Mac](https://github.com/openshift/origin/issues/19528)
| [cluster up fails to push to the registry on Docker for Mac CE 17.12.0 and newer](https://github.com/openshift/origin/issues/18596)
| [oc cluster up does not work in a proxied environment](https://github.com/openshift/origin/issues/11323)
| [No persistent volumes available in OpenShift Origin 3.7.1](https://lists.openshift.redhat.com/openshift-archives/users/2018-February/msg00017.html)

## 坑#8：/var/lib/kubelet/device-plugins问题

在Mac上启动OpenShift还会遇到下面的错误，导致集群启动失败：
```shell
Mounts denied: The path /var/lib/kubelet/device-plugins is not shared from OS X and is not known to Docker. You can configure shared paths from Docker -> Preferences... -> File Sharing. See https://docs.docker.com/docker-for-mac/osxfs/#namespaces for more info.
```

一种解决办法，是通过Docker Desktop for Mac的Preferences界面，先把Kubernetes开启，然后再把它关闭：Preferences... > Kubernetes > Enable Kubernetes。

有关这一问题的详细讨论，参见：

| 网络资源
| ----
| [openshift v3.11 does not start on Mac OS X](https://github.com/openshift/origin/issues/21230)

## 坑#9：OpenShift集群在Mac上重启失败

这个问题同样是和Mac相关的，当我们在先前已经成功启动过OpenShift集群的Mac机器上再次启动OpenShift时，会遇到因为端口被占用的错误而导致重启失败：
```
Getting a Docker client ...
Checking if image openshift/origin-control-plane:v3.11 is available ...
Checking type of volume mount ...
Determining server IP ...
Checking if OpenShift is already running ...
Checking for supported Docker version (=>1.22) ...
Checking if insecured registry is configured properly in Docker ...
Checking if required ports are available ...
error: a port needed by OpenShift is not available
```

这是由于，Docker Desktop默认会把Kubernetes相关的系统级Docker容器隐藏起来，包括：那些`k8s_POD_`打头的容器，以及`kube_system`名字空间下的容器，它们用`docker ps`是查不到的。

另一方面，`oc cluster down`在执行时由于获取不到这些系统级容器，所以也就无法停掉它们。OpenShift集群运行所需要的端口，就是被这些容器占用的。利用`lsof`命令可以看到，具体的端口包括`1936`，`443`，`80`：
```shell
$ lsof -Pni|grep com.docke.*TCP.*LISTEN
com.docke 45430 moyingbj    7u  IPv4 0x6dcd0b5b57f496b1      0t0  TCP 127.0.0.1:56861 (LISTEN)
com.docke 45434 moyingbj   15u  IPv6 0x6dcd0b5b2df9e889      0t0  TCP *:1936 (LISTEN)
com.docke 45434 moyingbj   17u  IPv6 0x6dcd0b5b3066f2c9      0t0  TCP *:443 (LISTEN)
com.docke 45434 moyingbj   20u  IPv6 0x6dcd0b5b3066ed09      0t0  TCP *:80 (LISTEN)
```

解决的办法同样是通过Docker Desktop for Mac的Preferences界面，首先开启Kubernetes，然后确保勾选了“Show system containers (advanced)”，最后再关闭Kubernetes。

![](https://morningspace.github.io/assets/images/lab/k8s/docker-mac-k8s-settings.png)

有关这一问题的详细讨论，参见：

| 网络资源
| ----
| [error: a port needed by OpenShift is not available](https://github.com/openshift/origin/issues/22381)
| [oc cluster down leave hypershift process running when using docker for mac](https://github.com/openshift/origin/issues/19486)
