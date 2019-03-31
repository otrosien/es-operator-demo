# Operating Elasticsearch in Kubernetes
## microXchg'19 - demo

This demo is part of the talk 'Operating Elasticsearch in Kubernetes' by Mikkel Larsen and Oliver Trosien. It goes step-by-stop through the eployment of the [ES Operator](https://github.com/zalando-incubator/es-operator) into Kubernetes, and creation and scaling of an Elasticsearch cluster.

### Steps

Install custom resource definitions and ES Operator as Deployment

```
zkubectl apply -f manifests/es-operator.yaml
```

Watch the logs

```
stern -n microxchg es
```

Watch resource creation

```
watch zkubectl -n microxchg get all
```

Bootstrap Elasticsearch cluster

```
zkubectl apply -f manifests/elasticsearch-cluster.yaml
```

Create first EDS

```
zkubectl apply -f manifests/data-group1.yaml
```

Create an index

```
http PUT localhost:9200/microxchg Content-type:application/json number_of_shards=20 number_of_replicas=2
```

Allow downscaling (set minReplicas=1)

```
zkubectl -n microxchg edit eds es-data-microxchg
```

Watch ES Operator scale down the nodes