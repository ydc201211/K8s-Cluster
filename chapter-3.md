# K8S Master环境搭建

---



## 配置kubete：
```
修改kubelet的配置  ，vi  /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_SYSTEM_PODS_ARGS=--pod-manifest-path=/etc/kubernetes/manifests --allow-privileged=true"
Environment="KUBELET_NETWORK_ARGS=--network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin"
Environment="KUBELET_DNS_ARGS=--cluster-dns=10.97.0.10 --cluster-domain=cluster.local"
Environment="KUBELET_AUTHZ_ARGS=--authorization-mode=Webhook --client-ca-file=/etc/kubernetes/pki/ca.crt"
Environment="KUBELET_CADVISOR_ARGS=--cadvisor-port=4194"
Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=cgroupfs"
Environment="KUBELET_SWAP_ARGS=--fail-swap-on=false"
Environment="KUBELET_INFRA_IMAGE=--pod-infra-container-image=10.5.88.69:9000/pause-amd64:3.0"
Environment="KUBELET_CERTIFICATE_ARGS=--rotate-certificates=true --cert-dir=/var/lib/kubelet/pki"
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_SWAP_ARGS $KUBELET_INFRA_IMAGE $KUBELET_KUBECONFIG_ARGS $KUBELET_SYSTEM_PODS_ARGS $KUBELET_NETWORK_ARGS $KUBELET_DNS_ARGS $KUBELET_AUTHZ_ARGS $KUBELET_CADVISOR_ARGS $KUBELET_CGROUP_ARGS $KUBELET_CERTIFICATE_ARGS $KUBELET_EXTRA_ARGS
```
配置说明：
```
  --cluster-dns=10.97.0.10  这个是配置k8s的kube-dns的 cluster-ip，这个值必须和后面 kubeadm init中的  --service-cidr的网段一致

 --cgroup-driver=cgroupfs  这个值需要和docker的 存储驱动一致，启动docker 之后 可以通过docker info指令查看，修改一致记性

 --pod-infra-container-image=10.5.88.69:9000/pause-amd64:3.0  这个配置指定了pod的基础容器镜像地址， 这个基础镜像的作用是 让pod中的 网络、进程命名空间都保持一致。

 --fail-swap-on=false 关闭pod容器中的swap相关功能，不关闭kubelet无法启动
```

## 在主节点kubeadm进行初始化
```
    kubeadm init --kubernetes-version=v1.9.6 --pod-network-cidr=10.244.0.0/16   --service-cidr=10.97.0.0/16    --token-ttl=0
```

    如果正常的话，最后会显示如下内容，
```
       kubeadm join --token 53d369.1be9e74bf81c3694 xxx.xxx.xxx.xxx:6443 --discovery-token-ca-cert-hash sha256:72717cb30e54363a5fe27bbe45a560f40882660c60bd46de43f3c1ff4ce68013
```
xxx.xxx.xxx.xxx为master的IP



## 修改环境变量
```
vi /root/.bash_profile

export KUBECONFIG=/etc/kubernetes/admin.conf

source /root/.bash_profile
```


## 创建flannel网络
```
   kubectl create -f kube-flannel-rbac.yml
   kubectl apply -f kube-flannel.yml
```
   成功之后检查相关pod状态
```
kubectl get pods --all-namespaces
NAMESPACE NAME READY STATUS RESTARTS AGE
kube-system etcd-k8s-master70 1/1 Running 0 17m
kube-system kube-apiserver-k8s-master70 1/1 Running 0 17m
kube-system kube-controller-manager-k8s-master70 1/1 Running 0 18m
kube-system kube-dns-6f4fd4bdf-wr5j9 3/3 Running 0 18m
kube-system kube-flannel-ds-bp6g7 1/1 Running 0 15m
kube-system kube-proxy-gtkmz 1/1 Running 0 18m
kube-system kube-scheduler-k8s-master70 1/1 Running 0 17m
```
如果为此状态就说明成功了
