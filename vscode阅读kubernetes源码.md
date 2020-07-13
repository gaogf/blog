## vscode阅读kubernetes源码

阅读环境是在Ubuntu18.04环境下搭建的。

## 一、准备工作

### 1.设置环境变量

```shell
export GOROOT=/usr/local/go/
export GOPATH=/home/gaogf/kubernetes
export PATH=$PATH:$GOROOT/bin
```

### 2.下载源码

```bash
sudo git clone https://github.com/kubernetes/kubernetes $GOPATH
git checkout -b 1.16 origin/release-1.16
```

### 3.安装一些工具

```shell
go get -u -v github.com/derekparker/delve/cmd/dlv
go get -u -v github.com/ramya-rao-a/go-outline  
go get -u -v github.com/acroca/go-symbols   
go get -u -v github.com/nsf/gocode  
go get -u -v github.com/rogpeppe/godef  
go get -u -v golang.org/x/tools/cmd/godoc   
go get -u -v github.com/zmb3/gogetdoc   
go get -u -v github.com/golang/lint/golint
go get -u -v github.com/fatih/gomodifytags
go get -u -v github.com/uudashr/gopkgs/cmd/gopkgs
go get -u -v golang.org/x/tools/cmd/gorename
go get -u -v sourcegraph.com/sqs/goreturns
go get -u -v github.com/cweill/gotests/...
go get -u -v golang.org/x/tools/cmd/guru
go get -u -v github.com/josharian/impl
go get -u -v github.com/haya14busa/goplay/cmd/goplay
```

## 二、安装docker



## 三、编译

分为两种编译方案。

### （1）本地二进制文件编译Kubernetes

修改运行平台配置参数（可选）

根据自己的运行平台（linux/amd64)修改hack/lib/golang.sh，把KUBE_SERVER_PLATFORMS，KUBE_CLIENT_PLATFORMS和KUBE_TEST_PLATFORMS中除linux/amd64以外的其他平台注释掉，以此来减少编译所用时间。

编译源码

进入Kubernetes根目录下

```shell
cd kubernetes
```

KUBE_BUILD_PLATFORMS指定目标平台，WHAT指定编译的组件，通过GOFLAGS和GOGCFLAGS传入编译时参数，如此处编译kubelet 组件。

```shell
KUBE_BUILD_PLATFORMS=linux/amd64 make all WHAT=cmd/kubelet GOFLAGS=-v GOGCFLAGS="-N -l"
###
如果不指定WHAT，则编译全部。
make all是在本地环境中进行编译的。
make release和make quick-release在容器中完成编译、打包成docker镜像。
编译kubelet这部分代码，也可执行make clean && make WHAT=cmd/kubelet
###
```

检查编译成果

编译过程较长，请耐心等待，编译后的文件在kubernetes/_output里。

或者进入cmd/kubelet (以kubelet为例子)

执行go build -v命令,如果没出错,会生成可执行文件kubelet

go build -v

生成的可执行文件在当前文件夹下面

```shell
ls cmd/kubelet/
app BUILD kubelet kubelet.go OWNERS
```

### (2)Docker镜像编译Kubernetes

查看kube-cross的TAG版本号

```bash
cat ./build/build-image/cross/VERSION
v1.12.12-1
```

这里，我使用DockerHub的Auto build功能，来构建K8s镜像。自然将编译需要用到的base镜像，放在了DockerHub上（也算是为促进国内K8s源码docker编译贡献绵薄之力吧！）。

```bash
docker pull registry.aliyuncs.com/google_container/pause-amd64:3.1
docker pull registry.aliyuncs.com/google_containers/kube-cross:v1.12.12-1
docker tag registry.aliyuncs.com/google_container/pause-amd64:3.1 k8s.gcr.io/pause-amd64:3.1
docker tag registry.aliyuncs.com/google_container/kube-cross:v1.12.12-1 k8s.gcr.io/kube-cross:v1.12.12-1
```

把build/lib/release.sh中的–pull去掉，避免构建镜像继续拉取镜像：

"${DOCKER[@]}" build --pull -q -t "${docker_image_tag}" ${docker_build_path} >/dev/null
修改为:
 "${DOCKER[@]}" build -q -t "${docker_image_tag}" ${docker_build_path} >/dev/null

编辑文件hack/lib/version.sh

将KUBE_GIT_TREE_STATE=”dirty” 改为 KUBE_GIT_TREE_STATE=”clean”，确保版本号干净。

执行编译命令

```shell
cd kubernetes
make clean
KUBE_BUILD_PLATFORMS=linux/amd64 KUBE_BUILD_CONFORMANCE=n KUBE_BUILD_HYPERKUBE=n make release-images GOFLAGS=-v GOGCFLAGS="-N -l"
```

其中KUBE_BUILD_PLATFORMS=linux/amd64指定目标平台为linux/amd64，GOFLAGS=-v开启verbose日志，GOGCFLAGS=”-N -l”禁止编译优化和内联，减小可执行程序大小。

编译的K8s Docker镜像以压缩包的形式发布在_output/release-tars目录中

```shell
ls _output/release-images/amd64/
cloud-controller-manager.tar 
kube-controller-manager.tar kube-scheduler.tarkube-apiserver.tar kube-proxy.tar
```

使用编译镜像

等待编译完成后，在_output/release-stage/server/linux-amd64/kubernetes/server/bin/目录下保存了编译生成的二进制可执行程序和docker镜像tar包。如导入kube-apiserver.tar镜像，并更新环境上部署的kube-apiserver镜像。

```shell
docker load -i kube-apiserver.tar
docker tag k8s.gcr.io/kube-apiserver:v1.16.4 registry.com/kube-apiserver:v1.16.4
```

整个编译过程结束后，现在就可以到master节点上，修改/etc/kubernetes/manifests/kube-apiserver.yaml描述文件中的image，修改完立即生效。

## 四、用vscode调试源码

```shell
code $GOPATH
```

设置断点

```shell
~/k8s/src/k8s.io/kubernetes/cmd/hyperkube/main.go 
```

**编辑 launch.json**

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Launch",
            "type": "go",
            "request": "launch",
            "mode": "exec",
            "remotePath": "",
            "port": 2345,
            "host": "127.0.0.1",
            "program": "/home/gaogf/kubernetes/src/k8s.io/kubernetes/cmd/hyperkube/hyperkube",
            "env": {},
            "args": ["kubelet",
            "--cluster-dns=10.96.0.10",
            "--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf",
            "--kubeconfig=/etc/kubernetes/kubelet.conf",
            "--v=2"],
            "showLog": true
        }
    ]
}
```

F5开始调试源码。