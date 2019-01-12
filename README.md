### Manual Setup 

```
minikube start --cpus 2 --memory 12288
git clone https://github.com/adavarski/hadoop-helm-k8s/
cd docker
cp Dockerfile.2.9.0 Dockerfile
docker build -t davarski/hadoop:2.9.0 .
docker login
docker push davarski/hadoop:2.9.0
cd ../helm
vi values.yml
grep "hadoopVer" values.yaml 
hadoopVersion: 2.9.0
grep "image: davarski" values.yaml 
image: davarski/hadoop:2.9.0

kubectl create namespace hadoop
helm install . --name hadoop --namespace hadoop

$ kubectl get pod -nhadoop
NAME                                        READY   STATUS    RESTARTS   AGE
hadoop-hadoop-hdfs-dn-0                     1/1     Running   0          2m
hadoop-hadoop-hdfs-nn-0                     1/1     Running   0          2m
hadoop-hadoop-yarn-nm-0                     0/1     Running   0          2m
hadoop-hadoop-yarn-rm-0                     1/1     Running   0          2m
hadoop-hadoop-zepplin-nn-77b4cf5c5c-z7r97   1/1     Running   0          2m

$ helm del --purge hadoop
release "hadoop" deleted
$ kubectl delete namespace hadoop
namespace "hadoop" deleted

```
### Jenkins Setup
```
minikube ssh ->  get /etc/kubernetes/admin.conf and change localhost to 192.168.99.109 (minikube ip) 
kubectl --kubeconfig=./admin.conf get po

Setup jenkins pipeline 

```
# Hadoop Chart

[Hadoop](https://hadoop.apache.org/) is a framework for running large scale distributed applications.

This chart is primarily intended to be used for YARN and MapReduce job execution where HDFS is just used as a means to transport small artifacts within the framework and not for a distributed filesystem. Data should be read from cloud based datastores such as Google Cloud Storage, S3 or Swift.

## Chart Details

## Installing the Chart

To install the chart with the release name `hadoop` that utilizes 50% of the available node resources:

```
$ helm install . --name hadoop-1.0.0 --set image.tag=hadoop-1.0.0
```    

port fordwar to the Yarn UI    
```
$ kubectl port-forward hadoop-hadoop-yarn-rm-0 8088:8088
```

port fordwar to the Zepplin UI    
```
$ kubectl --namespace hadoop get pods 
$ kubectl --namespace hadoop port-forward hadoop-hadoop-zepplin-nn-{unique-value} 8080:8080
```

> Note that you need at least 2GB of free memory per NodeManager pod, if your cluster isn't large enough, not all pods will be scheduled.

The optional [`calc_resources.sh`](./tools/calc_resources.sh) script is used as a convenience helper to set the `yarn.numNodes`, and `yarn.nodeManager.resources` appropriately to utilize all nodes in the Kubernetes cluster and a given percentage of their resources. For example, with a 3 node `n1-standard-4` GKE cluster and an argument of `50`, this would create 3 NodeManager pods claiming 2 cores and 7.5Gi of memory.

### Persistence

To install the chart with persistent volumes:

```
$ helm install --name hadoop . \
  --set persistence.nameNode.enabled=true \
  --set persistence.nameNode.storageClass=standard \
  --set persistence.dataNode.enabled=true \
  --set persistence.dataNode.storageClass=standard \
  --set image.tag=hadoop-1.0.0
```

> Change the value of `storageClass` to match your volume driver. `standard` works for Google Container Engine clusters.

## Configuration

The following table lists the configurable parameters of the Hadoop chart and their default values.

| Parameter                                         | Description                                                                        | Default                                                          |
| ------------------------------------------------- | -------------------------------                                                    | ---------------------------------------------------------------- |
| `image`                                           | Hadoop image ([source](https://github.com/Comcast/kube-yarn/tree/master/image))    | `danisla/hadoop:{VERSION}`                                       |
| `imagePullPolicy`                                 | Pull policy for the images                                                         | `IfNotPresent`                                                   |
| `hadoopVersion`                                   | Version of hadoop libraries being used                                              | `{VERSION}`                                                      |
| `antiAffinity`                                    | Pod antiaffinity, `hard` or `soft`                                                 | `hard`                                                           |
| `hdfs.nameNode.pdbMinAvailable`                   | PDB for HDFS NameNode                                                              | `1`                                                              |
| `hdfs.nameNode.resources`                         | resources for the HDFS NameNode                                                    | `requests:memory=256Mi,cpu=10m,limits:memory=2048Mi,cpu=1000m`   |
| `hdfs.dataNode.replicas`                          | Number of HDFS DataNode replicas                                                   | `1`                                                              |
| `hdfs.dataNode.pdbMinAvailable`                   | PDB for HDFS DataNode                                                              | `1`                                                              |
| `hdfs.dataNode.resources`                         | resources for the HDFS DataNode                                                    | `requests:memory=256Mi,cpu=10m,limits:memory=2048Mi,cpu=1000m`   |
| `yarn.resourceManager.pdbMinAvailable`            | PDB for the YARN ResourceManager                                                   | `1`                                                              |
| `yarn.resourceManager.resources`                  | resources for the YARN ResourceManager                                             | `requests:memory=256Mi,cpu=10m,limits:memory=2048Mi,cpu=1000m`   |
| `yarn.nodeManager.pdbMinAvailable`                | PDB for the YARN NodeManager                                                       | `1`                                                              |
| `yarn.nodeManager.replicas`                       | Number of YARN NodeManager replicas                                                | `2`                                                              |
| `yarn.nodeManager.parallelCreate`                 | Create all nodeManager statefulset pods in parallel (K8S 1.7+)                     | `false`                                                          |
| `yarn.nodeManager.resources`                      | Resource limits and requests for YARN NodeManager pods                             | `requests:memory=2048Mi,cpu=1000m,limits:memory=2048Mi,cpu=1000m`|
| `persistence.nameNode.enabled`                    | Enable/disable persistent volume                                                   | `false`                                                          | 
| `persistence.nameNode.storageClass`               | Name of the StorageClass to use per your volume provider                           | `-`                                                              |
| `persistence.nameNode.accessMode`                 | Access mode for the volume                                                         | `ReadWriteOnce`                                                  |
| `persistence.nameNode.size`                       | Size of the volume                                                                 | `50Gi`                                                           |
| `persistence.dataNode.enabled`                    | Enable/disable persistent volume                                                   | `false`                                                          | 
| `persistence.dataNode.storageClass`               | Name of the StorageClass to use per your volume provider                           | `-`                                                              |
| `persistence.dataNode.accessMode`                 | Access mode for the volume                                                         | `ReadWriteOnce`                                                  |
| `persistence.dataNode.size`                       | Size of the volume                                                                 | `200Gi`                                                          |
