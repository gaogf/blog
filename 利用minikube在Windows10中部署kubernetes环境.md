# 利用minikube在Windows10中部署kubernetes环境

​	为了有一个属于自己的kubernetes环境，便于学习验证kubernetes的操作。社区也提供了便于本地安装的minikube，但由于众所周知的网络环境，很多人无法直接使用minikube,本文利用最新的minikube在本机（Windows 10）环境下部署kubernetes环境。

Minikube支持以下Kubernetes功能：

- DNS
- NodePorts（可使用“minikube service”命令来管理）
- ConfigMaps和Secrets
- 仪表板（Dashboards，minikube dashboard）
- 容器运行时：Docker，*rkt*，*CRI-O*和*containerd*
- Enabling CNI（容器网络接口）
- Ingress
- LoadBalancer（负载均衡，可以使用“minikube tunnel”命令来启用）
- Multi-cluster（多集群，可以使用“minikube start -p <name>”命令来启用）
- Persistent Volumes
- RBAC
- 通过命令配置apiserver和kubelet

​	主要软件

[minikube.exe](https://github.com/kubernetes/minikube/releases/latest) : 下载后将名称改为`minikube.exe`，并加入环境变量PATH中；这里使用最新版的1.7.3

[kubectl.exe](https://storage.googleapis.com/kubernetes-release/release/v1.17.3/bin/windows/amd64/kubectl.exe) : 下载最新版本的1.17.3，最新的版本信息可以通过https://storage.googleapis.com/kubernetes-release/release/stable.txt这里查看。下载后加入环境变量PATH中。

​	基础工作做完了，就是这么简单。

​	由于通过minikube部署kubernetes环境，需要虚拟机来支撑，为了不使环境引入更多的软件，使用Windows10自带的hyper-v虚拟机。

![image-20200301111202057](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20200301111202057.png)

点击“启用或关闭Windows功能”，

![image-20200301111242308](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20200301111242308.png)

​	勾选Hyper-V,如果之前没有开启，勾选后需要重启系统。

​	基础工作做完了，接下来就开始正式的部署工作。

​	1.以管理员身份打开powershell或者cmd。运行以下命令：

```bash
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

​	2.以管理员身份在hyper-v虚拟机管理器中新增一个**外部**虚拟交换机。

![image-20200301111830003](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20200301111830003.png)

需要关闭虚拟机内存动态资源配置

```	
Set-VMMemory -VMName 'minikube' -DynamicMemoryEnabled $false
```

3.启动

```bash
--iso-url=*** 利用阿里云的镜像地址下载相应的 .iso 文件
--image-mirror-country cn 将缺省利用 registry.cn-hangzhou.aliyuncs.com/google_containers 作为安装Kubernetes的容器镜像仓库
--kubernetes-version=***: minikube 虚拟机将使用的 kubernetes 版本
##注册阿里云开发者账号，获取加速器：https://9a36ojpb.mirror.aliyuncs.com
完整的命令：
minikube start --image-mirror-country cn --iso-url=https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/iso/minikube-v1.7.3.iso --registry-mirror=https://9a36ojpb.mirror.aliyuncs.com --vm-driver="hyperv" --hyperv-virtual-switch="minikube" --memory=4096 --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers
```

![image-20200301114334027](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20200301114334027.png)

4.查看控制台

```bash
minikube dashboard
```

接下来就可以在kubernetes的世界自由翱翔了。