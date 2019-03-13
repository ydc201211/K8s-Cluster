kubeadm join --token 53d369.1be9e74bf81c3694 xxx.xxx.xxx.xxx:6443 --discovery-token-ca-cert-hash sha256:72717cb30e54363a5fe27bbe45a560f40882660c60bd46de43f3c1ff4ce68013

- 上面是在kubeadm初始化后的到指令，在node节点执行即可

部署dashboard
- 使用下面的命令安装
```
kubectl apply -f kube-dashboard.yaml
```

设置监听端口为8001
```
kubectl proxy --address=0.0.0.0 --port=8001 --accept-hosts='^.*'
```

通过下面的连接访问kube-dashboard
- http://xxx.xxx.xxx.xxx:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/node?namespace=kube-system