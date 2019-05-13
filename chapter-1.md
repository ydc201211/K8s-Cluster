#### 虚拟机Master与slave配置
- 在搭建环境之前需要修改主节点Master和子节点slave的网络配置，步骤如下：

- 修改Master节点和slave节点的IP地址，并且设置虚拟机网络配置为桥接模式

- 打开网络配置文件，设置好ip地址、网关，dns切记与pc机保持一致
``` 
 vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
```
- 修改hosts文件，将master与slave的ip写入
``` 
 vi /etc/hosts
```
- 最后再关闭防火墙,保持关闭且不开机启动
```
systemctl stop firewalld && systemctl disable firewalld
```
- OK，这样两个节点就能够互相访问了，而且使用桥接模式还可以通过外部的主机xshell连接两个虚拟机节点，方便操作。
