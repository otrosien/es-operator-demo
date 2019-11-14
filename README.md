# Operating Elasticsearch in Kubernetes

## Demo

[[Google Slides](https://docs.google.com/presentation/d/1uv9EB7iRG39Um6MzE3cOP6VABWuHLE_4nmPNQkqt4fs/present)]

This demo is part of the talk 'Operating Elasticsearch in Kubernetes' by Mikkel Larsen and Oliver Trosien. It goes step-by-stop through the deployment of the [ES Operator](https://github.com/zalando-incubator/es-operator) into Kubernetes, and creation and scaling of an Elasticsearch cluster.

## Prerequisites

Have a Kubernetes cluster at hand (e.g. [kind](https://github.com/kubernetes-sigs/kind) or [minikube](https://github.com/kubernetes/minikube/)), and `kubectl` configured to point to it.

## Step 1 - Set up Roles

The ES Operator needs special permissions to access Kubernetes APIs, and Elasticsearch needs privileged access to increase the operating system limits for memory-mapped files.

Therefore as the first step we deploy a serviceAccount `es-operator` with the necessary RBAC roles attached.

```
kubectl apply -f manifests/cluster-roles.yaml
```

## Step 2 - Register Custom Resource Definitions

The ES Operator manages two custom resources. These need to be registered in your cluster.

```
kubectl apply -f manifests/custom-resource-definitions.yaml
```


## Step 3 - Deploy ES Operator

Next, we'll deploy our operator. It will be created from a deployment manifest in the namespace `demo`, and pull the latest image.

```
kubectl apply -f manifests/es-operator.yaml
```

You can check if it was successfully launched:

```
kubectl -n demo get pods
```

## Step 4 - Bootstrap Your Elasticsearch Cluster

The Elasticsearch will be boot-strapped from a set of master nodes. For the purpose of this demo, a single master is sufficient. For production a set of three masters is recommended.

```
kubectl apply -f manifests/elasticsearch-cluster.yaml
```

The manifest also creates services for the transport and HTTP protocols. If you tunnel to port 9200 on the master, you should be able to communicate with your Elasticsearch cluster.

```
kubectl -n demo port-forward svc/es-http 9200
```

## Step 5 - Add Elasticsearch Data Sets

Finally, let's add data nodes. For the purpose of this demo we have a simple stack will launch one data node, and scales up based on the number of shards and CPU usage.

```
kubectl apply -f manifests/es-data1.yaml
```

To check the results, first look for the custom resources.

```
kubectl -n demo get eds
```

The ES Operator creates a StatefulSet, which will spawn the Pod. This can take a few minutes depending on your network and cluster performance.

```
kubectl -n demo get sts
kubectl -n demo get pods
```

## Step 6: Index Creation

We differentiated the stacks using an Elasticsearch node tag called `group`. It is advised to use this tag to bind indices with the same scaling requirements to nodes with the same `group` tag, by using the shard allocation setting like this:

 ```
curl -XPUT localhost:9200/demo-index1 -HContent-type:application/json \
 -d '{"settings": {"index": { "number_of_shards":5, "number_of_replicas":0, "routing.allocation.include.group": "data1"}}}'
 ```

## Step 7: Scaling Up

Create another index, and watch es-operator scale up to meet the maxShardsPerNode requirement.

```
curl -XPUT localhost:9200/demo-index2 -HContent-type:application/json  -d '{"settings": {"index": { "number_of_shards":6, "number_of_replicas":0, "routing.allocation.include.group": "data1"}}}'
```
