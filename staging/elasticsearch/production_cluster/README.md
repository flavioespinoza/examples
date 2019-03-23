# Elasticsearch on Kubernetes

[Kubernetes](https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/) makes it easy to build and scale [Elasticsearch](http://www.elasticsearch.org/) clusters. 

### Version
Elasticsearch version: `6.4.2`

### Best Practices
Before we start, one needs to know that `Elasticsearch` best-practices recommend to separate nodes in three roles:
* `Master` nodes - intended for clustering management only, no data, no HTTP API
* `Client` nodes - intended for client usage, no data, with HTTP API
* `Data` nodes - intended for storing and indexing your data, no HTTP API

This is enforced throughout this document.

### Important
Current `pod` descriptors use an `emptyDir` for storing data in each data node container. This is meant to be for the sake of simplicity and [should be adapted according to your storage needs](https://kubernetes.io/docs/concepts/storage/persistent-volumes/).

### Docker image

This service uses the [Official Docker Image](https://hub.docker.com/_/elasticsearch) for Elasticsearch

<h1 class="bt bb">Deploy</h1>

### Service Account
Create `service-account`

```bash {.copy-clip}
kubectl create -f staging/elasticsearch/production_cluster/service-account.yaml
```

### Discovery
Create `es-discovery`
```bash {.copy-clip}
kubectl create -f staging/elasticsearch/production_cluster/es-discovery-svc.yaml
```

### Service
Create `es-svc`
```bash {.copy-clip}
kubectl create -f staging/elasticsearch/production_cluster/es-svc.yaml
```

### Master
Wait until `service-account`, `es-discovery`, and `es-svc` are provisioned then create `es-master`

```bash {.copy-clip}
kubectl create -f staging/elasticsearch/production_cluster/es-master-rc.yaml
```

### Roll Based Access Control
 Create `Role` and `RoleBinding` for `RBAC` authorization requirement: [Read the doc](https://cloud.google.com/kubernetes-engine/docs/how-to/role-based-access-control)

```bash {.copy-clip}
kubectl create -f staging/elasticsearch/rbac.yaml
```

### Client
Wait until `es-master` is provisioned then create `es-client`

```bash {.copy-clip}
kubectl create -f staging/elasticsearch/production_cluster/es-client-rc.yaml
```

### Data
Wait until `es-client` is provisioned then create `es-data`

```bash {.copy-clip}
kubectl create -f staging/elasticsearch/production_cluster/es-data-rc.yaml
```

### Pods
Wait until `es-data` is provisioned then check `pods` to see if all is up and running

```bash {.copy-clip}
kubectl get pods
```

```bash {.copy-clip}
# Console output:
NAME              READY     STATUS    RESTARTS   AGE
es-client-2ep9o   1/1       Running   0          2m
es-data-r9tgv     1/1       Running   0          1m
es-master-vxl6c   1/1       Running   0          6m
```

### Logs
Check `Elasticsearch` master `logs`
```bash {.copy-clip}
kubectl logs es-master-vxl6c
```

```bash {.copy-clip}
# Console output:
log4j:WARN No such property [maxBackupIndex] in org.apache.log4j.DailyRollingFileAppender.
log4j:WARN No such property [maxBackupIndex] in org.apache.log4j.DailyRollingFileAppender.
log4j:WARN No such property [maxBackupIndex] in org.apache.log4j.DailyRollingFileAppender.
[2015-08-21 10:58:51,324][INFO ][node                     ] [Arc] version[1.7.1], pid[8], build[b88f43f/2015-07-29T09:54:16Z]
[2015-08-21 10:58:51,328][INFO ][node                     ] [Arc] initializing ...
[2015-08-21 10:58:51,542][INFO ][plugins                  ] [Arc] loaded [cloud-kubernetes], sites []
[2015-08-21 10:58:51,624][INFO ][env                      ] [Arc] using [1] data paths, mounts [[/data (/dev/sda9)]], net usable_space [14.4gb], net total_space [15.5gb], types [ext4]
[2015-08-21 10:58:57,439][INFO ][node                     ] [Arc] initialized
[2015-08-21 10:58:57,439][INFO ][node                     ] [Arc] starting ...
[2015-08-21 10:58:57,782][INFO ][transport                ] [Arc] bound_address {inet[/0:0:0:0:0:0:0:0:9300]}, publish_address {inet[/10.244.15.2:9300]}
[2015-08-21 10:58:57,847][INFO ][discovery                ] [Arc] myesdb/-x16XFUzTCC8xYqWoeEOYQ
[2015-08-21 10:59:05,167][INFO ][cluster.service          ] [Arc] new_master [Arc][-x16XFUzTCC8xYqWoeEOYQ][es-master-vxl6c][inet[/10.244.15.2:9300]]{data=false, master=true}, reason: zen-disco-join (elected_as_master)
[2015-08-21 10:59:05,202][INFO ][node                     ] [Arc] started
[2015-08-21 10:59:05,238][INFO ][gateway                  ] [Arc] recovered [0] indices into cluster_state
[2015-08-21 11:02:28,797][INFO ][cluster.service          ] [Arc] added {[Gideon][4EfhWSqaTqikbK4tI7bODA][es-data-r9tgv][inet[/10.244.59.4:9300]]{master=false},}, reason: zen-disco-receive(join from node[[Gideon][4EfhWSqaTqikbK4tI7bODA][es-data-r9tgv][inet[/10.244.59.4:9300]]{master=false}])
[2015-08-21 11:03:16,822][INFO ][cluster.service          ] [Arc] added {[Venomm][tFYxwgqGSpOejHLG4umRqg][es-client-2ep9o][inet[/10.244.53.2:9300]]{data=false, master=false},}, reason: zen-disco-receive(join from node[[Venomm][tFYxwgqGSpOejHLG4umRqg][es-client-2ep9o][inet[/10.244.53.2:9300]]{data=false, master=false}])
```
Elasticsearch cluster is up and running


<h1 class="bt bb">Scale</h1>

##### Scale each `node type` to handle your cluster

```bash {.copy-clip}
kubectl scale --replicas=3 rc es-master
kubectl scale --replicas=2 rc es-client
kubectl scale --replicas=2 rc es-data
```


##### Verify it worked

```bash {.copy-clip}
kubectl get pods
```

```bash {.copy-clip}
# Console output:
NAME              READY     STATUS    RESTARTS   AGE
es-client-2ep9o   1/1       Running   0          4m
es-client-ye5s1   1/1       Running   0          50s
es-data-8az22     1/1       Running   0          47s
es-data-r9tgv     1/1       Running   0          3m
es-master-57h7k   1/1       Running   0          52s
es-master-kuwse   1/1       Running   0          52s
es-master-vxl6c   1/1       Running   0          8m
```

##### Let's take another look of the Elasticsearch master logs

```bash {.copy-clip}
kubectl logs es-master-vxl6c
```

```bash {.copy-clip}
# Console output:
log4j:WARN No such property [maxBackupIndex] in org.apache.log4j.DailyRollingFileAppender.
log4j:WARN No such property [maxBackupIndex] in org.apache.log4j.DailyRollingFileAppender.
log4j:WARN No such property [maxBackupIndex] in org.apache.log4j.DailyRollingFileAppender.
[2015-08-21 10:58:51,324][INFO ][node                     ] [Arc] version[1.7.1], pid[8], build[b88f43f/2015-07-29T09:54:16Z]
[2015-08-21 10:58:51,328][INFO ][node                     ] [Arc] initializing ...
[2015-08-21 10:58:51,542][INFO ][plugins                  ] [Arc] loaded [cloud-kubernetes], sites []
[2015-08-21 10:58:51,624][INFO ][env                      ] [Arc] using [1] data paths, mounts [[/data (/dev/sda9)]], net usable_space [14.4gb], net total_space [15.5gb], types [ext4]
[2015-08-21 10:58:57,439][INFO ][node                     ] [Arc] initialized
[2015-08-21 10:58:57,439][INFO ][node                     ] [Arc] starting ...
[2015-08-21 10:58:57,782][INFO ][transport                ] [Arc] bound_address {inet[/0:0:0:0:0:0:0:0:9300]}, publish_address {inet[/10.244.15.2:9300]}
[2015-08-21 10:58:57,847][INFO ][discovery                ] [Arc] myesdb/-x16XFUzTCC8xYqWoeEOYQ
[2015-08-21 10:59:05,167][INFO ][cluster.service          ] [Arc] new_master [Arc][-x16XFUzTCC8xYqWoeEOYQ][es-master-vxl6c][inet[/10.244.15.2:9300]]{data=false, master=true}, reason: zen-disco-join (elected_as_master)
[2015-08-21 10:59:05,202][INFO ][node                     ] [Arc] started
[2015-08-21 10:59:05,238][INFO ][gateway                  ] [Arc] recovered [0] indices into cluster_state
[2015-08-21 11:02:28,797][INFO ][cluster.service          ] [Arc] added {[Gideon][4EfhWSqaTqikbK4tI7bODA][es-data-r9tgv][inet[/10.244.59.4:9300]]{master=false},}, reason: zen-disco-receive(join from node[[Gideon][4EfhWSqaTqikbK4tI7bODA][es-data-r9tgv][inet[/10.244.59.4:9300]]{master=false}])
[2015-08-21 11:03:16,822][INFO ][cluster.service          ] [Arc] added {[Venomm][tFYxwgqGSpOejHLG4umRqg][es-client-2ep9o][inet[/10.244.53.2:9300]]{data=false, master=false},}, reason: zen-disco-receive(join from node[[Venomm][tFYxwgqGSpOejHLG4umRqg][es-client-2ep9o][inet[/10.244.53.2:9300]]{data=false, master=false}])
[2015-08-21 11:04:40,781][INFO ][cluster.service          ] [Arc] added {[Erik Josten][QUJlahfLTi-MsxzM6_Da0g][es-master-kuwse][inet[/10.244.59.5:9300]]{data=false, master=true},}, reason: zen-disco-receive(join from node[[Erik Josten][QUJlahfLTi-MsxzM6_Da0g][es-master-kuwse][inet[/10.244.59.5:9300]]{data=false, master=true}])
[2015-08-21 11:04:41,076][INFO ][cluster.service          ] [Arc] added {[Power Princess][V4qnR-6jQOS5ovXQsPgo7g][es-master-57h7k][inet[/10.244.53.3:9300]]{data=false, master=true},}, reason: zen-disco-receive(join from node[[Power Princess][V4qnR-6jQOS5ovXQsPgo7g][es-master-57h7k][inet[/10.244.53.3:9300]]{data=false, master=true}])
[2015-08-21 11:04:53,966][INFO ][cluster.service          ] [Arc] added {[Cagliostro][Wpfx5fkBRiG2qCEWd8laaQ][es-client-ye5s1][inet[/10.244.15.3:9300]]{data=false, master=false},}, reason: zen-disco-receive(join from node[[Cagliostro][Wpfx5fkBRiG2qCEWd8laaQ][es-client-ye5s1][inet[/10.244.15.3:9300]]{data=false, master=false}])
[2015-08-21 11:04:56,803][INFO ][cluster.service          ] [Arc] added {[Thog][vkdEtX3ESfWmhXXf-Wi0_Q][es-data-8az22][inet[/10.244.15.4:9300]]{master=false},}, reason: zen-disco-receive(join from node[[Thog][vkdEtX3ESfWmhXXf-Wi0_Q][es-data-8az22][inet[/10.244.15.4:9300]]{master=false}])
```

<h1 class="bt bb">Access the Elasticsearch service</h1>

**Don't forget** that services in Kubernetes are only accessible from containers in the cluster. For different behavior you should [configure the creation of an external load-balancer](https://kubernetes.io/docs/concepts/services-networking/service/#type-loadbalancer). While it's supported within this example service descriptor, its usage is out of scope of this document, for now.

```bash {.copy-clip}
kubectl get service elasticsearch
```

```bash {.copy-clip}
NAME            LABELS                                SELECTOR                              IP(S)          PORT(S)
elasticsearch   component=elasticsearch,role=client   component=elasticsearch,role=client   10.100.134.2   9200/TCP
```

### Connect
From any host on your cluster (that's running `kube-proxy`), run:

```bash {.copy-clip}
curl http://10.100.134.2:9200
```

You should see something similar to the following:

```json
{
  "status" : 200,
  "name" : "node-1",
  "cluster_name" : "es-cluster-1",
  "version" : {
    "number" : "1.7.1",
    "build_hash" : "b88f43fc40b0bcd7f173a1f9ee2e97816de80b19",
    "build_timestamp" : "2015-07-29T09:54:16Z",
    "build_snapshot" : false,
    "lucene_version" : "4.10.4"
  },
  "tagline" : "You Know, for Search"
}
```

### Cluster Info
Check cluster information:

```bash {.copy-clip}
curl http://10.100.134.2:9200/_cluster/health?pretty
```

You should see something similar to the following:

```json
{
  "cluster_name" : "es-cluster-1",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 7,
  "number_of_data_nodes" : 2,
  "active_primary_shards" : 0,
  "active_shards" : 0,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0
}
```
