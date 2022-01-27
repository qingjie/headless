kubernetes的Headless Services
1.什么是Headless Services
Headless Services是一种特殊的service，其spec:clusterIP表示为None，这样在实际运行时就不会被分配ClusterIP。

2.Headless Services使用场景（客户端负载）
服务端负载：正常的service 下面挂的是Endpoints（podIP数组），通过iptables规则转发到实际的POD上
客户端负载：Headless Services不会分配ClusterIP,而是将Endpoints（即podIP数组）返回，也就将服务端的所有节点地址返回，让客户端自行要通过负载策略完成负载均衡。

3.实践
```
#nginx yaml
[root@node1 yaml]#  cat  nginx.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx-demo
  ports:
  - port: 80
    name: nginx
  clusterIP: None
---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: nginx-dp
spec:
  serviceName: "nginx-service"
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx-demo
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
          name: web
[root@node1 ~]# kubectl get svc
NAME            TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
nginx-service   ClusterIP   None         <none>        443/TCP   2d10h
[root@node1 ~]# kubectl describe  service nginx-service
Name:                   nginx-service
Namespace:              default
Labels:                 <none>
Selector:               app=nginx-demo
Type:                   ClusterIP
IP:                     None
Port:                   web     80/TCP
Endpoints:              10.235.1.88:80,10.235.1.89:80
Session Affinity:       None
No events.
[root@node1 ~]# nslookup nginx-demo.default.svc.cluster.local 10.235.1.15
Server:         10.235.1.15
Address:        10.235.1.15#53

Name:   nginx-demo.default.svc.cluster.local
Address: 10.235.1.88
Name:   nginx-demo.default.svc.cluster.local
Address: 10.235.1.89
```
