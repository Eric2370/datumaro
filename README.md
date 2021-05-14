# Datumaro Feature List

* 本文件用于罗列datumaro commandline直接可用的工具
* 同时罗列出不可以用但是后续可能需要用到的工具

## 使用须知
* datumaro gihub: https://github.com/openvinotoolkit/datumaro
* 现datumaro工具在command line直接下载运行会有bug
* 于是我们就拉其代码到本地，并且使用用python 3.7.6以下的版本运行datumaro

## Datumaro feature 介绍

### 1. 数据集(dataset)转换

* convert PASCAL VOC dataset to COCO format
- Convert PASCAL VOC dataset to COCO format, keep only images with `cat` class presented:
  ```bash
  # Download VOC dataset:
  # http://host.robots.ox.ac.uk/pascal/VOC/voc2012/VOCtrainval_11-May-2012.tar
  datum convert --input-format voc --input-path <path/to/voc> \
                --output-format coco \
                --filter '/item[annotation/label="cat"]' \
                -- --reindex 1 # avoid annotation id conflicts
  ```

### 2. 准备k8s基础组件docker镜像

* kubeadm默认会从gcr(google云镜像仓库)拉取镜像，gcr国内网络无法访问
* 从阿里云镜像仓库拉取镜像到本地

#### 2.1 获取镜像列表

```sh
kubeadm config images list

# 结果样例
k8s.gcr.io/kube-apiserver:v1.19.9
k8s.gcr.io/kube-controller-manager:v1.19.9
k8s.gcr.io/kube-scheduler:v1.19.9
k8s.gcr.io/kube-proxy:v1.19.9
k8s.gcr.io/pause:3.2
k8s.gcr.io/etcd:3.4.13-0
k8s.gcr.io/coredns:1.7.0
```

#### 2.2 拉取镜像并重命名

* 可使用`get_k8s_imgs.sh`, 提前修改其中images列表

```sh
# get_k8s_imgs.sh

# 可根据上一步实际结果替换镜像
images=(  # 下面的镜像应该去除"k8s.gcr.io/"的前缀，版本换成上面获取到的版本
    kube-apiserver:v1.19.9
    kube-controller-manager:v1.19.9
    kube-scheduler:v1.19.9
    kube-proxy:v1.19.9
    pause:3.2
    etcd:3.4.13-0
    coredns:1.7.0
)

for imageName in ${images[@]} ; do
    docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
    docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName
    docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
done
```

### 3. 使用kubeadm初始化集群

* 使用calico，需要指定一个专属内网地址，用于k8s集群内pod之间通信
* 该内网地址网段，不要与宿主机重复

```sh
# 例如本地内网地址段为192.168.0.0/16, 可写如下网段，避免重复
sudo kubeadm init --pod-network-cidr=10.31.3.0/24
```

* 该过程可能会执行几分钟，结束后进行下一步

### 4. 设置kubectl客户端

* 将新创建的集群配置，copy到当前用户`~/.kube目录下`

```sh
mkdir ~/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

* 完成后，可执行`kubectl get all --all-namespaces`, 看输出是否正确，没有报错则初始化成功

### 5. 安装calico

* 上一步创建的k8s集群，可以基本使用，但集群内部pod无法通信
* 需要安装cni进行网络通信
* 此处选用calico，参照calico官网安装步骤

```sh
# Install the Tigera Calico operator and custom resource definitions.
#kubectl create -f https://docs.projectcalico.org/manifests/tigera-operator.yaml
kubectl create -f tigera-operator.yaml

# Install Calico by creating the necessary custom resource.
#kubectl create -f https://docs.projectcalico.org/manifests/custom-resources.yaml
kubectl create -f custom-resources.yaml

# Confirm that all of the pods are running with the following command
watch kubectl get pods -n calico-system # Wait until each pod has the STATUS of Running.

# Remove the taints on the master so that you can schedule pods on it.
kubectl taint nodes --all node-role.kubernetes.io/master-

# Confirm that you now have a node in your cluster with the following command.
kubectl get nodes -o wide
```

* 节点状态显示正常后，即代表calico安装成功

### 6. 简单调试k8s

* 可使用busybox/dnstools等镜像进行简单调试，确保网络正常

```sh
# 起busybox pod
kubectl run -it --rm --restart=Never busybox-debug --image=busybox sh
# 起dnstools pod
kubectl run -it --rm --restart=Never --image=infoblox/dnstools:latest dnstools
```
