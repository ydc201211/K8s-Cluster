# K8S 所有服务器公共环境搭建

---

本章我们来讨论如何在CentOS 7上安装Docker。
> **[warning] 提醒**

Docker必须安装在CentOS7 64位机器上。如果您的系统是CentOS 6.x，请升级；如果您的机器是32位系统，请更换。<br>


## 实验环境说明
```
master: master 192.168.20.10
slave1: node 192.168.20.11
slave2: node 192.168.20.12
```


## 实验环境说明
cat >>/etc/hosts<<EOF
192.168.20.10 master 
192.168.20.11 slave1
192.168.20.12 slave2
EOF

## 卸载老版本Docker
> 对于没有安装过docker的可以忽略本步骤。

如果之前有安装过docker或者k8s需要进行以下清理操作（没有的可跳过操作）。相关命令如下：

执行以下命令即可：
```
yum -y remove docker docker-common docker-selinux docker-engine

yum -y remove flannel

yum  -y remove etcd

yum -y remove  kube*

需要注意的是，执行该命令只会卸载Docker本身，而不会删除Docker内容，例如镜像、容器、卷以及网络。这些文件保存在/var/lib/docker 目录中，需要手动删除。

如：rm -rf /var/lib/etcd

rm -rf  /var/lib/docker等
```



## 设置宿主机环境
>对于公司环境，有可能生产服务或者本地不能连接网络的情况。我们就需要通过安装包的方式安装。

首先关闭防火墙，
```
systemctl stop firewalld && systemctl disable firewalld。
```

修改网桥，配置系统路由参数,防止kubeadm报路由警告


```
echo "
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
" >> /etc/sysctl.conf
sysctl -p
```


```
echo "vm.swappiness=0" >> /etc/sysctl.conf
sysctl -p
```
关闭虚拟内存


```
swapoff -a
vi /etc/fstab
```
>注释掉 SWAP 的自动挂载，使用free -m确认swap列全部为0




## SSH 访问配置



为了让主节点能够免密码访问各个子节点，需要做以下配置：
```
# 在主节点执行该指令，会创建 /home/root/.ssh 文件夹
# 并生成私钥 id_rsa、公钥 id_rsa.pub
ssh-keygen -t rsa -P ""

# 将上面生成的公钥文件 id_rds.pub 复制到子节点
# 然后将公钥文件中的内容添加到文件
.ssh/authorized_keys 中

scp /root/.ssh/id_rsa.pub root@192.168.255.160:/root/.ssh/id_rsa.pub
cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys
chmod 600 .ssh/authorized_keys
```


## 安装docker
请自行安装docker，这里不提供docker安装包了


## 安装k8s组件
将该项目中所有k8s组件的rpm文件安装
```
执行yum localinstall -y *.rpm
```


## 导入镜像

将该项目中的所有镜像导入
```
docker load < busybox.tar
docker load < etcd-amd64_3.1.11.tar
docker load < flannel-0.9.1-amd64.tar
docker load < k8s-dns-dnsmasq-nanny-amd64_v1.14.7.tar
docker load < k8s-dns-kube-dns-amd64_1.14.7.tar
docker load < k8s-dns-sidecar-amd64_1.14.7.tar
docker load < kube-apiserver-amd64_v1.9.6.tar
docker load < kube-controller-manager-amd64_v1.9.6.tar
docker load < kube-proxy-amd64_v1.9.6.tar
docker load < kubernetes-dashboard-amd64_v1.8.3.tar
docker load < kube-scheduler-amd64_v1.9.6.tar
docker load < pause-amd64_3.0.tar
```