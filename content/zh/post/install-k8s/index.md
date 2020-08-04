---
title: "centos7.2 安装k8s v1.11.0"
date: 2018-08-14T20:07:03+08:00
draft: false
type: blog
tags: ["容器", "kubernetes"]
categories: ["部署安装"]
banner: "http://wx4.sinaimg.cn/large/ad5fbf65ly1g0t1a2z4m7j21rt15otlh.jpg"
authors: ["guoxudong"]
authorlink: "https://github.com/sunny0826"
summary: "最近由于公司业务发展到了瓶颈，原有的技术架构已经逐渐无法满足业务开发和测试的需求，出现了应用测试环境搭建复杂，有许多套（真的很多很多）应用环境，应用在持续集成/持续交付也遇到了很大的困难，经过讨论研究决定对应用和微服务进行容器化，这就是我首次直面docker和k8s的契机。"
keywords: ["容器", "kubernetes"]
image:
  url: "https://tvax1.sinaimg.cn/large/ad5fbf65ly1ge3ilc4jkvj21rt15otlh.jpg"
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
# 前言
>最近由于公司业务发展到了瓶颈，原有的技术架构已经逐渐无法满足业务开发和测试的需求，出现了应用测试环境搭建复杂，有许多套（真的很多很多）应用环境，应用在持续集成/持续交付也遇到了很大的困难，经过讨论研究决定对应用和微服务进行容器化，这就是我首次直面docker和k8s的契机.

# Kubernetes 介绍
Kubernetes 是 Google 团队发起的开源项目，它的目标是管理跨多个主机的容器，提供基本的部署，维护以及运用伸缩，主要实现语言为
Go 语言。
Kubernetes的特点：

* 易学：轻量级，简单，容易理解
* 便携：支持公有云，私有云，混合云，以及多种云平台
* 可拓展：模块化，可插拔，支持钩子，可任意组合
* 自修复：自动重调度，自动重启，自动复制

# 准备工作
**注：以下操作都是在root权限下执行的**

1. 安装docker-ce，这里使用docker-ce-17.09.0.c版本，安装方法见[之前的教程](/2018/install-docker)
2. 安装Kubeadm
        
    ```bash
    #安装 Kubeadm 首先我们要配置好阿里云的国内源，执行如下命令：
    cat <<EOF > /etc/yum.repos.d/kubernetes.repo
    [kubernetes]
    name=Kubernetes
    baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
    enabled=1
    gpgcheck=0
    EOF

    #之后，执行以下命令来重建yum缓存：
    yum -y install epel-releaseyum
    clean all
    yum makecache
    ```

    接下来需要安装指定版本的Kubeadm（这里要安装指定版本，因为后续依赖的镜像由于有墙无法拉取，这里我们只有指定版本的镜像），注意：**这里是安装指定版本的Kubeadm，k8s的版本更新之快完全超出你的想象！**

    ```bash
    yum -y install kubelet-1.11.0-0
    yum -y install kubeadm-1.11.0-0
    yum -y install kubectl-1.11.0-0
    yum -y install kubernetes-cni
    
    #执行命令启动Kubeadm服务：
    systemctl enable kubelet && systemctl start kubelet
    ```
3. 配置 Kubeadm 所用到的镜像
这里是重中之重，因为在国内的原因，无法访问到 Google 的镜像库，所以我们需要执行以下脚本来从 Docker Hub 仓库中获取相同的镜像，并且更改 TAG 让其变成与 Google 拉去镜像一致。

    **新建一个 Shell 脚本，填入以下代码之后保存**

    ```vim
    #docker.sh
    #!/bin/bash
    images=(kube-proxy-amd64:v1.11.0 kube-scheduler-amd64:v1.11.0 kube-controller-manager-amd64:v1.11.0 kube-apiserver-amd64:v1.11.0 etcd-amd64:3.2.18 coredns:1.1.3 pause-amd64:3.1 kubernetes-dashboard-amd64:v1.8.3 k8s-dns-sidecar-amd64:1.14.9 k8s-dns-kube-dns-amd64:1.14.9 k8s-dns-dnsmasq-nanny-amd64:1.14.9 )
    for imageName in ${images[@]} ; do
    docker pull keveon/$imageName
    docker tag keveon/$imageName k8s.gcr.io/$imageName
    docker rmi keveon/$imageName
    done
    # 个人新加的一句，V 1.11.0 必加
    docker tag da86e6ba6ca1 k8s.gcr.io/pause:3.1
    ```

    **保存后使用chmod命令赋予脚本执行权限**

    ```bash
    chmod -R 777 ./docker.sh
    ```

    **执行脚本拉取镜像**

    ```bash
    sh docker.sh
    #这里就开始了漫长的拉取镜像之路
    ```

    **关闭掉swap**

    ```bash
    sudo swapoff -a
    #要永久禁掉swap分区，打开如下文件注释掉swap那一行
    # sudo vi /etc/stab
    ```

    **关闭SELinux的**

    ```bash
    # 临时禁用selinux
    # 永久关闭 修改/etc/sysconfig/selinux文件设置
    sed -i 's/SELINUX=permissive/SELINUX=disabled/' /etc/sysconfig/selinux
    # 这里按回车，下面是第二条命令
    setenforce 0
    ```

    **关闭防火墙**

    ```bash
    systemctl disable firewalld.service && systemctl stop firewalld.service
    ```

    **配置转发参数**

    ```bash
    # 配置转发相关参数，否则可能会出错
    cat <<EOF > /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    vm.swappiness=0
    EOF
    # 这里按回车，下面是第二条命令
    sysctl --system
    ```

    **这里就完成了k8s集群搭建的准备工作，集群搭建的话以上操作结束后将操作完的系统制作成系统镜像，方便集群搭建**

# 正式安装
**以下的操作都只在主节点上进行：**

**初始化镜像**

```bash
kubeadm init --kubernetes-version=v1.11.0 --pod-network-cidr=10.10.0.0/16  #这里填写集群所在网段
```

**之后的输出会是这样：**

```go
I0712 10:46:30.938979   13461 feature_gate.go:230] feature gates: &{map[]}
[init] using Kubernetes version: v1.11.0
[preflight] running pre-flight checks
I0712 10:46:30.961005   13461 kernel_validator.go:81] Validating kernel version
I0712 10:46:30.961061   13461 kernel_validator.go:96] Validating kernel config
    [WARNING SystemVerification]: docker version is greater than the most recently validated version. Docker version: 18.03.1-ce. Max validated version: 17.03
    [WARNING Hostname]: hostname "g2-apigateway" could not be reached
    [WARNING Hostname]: hostname "g2-apigateway" lookup g2-apigateway on 100.100.2.138:53: no such host
[preflight/images] Pulling images required for setting up a Kubernetes cluster
[preflight/images] This might take a minute or two, depending on the speed of your internet connection
[preflight/images] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[preflight] Activating the kubelet service
[certificates] Generated ca certificate and key.
[certificates] Generated apiserver certificate and key.
[certificates] apiserver serving cert is signed for DNS names [g2-apigateway kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 172.16.8.62]
[certificates] Generated apiserver-kubelet-client certificate and key.
[certificates] Generated sa key and public key.
[certificates] Generated front-proxy-ca certificate and key.
[certificates] Generated front-proxy-client certificate and key.
[certificates] Generated etcd/ca certificate and key.
[certificates] Generated etcd/server certificate and key.
[certificates] etcd/server serving cert is signed for DNS names [g2-apigateway localhost] and IPs [127.0.0.1 ::1]
[certificates] Generated etcd/peer certificate and key.
[certificates] etcd/peer serving cert is signed for DNS names [g2-apigateway localhost] and IPs [172.16.8.62 127.0.0.1 ::1]
[certificates] Generated etcd/healthcheck-client certificate and key.
[certificates] Generated apiserver-etcd-client certificate and key.
[certificates] valid certificates and keys now exist in "/etc/kubernetes/pki"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/admin.conf"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/kubelet.conf"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/controller-manager.conf"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/scheduler.conf"
[controlplane] wrote Static Pod manifest for component kube-apiserver to "/etc/kubernetes/manifests/kube-apiserver.yaml"
[controlplane] wrote Static Pod manifest for component kube-controller-manager to "/etc/kubernetes/manifests/kube-controller-manager.yaml"
[controlplane] wrote Static Pod manifest for component kube-scheduler to "/etc/kubernetes/manifests/kube-scheduler.yaml"
[etcd] Wrote Static Pod manifest for a local etcd instance to "/etc/kubernetes/manifests/etcd.yaml"
[init] waiting for the kubelet to boot up the control plane as Static Pods from directory "/etc/kubernetes/manifests"
[init] this might take a minute or longer if the control plane images have to be pulled
[apiclient] All control plane components are healthy after 41.001672 seconds
[uploadconfig] storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.11" in namespace kube-system with the configuration for the kubelets in the cluster
[markmaster] Marking the node g2-apigateway as master by adding the label "node-role.kubernetes.io/master=''"
[markmaster] Marking the node g2-apigateway as master by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[patchnode] Uploading the CRI Socket information "/var/run/dockershim.sock" to the Node API object "g2-apigateway" as an annotation
[bootstraptoken] using token: o337m9.ceq32wg9g2gro7gx
[bootstraptoken] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstraptoken] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstraptoken] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstraptoken] creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

kubeadm join 10.10.207.253:6443 --token t69z6h.lr2etdbg9mfx5r15 --discovery-token-ca-cert-hash sha256:90e3a748c0eb4cb7058f3d0ee8870ee5d746214ab0589b5e841fd5d68fec8f00
```

**这里注意最后一行：**

```go
kubeadm join 10.10.207.253:6443 --token t69z6h.lr2etdbg9mfx5r15 --discovery-token-ca-cert-hash sha256:90e3a748c0eb4cb7058f3d0ee8870ee5d746214ab0589b5e841fd5d68fec8f00
```

证明集群主节点安装成功，这里要记得保存这条命令，以便之后各个节点加入集群

**配置kubetl认证信息**

```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
# 如果你想持久化的话，直接执行以下命令【推荐】
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bash_profile
```

**安装flanel网络**

```bash
mkdir -p /etc/cni/net.d/

cat <<EOF> /etc/cni/net.d/10-flannel.conf
{
"name": "cbr0",
"type": "flannel",
"delegate": {
"isDefaultGateway": true
}
}
EOF

mkdir /usr/share/oci-umount/oci-umount.d -p

mkdir /run/flannel/

cat <<EOF> /run/flannel/subnet.env
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.1.0/24
FLANNEL_MTU=1450
FLANNEL_IPMASQ=true
EOF
```

**最后需要新建一个flannel.yml文件：**

```yaml
image:
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
name: flannel
rules:
- apiGroups:
    - ""
    resources:
    - pods
    verbs:
    - get
- apiGroups:
    - ""
    resources:
    - nodes
    verbs:
    - list
    - watch
- apiGroups:
    - ""
    resources:
    - nodes/status
    verbs:
    - patch
image:
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
name: flannel
roleRef:
apiGroup: rbac.authorization.k8s.io
kind: ClusterRole
name: flannel
subjects:
- kind: ServiceAccount
name: flannel
namespace: kube-system
image:
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
apiVersion: v1
kind: ServiceAccount
metadata:
name: flannel
namespace: kube-system
image:
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
kind: ConfigMap
apiVersion: v1
metadata:
name: kube-flannel-cfg
namespace: kube-system
labels:
    tier: node
    app: flannel
data:
cni-conf.json: |
    {
    "name": "cbr0",
    "type": "flannel",
    "delegate": {
        "isDefaultGateway": true
    }
    }
net-conf.json: |
    {
    "Network": "10.10.0.0/16",    #这里换成集群所在的网段
    "Backend": {
        "Type": "vxlan"
    }
    }
image:
  caption: "Image from: [**Pexels**](https://www.pexels.com)"
  focal_point: ""
  preview_only: false
---
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
name: kube-flannel-ds
namespace: kube-system
labels:
    tier: node
    app: flannel
spec:
template:
    metadata:
    labels:
        tier: node
        app: flannel
    spec:
    hostNetwork: true
    nodeSelector:
        beta.kubernetes.io/arch: amd64
    tolerations:
    - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
    serviceAccountName: flannel
    initContainers:
    - name: install-cni
        image: quay.io/coreos/flannel:v0.9.1-amd64
        command:
        - cp
        args:
        - -f
        - /etc/kube-flannel/cni-conf.json
        - /etc/cni/net.d/10-flannel.conf
        volumeMounts:
        - name: cni
        mountPath: /etc/cni/net.d
        - name: flannel-cfg
        mountPath: /etc/kube-flannel/
    containers:
    - name: kube-flannel
        image: quay.io/coreos/flannel:v0.9.1-amd64
        command: [ "/opt/bin/flanneld", "--ip-masq", "--kube-subnet-mgr" ]
        securityContext:
        privileged: true
        env:
        - name: POD_NAME
        valueFrom:
            fieldRef:
            fieldPath: metadata.name
        - name: POD_NAMESPACE
        valueFrom:
            fieldRef:
            fieldPath: metadata.namespace
        volumeMounts:
        - name: run
        mountPath: /run
        - name: flannel-cfg
        mountPath: /etc/kube-flannel/
    volumes:
        - name: run
        hostPath:
            path: /run
        - name: cni
        hostPath:
            path: /etc/cni/net.d
        - name: flannel-cfg
        configMap:
            name: kube-flannel-cfg
```

**执行：**
    
```bash
kubectl create -f ./flannel.yml
```

默认情况下，master节点不参与工作负载，但如果希望安装出一个all-in-one的k8s环境，则可以执行以下命令：

**让master节点成为一个node节点：**

```bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```

**查看节点信息：**

```bash
kubectl get nodes
```

**会看到如下的输出：**

```bash
NAME            STATUS     ROLES     AGE       VERSION
k8s-master      Ready      master    18h       v1.11.0
```

**以下是节点配置**

在配置好主节点之后，就可以配置集群的其他节点了，这里建议直接安装之前做好准备工作的系统镜像
进入节点机器之后，直接执行之前保存好的命令

```bash
kubeadm join 10.10.207.253:6443 --token t69z6h.lr2etdbg9mfx5r15 --discovery-token-ca-cert-hash sha256:90e3a748c0eb4cb7058f3d0ee8870ee5d746214ab0589b5e841fd5d68fec8f00
```

执行完后会看到：

```go
[preflight] running pre-flight checks
        [WARNING RequiredIPVSKernelModulesAvailable]: the IPVS proxier will not be used, because the following required kernel modules are not loaded: [ip_vs_wrr ip_vs_sh ip_vs ip_vs_rr] or no builtin kernel ipvs support: map[ip_vs_rr:{} ip_vs_wrr:{} ip_vs_sh:{} nf_conntrack_ipv4:{} ip_vs:{}]
you can solve this problem with following methods:
1. Run 'modprobe -- ' to load missing kernel modules;
2. Provide the missing builtin kernel ipvs support

I0725 09:59:27.929247   10196 kernel_validator.go:81] Validating kernel version
I0725 09:59:27.929356   10196 kernel_validator.go:96] Validating kernel config
[discovery] Trying to connect to API Server "10.10.207.253:6443"
[discovery] Created cluster-info discovery client, requesting info from "https://10.10.207.253:6443"
[discovery] Requesting info from "https://10.10.207.253:6443" again to validate TLS against the pinned public key
[discovery] Cluster info signature and contents are valid and TLS certificate validates against pinned roots, will use API Server "10.10.207.253:6443"
[discovery] Successfully established connection with API Server "10.10.207.253:6443"
[kubelet] Downloading configuration for the kubelet from the "kubelet-config-1.11" ConfigMap in the kube-system namespace
[kubelet] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[preflight] Activating the kubelet service
[tlsbootstrap] Waiting for the kubelet to perform the TLS Bootstrap...
[patchnode] Uploading the CRI Socket information "/var/run/dockershim.sock" to the Node API object "k8s-node1" as an annotation

This node has joined the cluster:
* Certificate signing request was sent to master and a response
was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the master to see this node join the cluster.
```

**这里就表示执行完毕了，可以去主节点执行命令：**

```bash
kubectl get nodes
```

**可以看到节点已加入集群：**

```bash
NAME        STATUS    ROLES     AGE       VERSION
k8s-master  Ready     master    20h       v1.11.0
k8s-node1   Ready     <none>    20h       v1.11.0
k8s-node2   Ready     <none>    20h       v1.11.0
```

这期间可能需要等待一段时间，状态才会全部变为ready

# kubernetes-dashboard安装

详见：[kubernetes安装dashboard](/2018/dashboard-k8s)

# 采坑指南
有时会出现master节点一直处于notready的状态，这里可能是没有启动flannel，只需要按照上面的教程配置好flannel，然后执行：

```bash
kubectl create -f ./flannel.yml
```