# Operating Elasticsearch in Kubernetes
## microXchg'19 - demo

[[slides.pptx](slides.pptx)] [[slides.pdf](slides.pdf)] [[Google Slides](https://docs.google.com/presentation/d/1RTgNHIbHmPUccfjyn4X44xwMc0i_-u92r6G-mYRCGNM/present)]

This demo is part of the talk 'Operating Elasticsearch in Kubernetes' by Mikkel Larsen and Oliver Trosien. It goes step-by-stop through the eployment of the [ES Operator](https://github.com/zalando-incubator/es-operator) into Kubernetes, and creation and scaling of an Elasticsearch cluster.

### Steps

Install custom resource definitions and ES Operator as Deployment

```
zkubectl apply -f manifests/es-operator.yaml
```

Watch the logs

```
zkubectl logtail -n microxchg es
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

Tunnel into Elasticsearch

```
zkubectl -n microxchg port-forward `zkubectl -n microxchg get pods -l application=elasticsearch,role=master -o custom-columns=:metadata.name --no-headers | head -n 1` 9200
```

Create an index

```
curl -XPUT localhost:9200/microxchg -HContent-type:application/json -d '{"number_of_shards":20, "number_of_replicas":2}'
```

Upgrade Elasticsearch (set image version to 6.6.2)

```
zkubectl -n microxchg edit eds es-data-microxchg
```

Allow downscaling (set minReplicas=1)

```
zkubectl -n microxchg edit eds es-data-microxchg
```

Watch ES Operator scale down the nodes