# Update ClickHouse Installation - add replication to existing non-replicated cluster 

## Prerequisites
  1. Assume we have `clickhouse-operator` already installed and running
  1. Assume we have `Zookeeper` already installed and running
  
## Install ClickHouseInstallation example
We are going to install everything into `dev` namespace. `clickhouse-operator` is already installed into `dev` namespace

Check pre-initial position.
```bash
kubectl -n dev get all,configmap
```
`clickhouse-operator`-only is available
```text
NAME                                      READY   STATUS    RESTARTS   AGE
pod/clickhouse-operator-5cbc47484-v5cfg   1/1     Running   0          17s
NAME                                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/clickhouse-operator-metrics   ClusterIP   10.102.229.22   <none>        8888/TCP   17s
NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/clickhouse-operator   1/1     1            1           17s
NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/clickhouse-operator-5cbc47484   1         1         1       17s
```
Now let's install ClickHouse from provided examples:
```bash
kubectl -n dev apply -f 07-rolling-update-01-initial-position.yaml
```
Check initial position.
```bash
kubectl -n dev get all,configmap
```
ClickHouse is installed, 4 pods are running
```text
NAME                                      READY   STATUS    RESTARTS   AGE
pod/chi-d02eaa-347e-0-0-0                 1/1     Running   0          19s
pod/chi-d02eaa-347e-1-0-0                 1/1     Running   0          19s
pod/chi-d02eaa-347e-2-0-0                 1/1     Running   0          19s
pod/chi-d02eaa-347e-3-0-0                 1/1     Running   0          19s
pod/clickhouse-operator-5cbc47484-v5cfg   1/1     Running   0          95s

NAME                                  TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                         AGE
service/chi-d02eaa-347e-0-0           ClusterIP      None            <none>        8123/TCP,9000/TCP,9009/TCP      19s
service/chi-d02eaa-347e-1-0           ClusterIP      None            <none>        8123/TCP,9000/TCP,9009/TCP      19s
service/chi-d02eaa-347e-2-0           ClusterIP      None            <none>        8123/TCP,9000/TCP,9009/TCP      19s
service/chi-d02eaa-347e-3-0           ClusterIP      None            <none>        8123/TCP,9000/TCP,9009/TCP      19s
service/clickhouse-operator-metrics   ClusterIP      10.102.229.22   <none>        8888/TCP                        95s
service/clickhouse-rolling-update     LoadBalancer   10.106.139.11   <pending>     8123:31328/TCP,9000:32091/TCP   19s

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/clickhouse-operator   1/1     1            1           95s

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/clickhouse-operator-5cbc47484   1         1         1       95s

NAME                                   READY   AGE
statefulset.apps/chi-d02eaa-347e-0-0   1/1     19s
statefulset.apps/chi-d02eaa-347e-1-0   1/1     19s
statefulset.apps/chi-d02eaa-347e-2-0   1/1     19s
statefulset.apps/chi-d02eaa-347e-3-0   1/1     19s

NAME                                         DATA   AGE
configmap/chi-d02eaa-common-configd          2      19s
configmap/chi-d02eaa-common-usersd           0      19s
configmap/chi-d02eaa-deploy-confd-347e-0-0   1      19s
configmap/chi-d02eaa-deploy-confd-347e-1-0   1      19s
configmap/chi-d02eaa-deploy-confd-347e-2-0   1      19s
configmap/chi-d02eaa-deploy-confd-347e-3-0   1      19s
```
Let's explore one Pod in order to check available ClickHouse config files.
Navigate directly inside the Pod
```bash
kubectl -n dev exec -it chi-d02eaa-347e-0-0-0 -- bash
```
And take a look on ClickHouse config files available
```text
root@chi-d02eaa-347e-0-0-0:/etc/clickhouse-server# ls /etc/clickhouse-server/conf.d/
macros.xml
root@chi-d02eaa-347e-0-0-0:/etc/clickhouse-server# ls /etc/clickhouse-server/config.d
listen.xml  remote_servers.xml
```
Standard minimal set of config files is in place.
All is well.

## Update ClickHouse Installation - add replication
Let's run update - change `.yaml` manifest so we'll have replication.

Apply update file available 
```bash
kubectl -n dev apply -f 07-rolling-update-02-apply-update.yaml
```
And let's watch on how update is rolling over:
```text
NAME                                      READY   STATUS              RESTARTS   AGE
pod/chi-d02eaa-347e-0-0-0                 0/1     ContainerCreating   0          1s
pod/chi-d02eaa-347e-1-0-0                 1/1     Running             0          4m41s
pod/chi-d02eaa-347e-2-0-0                 1/1     Running             0          4m41s
pod/chi-d02eaa-347e-3-0-0                 1/1     Running             0          4m41s
```
As we can see, the first Pod is being updated - `pod/chi-d02eaa-347e-0-0-0`. 
Then we can see how the first replica is being created - `pod/chi-d02eaa-347e-0-1-0`, as well as 
next Pod is being updated - `pod/chi-d02eaa-347e-1-0-0` 
```text
NAME                                      READY   STATUS              RESTARTS   AGE
pod/chi-d02eaa-347e-0-0-0                 1/1     Running             0          21s
pod/chi-d02eaa-347e-0-1-0                 0/1     ContainerCreating   0          2s
pod/chi-d02eaa-347e-1-0-0                 0/1     ContainerCreating   0          2s
pod/chi-d02eaa-347e-2-0-0                 1/1     Running             0          5m1s
pod/chi-d02eaa-347e-3-0-0                 1/1     Running             0          5m1s
```
Update is rolling further.
```text
NAME                                      READY   STATUS              RESTARTS   AGE
pod/chi-d02eaa-347e-0-0-0                 1/1     Running             0          62s
pod/chi-d02eaa-347e-0-1-0                 1/1     Running             0          43s
pod/chi-d02eaa-347e-1-0-0                 1/1     Running             0          43s
pod/chi-d02eaa-347e-1-1-0                 1/1     Running             0          20s
pod/chi-d02eaa-347e-2-0-0                 1/1     Running             0          20s
pod/chi-d02eaa-347e-2-1-0                 0/1     ContainerCreating   0          1s
pod/chi-d02eaa-347e-3-0-0                 0/1     ContainerCreating   0          1s
```
And, after all Pod updated, let's take a look on ClickHouse config files available:
```text
root@chi-d02eaa-347e-0-0-0:/# ls /etc/clickhouse-server/conf.d/
macros.xml
root@chi-d02eaa-347e-0-0-0:/# ls /etc/clickhouse-server/config.d
listen.xml  remote_servers.xml  zookeeper.xml
```
We can see new file added: `zookeeper.xml`. It is Zookeeper configuration for ClickHouse. Let's take a look
```text
root@chi-d02eaa-347e-0-0-0:/# cat /etc/clickhouse-server/config.d/zookeeper.xml 
<yandex>
    <zookeeper>
        <node>
            <host>zookeeper.zoo1ns</host>
            <port>2181</port>
        </node>
    </zookeeper>
    <distributed_ddl>
        <path>/clickhouse/rolling-update/task_queue/ddl</path>
    </distributed_ddl>
</yandex>
```