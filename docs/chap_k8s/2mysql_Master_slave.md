# **2 在K8s中部署主从结构的MySQL服务**

## 01 介绍

RC、Deployment、DaemonSet都是面向无状态的服务，它们所管理的Pod的IP、名字、启停顺序等都是随机分配的，而StatefulSet，管理所有有状态的服务。

StatefulSet为了解决有状态服务的问题，它所管理的Pod拥有固定的Pod名称，一定的启停顺序，在StatefulSet中，Pod名字称为网络标识(hostname)，还必须要用到共享存储。

在Deployment中，与之对应的服务是service，而在StatefulSet中与之对应的headless service。

**headless service，即无头服务，与service的区别就是它没有Cluster IP，解析它的名称时将返回该Headless Service对应的全部Pod的节点列表**。

**除此之外，StatefulSet在Headless Service的基础上又为StatefulSet控制的每个Pod副本创建了一个DNS域名，这个域名的格式为**：

```
(podname).(headless server name).namespace.svc.cluster.local
```

## 02 部署mysql

MySQL 示例部署包含一个ConfigMap、两个存储挂载pv和pvc、两个 Service 与一个 StatefulSet。

### **创建一个ConfigMap**

使用以下的 YAML 配置文件创建 ConfigMap ：

**`mysql-master-cnf.yaml`**

```
#master--my.cnf
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-master-cnf
  namespace: bc-cnp
data:
  my.cnf: |-
    [client]
    default-character-set=utf8
    [mysql]
    default-character-set=utf8
    [mysqld]
    init_connect='SET collation_connection = utf8_unicode_ci'
    init_connect='SET NAMES utf8'
    character-set-server=utf8
    collation-server=utf8_unicode_ci
    skip-character-set-client-handshake
    skip-name-resolve
    server_id=1
    log-bin=mysql-bin
    read-only=1
    replicate-ignore-db=mysql
    replicate-ignore-db=sys
    replicate-ignore-db=information_schema
    replicate-ignore-db=performance_schema
```

**`mysql-slave-cnf.yaml`**

```
#slave--my.cnf
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-slave-cnf
  namespace: bc-cnp
data:
  my.cnf: |-
    [client]
    default-character-set=utf8
    [mysql]
    default-character-set=utf8
    [mysqld]
    init_connect='SET collation_connection = utf8_unicode_ci'
    init_connect='SET NAMES utf8'
    character-set-server=utf8
    collation-server=utf8_unicode_ci
    skip-character-set-client-handshake
    skip-name-resolve
    server_id=2
    log-bin=mysql-bin
    read-only=0
    replicate-ignore-db=mysql
    replicate-ignore-db=sys
    replicate-ignore-db=information_schema
    replicate-ignore-db=performance_schema
```

```
$ kubectl create ns bc-cnp

$ kubectl apply -f mysql-master-cnf.yaml

$ kubectl apply -f mysql-slave-cnf.yaml
```

这个 ConfigMap 提供 `my.cnf` 覆盖设置，可以独立控制 MySQL 主服务器配置。

ConfigMap 本身没有什么特别之处，因而也不会出现不同部分应用于不同的 Pod 的情况。

每个 Pod 都会在初始化时基于 StatefulSet 控制器提供的信息决定要查看的部分。

slave从服务器配置和主服务器配置基本相同，需要修改


* `metadata.name= mysql-slave-cnf`
* `server_id=2`
* `read-only=0`

获取`mysql-master-0`和`mysql-slave-0`的ConfigMap ：

```
kubectl get cm -nbc-cnp

NAME               DATA   AGE
kube-root-ca.crt   1      2m24s
mysql-master-cnf   1      115s
mysql-slave-cnf    1      55s
```

### **创建一个MYSQL ROOT Secret**

`mysql-secret.yaml`

```
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
  namespace: bc-cnp
  labels:
    app: mysql
type: Opaque
data:
  MYSQL_ROOT_PASSWORD: MTIzNDU2 # echo -n "123456" | base64
```

```
$ kubectl apply -f mysql-secret.yaml
```

### **创建pv和pvc，写入稳定存储**

使用以下 YAML 配置文件创建服务：

**`master-pvc-pv.yaml`**

```
---
#master--pv
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv-master
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  local:
    path: /Users/i515190/k8s_test/mysql
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - docker-desktop 
---
#master--pvc
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc-master
  namespace: bc-cnp
spec:
  accessModes:
    - ReadWriteOnce
  # storageClassName: hostpath
  resources:
    requests:
      storage: 1Gi
  # volumeName: mysql-pv-master
```

**`slave-pvc-pv.yaml`**

```
---
#slave--pv
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv-master
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 1Gi
  local:
    path: /Users/i515190/k8s_test/mysql
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - docker-desktop 
---
#master--pvc
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc-master
  namespace: bc-cnp
spec:
  accessModes:
    - ReadWriteOnce
  # storageClassName: hostpath
  resources:
    requests:
      storage: 1Gi
  # volumeName: mysql-pv-master
```

slave采用同样的方式创建pv及pvc，只名字修改为slave即可。

**获取master和slave的PersistentVolumeClaims**：


```
$ kubectl apply -f master-pvc-pv.yaml 

$ kubectl apply -f slave-pvc-pv.yaml 

kubectl get pvc -nbc-cnp
```

为StatefulSet 控制器创建了两个 PersistentVolumeClaims， 绑定到两个 PersistentVolumes。

Mysql 服务器默认会加载位于 `/var/lib/mysql-files` 的文件。

**StatefulSet spec 中的 volumeMounts 字段保证了`/var/lib/mysql-files` 文件夹由一个 PersistentVolume 卷支持**。

### 创建 Service

使用以下 YAML 配置文件创建服务：

**`mysql-master-services.yaml`**

```
#master--headless service
apiVersion: v1
kind: Service
metadata:
  namespace: bc-cnp
  labels:
    app: mysql-master
  annotations:
    kubesphere.io/serviceType: statefulservice
    kubesphere.io/alias-name: mysql-master
  name: mysql-master
spec:
  sessionAffinity: ClientIP
  selector:
    app: mysql-master
  ports:
    - name: tcp-3306
      protocol: TCP
      port: 3306
      targetPort: 3306
    - name: tcp-33060
      protocol: TCP
      port: 33060
      targetPort: 33060
  clusterIP: None
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800

---
#master--nodePort service
apiVersion: v1
kind: Service
metadata:
  name: mysql-master-front
  labels:
    app: mysql-master
  namespace: bc-cnp
spec:
  selector:
    app: mysql-master
  type: NodePort
  ports:
    - name: ''
      port: 3306
      protocol: TCP
      targetPort: 3306
      nodePort: 30001  
  sessionAffinity: None
```

**`mysql-slave-services.yaml`**

```
#master--headless service
apiVersion: v1
kind: Service
metadata:
  namespace: bc-cnp
  labels:
    app: mysql-slave
  annotations:
    kubesphere.io/serviceType: statefulservice
    kubesphere.io/alias-name: mysql-slave
  name: mysql-slave
spec:
  sessionAffinity: ClientIP
  selector:
    app: mysql-slave
  ports:
    - name: tcp-3306
      protocol: TCP
      port: 3306
      targetPort: 3306
    - name: tcp-33060
      protocol: TCP
      port: 33060
      targetPort: 33060
  clusterIP: None
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800

---
#master--nodePort service
apiVersion: v1
kind: Service
metadata:
  name: mysql-slave-front
  labels:
    app: mysql-slave
  namespace: bc-cnp
spec:
  selector:
    app: mysql-slave
  type: NodePort
  ports:
    - name: ''
      port: 3306
      protocol: TCP
      targetPort: 3306
      nodePort: 30002  
  sessionAffinity: None
```

执行YAML：

slave同样执行类似yaml，名字需要修改为mysql-slave相关，暴露nodeport改为：30002。

输出类似于：

```
$ kubectl apply -f mysql-master-services.yaml 

$ kubectl apply -f mysql-slave-services.yaml 

$ kubectl get svc -n bc-cnp 
NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)              AGE
mysql-master         ClusterIP   None             <none>        3306/TCP,33060/TCP   69s
mysql-master-front   NodePort    10.101.3.70      <none>        3306:30001/TCP       69s
mysql-slave          ClusterIP   None             <none>        3306/TCP,33060/TCP   62s
mysql-slave-front    NodePort    10.107.251.175   <none>        3306:30002/TCP       62s
```


**headless service 的 CNAME 指向 SRV 记录（记录每个 Running 和 Ready 状态的 Pod）。SRV 记录指向一个包含 Pod IP 地址的记录表项。**

headless service给 StatefulSet 控制器 为集合中每个 Pod 创建的 DNS 条目提供了一个宿主。


因为无头服务名为 mysql-master，所以可以通过在同一 Kubernetes 集群和命名空间中的任何其他 Pod 内解析 `<Pod 名称>.mysql-master` 来访问 Pod。

mysql-master-front是一种常规 Service，具有其自己的集群 IP。该集群 IP 在报告就绪的所有 MySQL Pod 之间分配连接。可能的端点集合包括 MySQL 主节点和从节点。

请注意，只有读查询才能使用负载平衡的客户端 Service。因为只有一个 MySQL 主服务器，所以客户端应直接连接到 MySQL 主服务器 Pod （通过其在headless service 中的 DNS 条目）以执行写入操作


### **创建 StatefulSet**

最后，使用以下 YAML 配置文件创建 StatefulSet：

**`mysql-master-statefulset.yaml`**

```
#master--statefulset
apiVersion: apps/v1
kind: StatefulSet
metadata:
  namespace: bc-cnp
  labels:
    app: mysql-master
  name: mysql-master
  annotations:
    kubesphere.io/alias-name: mysql-master
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql-master
  template:
    metadata:
      labels:
        app: mysql-master
    spec:
      containers:
        - name: master-container
         # type: worker
          imagePullPolicy: IfNotPresent
          resources:
            requests:
              cpu: '1'
              memory: 1Gi
            limits:
              cpu: '1'
              memory: 1Gi
          image: mysql:8.0.18
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_ROOT_PASSWORD
          volumeMounts:
            - name: master-cnf-volume
              readOnly: false
              mountPath: /etc/mysql
            - name: master-data-volume
              readOnly: false
              mountPath: /var/lib/mysql-files
      serviceAccount: default
      # affinity:
      #   podAntiAffinity:
      #     preferredDuringSchedulingIgnoredDuringExecution:
      #       - weight: 100
      #         podAffinityTerm:
      #           labelSelector:
      #             matchLabels:
      #               app: mysql-master
      #           topologyKey: kubernetes.io/hostname
      volumes:
        - name: master-cnf-volume
          configMap:
            name: mysql-master-cnf
            items:
              - key: my.cnf
                path: my.cnf
        - name: master-data-volume
          persistentVolumeClaim:
            claimName: mysql-pvc-master
  serviceName: mysql-master
```

**`mysql-slave-statefulset.yaml`**

```
#slave--statefulset
apiVersion: apps/v1
kind: StatefulSet
metadata:
  namespace: bc-cnp
  labels:
    app: mysql-slave
  name: mysql-slave
  annotations:
    kubesphere.io/alias-name: mysql slave
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql-slave
  template:
    metadata:
      labels:
        app: mysql-slave
    spec:
      containers:
        - name: slave-container
         # type: worker
          imagePullPolicy: IfNotPresent
          resources:
            requests:
              cpu: '1'
              memory: 1Gi
            limits:
              cpu: '1'
              memory: 1Gi
          image: mysql:8.0.18
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: MYSQL_ROOT_PASSWORD
          volumeMounts:
            - name: slave-cnf-volume
              readOnly: false
              mountPath: /etc/mysql
            - name: slave-data-volume
              readOnly: false
              mountPath: /var/lib/mysql-files
      serviceAccount: default
      # affinity:
      #   podAntiAffinity:
      #     preferredDuringSchedulingIgnoredDuringExecution:
      #       - weight: 100
      #         podAffinityTerm:
      #           labelSelector:
      #             matchLabels:
      #               app: mysql-slave
      #           topologyKey: kubernetes.io/hostname
      volumes:
        - name: slave-cnf-volume
          configMap:
            name: mysql-slave-cnf
            items:
              - key: my.cnf
                path: my.cnf
        - name: slave-data-volume
          persistentVolumeClaim:
            claimName: mysql-pvc-slave
  serviceName: mysql-slave
```

通过运行以下命令查看启动进度：

可以看到所有 2 个 Pod 进入 Running 状态：

```
$ kubectl apply -f mysql-master-statefulset.yaml
statefulset.apps/mysql-master created
$ kubectl apply -f mysql-slave-statefulset.yaml 
statefulset.apps/mysql-slave created


$ kubectl get pods -n bc-cnp 
NAME             READY   STATUS    RESTARTS   AGE
mysql-master-0   1/1     Running   0          67s
mysql-slave-0    1/1     Running   0          27s
```

StatefulSet 中的每个 Pod 拥有一个唯一的顺序索引和稳定的网络身份标识。

这个标志基于 StatefulSet 控制器分配给每个 Pod 的唯一顺序索引。Pod 名称的格式为 `<statefulset 名称>-<序号索引>`。

## 03 主从同步

进入mysql-master容器内部

```
$ kubectl exec -it mysql-master-0 sh -n bc-cnp 
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.

# 1.进入mysql内部
$ kubectl exec -it mysql-master-0 sh -n bc-cnp 
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
# mysql -uroot -p123456

#切换到 mysql DB
mysql> USE mysql;   

Database changed

# 查看root用户是否具备远程访问权限
mysql> select Host,User,authentication_string,password_expired,password_last_changed from user;
+-----------+------------------+------------------------------------------------------------------------+------------------+-----------------------+
| Host      | User             | authentication_string                                                  | password_expired | password_last_changed |
+-----------+------------------+------------------------------------------------------------------------+------------------+-----------------------+
| %         | root             | $A$005$|`K{0M=_,;/x'-cRV#kTHWatUJJ7.ERa9WgkADdVtfvb4OzWkfa1qKjMwl/f1 | N                | 2023-03-12 06:15:14   |
| localhost | mysql.infoschema | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED | N                | 2023-03-12 06:15:08   |
| localhost | mysql.session    | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED | N                | 2023-03-12 06:15:08   |
| localhost | mysql.sys        | $A$005$THISISACOMBINATIONOFINVALIDSALTANDPASSWORDTHATMUSTNEVERBRBEUSED | N                | 2023-03-12 06:15:08   |
| localhost | root             | $A$005$
                                        gb.,#;t)4ZPy+!IKDjAJA9Yh1nOzdQ5mRnBRy5srJq.M6iZ0KNxGaX9P1 | N                | 2023-03-12 06:15:14   |
+-----------+------------------+------------------------------------------------------------------------+------------------+-----------------------+
5 rows in set (0.00 sec)

# 2.授权 root可以远程访问（主从无关，如root没有访问权限，执行以下命令，方便我们远程连接MySQL）

mysql> grant all privileges on *.* to 'root'@'%';
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

# 3.添加用来同步的用户
CREATE USER 'repl'@'%' IDENTIFIED BY 'Passw0rd';
GRANT REPLICATION SLAVE ON *.* to 'repl'@'%';

# 4.查看master状态
mysql> show master status\G;
*************************** 1. row ***************************
             File: mysql-bin.000003
         Position: 1095
     Binlog_Do_DB: 
 Binlog_Ignore_DB: 
Executed_Gtid_Set: 
1 row in set (0.00 sec)
```

### 然后进入到mysql-slave内部

```
$ kubectl exec -it mysql-slave-0 sh -n bc-cnp 
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.

# mysql -uroot -p123456     
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.18 MySQL Community Server - GPL

mysql> 

change master to master_host='mysql-master.bc-cnp.svc.cluster.local',master_user='repl',master_password='Passw0rd',master_log_file='mysql_bin.000003',master_log_pos=0;

# 启动从库同步
start slave;
# 查看从从库状态
show slave status\G;
```

`mysql -h mysql-master.bc-cnp.svc.cluster.local  -urepl -pPassw0rd`


只有当以下两项都是yes，才意味着同步成功。

```
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: mysql-master.bc-cnp.svc.cluster.local
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000003
          Read_Master_Log_Pos: 1095
               Relay_Log_File: mysql-slave-0-relay-bin.000005
                Relay_Log_Pos: 1309
        Relay_Master_Log_File: mysql-bin.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: mysql,sys,information_schema,performance_schema
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
```


如果同步不成功，尝试执行以下命令，再次查看。

```
top slave;
reset slave;
start slave;
```

## 04 结果验证

最终验证结果，在master mysql创建一个database及table，插入数据，在slave中依然能够查看相关数据，确认主从同步成功

进入主的mysql-master容器内部，执行创建操作，如下：

```
$ kubectl exec -it mysql-master-0 sh -n bc-cnp 
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
# mysql -uroot -p123456   

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)

create database repl_test;
use repl_test;

create table student(Sno int, name varchar(10));
insert into student(Sno, name) values (20230312, "ubsjx");
show tables;

select * from student;

mysql> create database repl_test;
Query OK, 1 row affected (0.01 sec)

mysql> use repl_test;
Database changed
mysql> create table student(Sno int, name varchar(10));
Query OK, 0 rows affected (0.02 sec)

mysql> insert into student(Sno, name) values (20230312, "ubsjx");
Query OK, 1 row affected (0.01 sec)

mysql> show tables;
+---------------------+
| Tables_in_repl_test |
+---------------------+
| student             |
+---------------------+
1 row in set (0.00 sec)

mysql> select * from student;
+----------+-------+
| Sno      | name  |
+----------+-------+
| 20230312 | ubsjx |
+----------+-------+
1 row in set (0.01 sec)
```

进入从的mysql-slave容器内部，执行创建操作，如下：

```
$ kubectl exec -it mysql-slave-0 sh -n bc-cnp 
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
# mysql -uroot -p123456   

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| repl_test          |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> use repl_test;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+---------------------+
| Tables_in_repl_test |
+---------------------+
| student             |
+---------------------+
1 row in set (0.01 sec)


mysql> select * from student;
+----------+-------+
| Sno      | name  |
+----------+-------+
| 20230312 | ubsjx |
+----------+-------+
1 row in set (0.00 sec)
```

至此，确认从库中已有主库中数据，同步成功。


## 05 总结

StatefulSet 中每个 Pod 都拥有一个基于其顺序索引的稳定的主机名，如果pod发生故障，StatefulSet 重启他们，Pod 的序号、主机名、SRV 条目和记录名称没有改变，但和 Pod 相关联的 IP 地址可能发生了改变。这就是为什么不要在其他应用中使用 StatefulSet 中 Pod 的 IP 地址进行连接，这点很重要。

StatefulSet 的活动成员是和 CNAME 相关联的 SRV 记录，其中只会包含 StatefulSet 中处于 Running 和 Ready 状态的 Pod。

如果我们的应用已经实现了用于测试是否已存活（liveness）并就绪（readiness）的连接逻辑，我们可以使用 Pod 的 SRV 记录（mysql-master.default.svc.cluster.local）。并且当里面的 Pod 的状态变为 Running 和 Ready 时，我们的应用就能够发现各个pod的地址。

虽然 mysql-master-0 和 mysql-slave-0 有时候会被重新调度，但它们仍然继续监听各自的主机名，因为和它们的 PersistentVolumeClaim 相关联的 PersistentVolume 卷被重新挂载到了各自的 volumeMount 上。不管mysql-master-0 和 mysql-slave-0 被调度到了哪个节点上，它们的 PersistentVolume 卷将会被挂载到合适的挂载点上。

至此，以上分享只是其中的一种主从方式容器化部署，还有其他的方案，后面也会陆续的分享，希望大家持续的关注。


