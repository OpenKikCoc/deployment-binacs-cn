# deployment-binacs-cn
Kubernetes deployment files.





## 1. 基本划分



|              任务              |        主机         |      service      |      deployment      | namespace |     备注      |
| :----------------------------: | :-----------------: | :---------------: | :------------------: | :-------: | :-----------: |
|           binacs.cn            |  binacs-qcloud-01   | binacs-cn-service | binacs-cn-deployment | binacs-cn | node selector |
| prometheus/pushgateway/grafana | binacs-huaweiyun-03 |  monitor-service  |  monitor-deployment  | binacs-cn | node selector |
|            jenkins             |  binacs-qcloud-06   |  jenkins-service  |   jenkins-service    | binacs-cn | node selector |
|            gitbook             |          -          |  gitbook-service  |                      | binacs-cn |      amd      |
|          crypto-func           |          -          |         -         |          -           |     -     |       -       |



以及 **Ingress-Nginx** 同样部署在 qcloud-01



## 2. 实现

### 2.1 NodeSelector

```shell
# 显示标签
kubectl get nodes --show-labels

# 给指定node添加label
kubectl label node binacs-qcloud-01 labelName=master
kubectl label node binacs-qcloud-06 labelName=jenkins
kubectl label node binacs-huaweiyun-03 labelName=monitor
```





### 2.2 service & deployment 

见文件内容



```
sudo chown -R 1000:1000 /var/jenkins_home
```

等等的文件目录权限



### 2.3 ingress

防火墙:

```shell
iptables -P INPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -F
iptables -L -n
```


创建名为 grpcs-secret 的 secret :

```shell
$ kubectl create secret tls grpcs-secret --key binacs.cn.key --cert binacs.cn.crt
```

创建 secret （指定 namespace） :

```shell
kubectl create secret -n <your namespace> tls grpcs-secret --key binacs.cn.key --cert binacs.cn.crt
kubectl create secret -n binacs-cn tls grpcs-secret --key binacs.cn.key --cert binacs.cn.crt

#
kubectl create secret -n binacs-cn tls grpcs-secret-prometheus --key prometheus.binacs.cn.key --cert prometheus.binacs.cn.crt

kubectl create secret -n binacs-cn tls grpcs-secret-pushgateway --key pushgateway.binacs.cn.key --cert pushgateway.binacs.cn.crt

kubectl create secret -n binacs-cn tls grpcs-secret-grafana --key grafana.binacs.cn.key --cert grafana.binacs.cn.crt

kubectl create secret -n binacs-cn tls grpcs-secret-jenkins --key jenkins.binacs.cn.key --cert jenkins.binacs.cn.crt

kubectl create secret -n binacs-cn tls grpcs-secret-gitbook --key gitbook.binacs.cn.key --cert gitbook.binacs.cn.crt

kubectl create secret -n binacs-cn tls grpcs-secret-docs --key docs.binacs.cn.key --cert docs.binacs.cn.crt

kubectl create secret -n binacs-cn tls grpcs-secret-kiki --key kiki.binacs.cn.key --cert kiki.binacs.cn.crt

```


在 `ingress.yml` 中需设置该 tls 字段



## 3. 顺序

1.  namespace
2.  binacs-cn
3.  monitor
4.  jenkins
5.  gitbook
6.  docs
7.  ingress




```shell
kubectl create -f namespaces.yml
kubectl create -f binacs-cn.yml
kubectl create -f monitor.yml
kubectl create -f jenkins.yml
kubectl create -f ===========
kubectl create -f ingress-deploy.yml
kubectl create -f ingress-rule.yml
```




