# Operating Elasticsearch in Kubernetes

## Friday Demo

[[slides.pptx](slides.pptx)] [[slides.pdf](slides.pdf)] [[Google Slides](https://docs.google.com/presentation/d/1RTgNHIbHmPUccfjyn4X44xwMc0i_-u92r6G-mYRCGNM/present)]

This demo is part of the talk 'Operating Elasticsearch in Kubernetes' by Mikkel Larsen and Oliver Trosien. It goes step-by-stop through the deployment of the [ES Operator](https://github.com/zalando-incubator/es-operator) into Kubernetes, and creation and scaling of an Elasticsearch cluster.

### Prerequesites

Have a Kubernetes cluster with two operator roles

* operator (needs access to kubernetes APIs, and priviledged container permissions)
* cdp (needs cross-namespace access to kubernetes APIs)


### Steps

Install custom resource definitions and ES Operator as Deployment

```
zkubectl apply -f manifests/es-operator.yaml
```

Watch the logs

```
zkubectl logtail -n demo es
```

Watch resource creation

```
watch zkubectl -n demo get all
```

Bootstrap Elasticsearch cluster

```
zkubectl apply -f manifests/elasticsearch-cluster.yaml
```

Create first EDS. It should scale to three Pods initially, as defined by `spec.minReplicas`.

```
zkubectl apply -f manifests/data-group1.yaml
```

Tunnel into Elasticsearch

```
zkubectl -n demo port-forward svc/es-http 9200
```

### Scaling Up

Create an index, and watch es-operator scale up to meet the maxShardsPerNode requirement.

```
curl -XPUT localhost:9200/demo -HContent-type:application/json -d '{"number_of_shards":21, "number_of_replicas":2}'
```

### Rolling Restart

Upgrade Elasticsearch (set image version to 7.3.1)

```
zkubectl -n demo edit eds es-data-demo1
```

### Scaling Down

Allow downscaling (set minReplicas=1, minIndexReplicas=0)

```
zkubectl -n demo edit eds es-data-demo1
```

Watch ES Operator scale down the nodes to two. It won't scale down to one node, as we would be violating the maxShardsPerNode again.
