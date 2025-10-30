# Elasticsearch, Logstash and Kibana (ELK)

For installation of eleasticsearch and Kibana check [EFK](../efk/README.md#elasticsearch)

:warning: AWS Fargate does not support daemonsets, so EFK and ELK [can not be installed on fargate profiles](https://discuss.elastic.co/t/elasticsearch-eck-on-aws-eks-hosted-on-fargate/332477/2). Injecting agent as sidecat to pods should work.

## Logstash

```bash
helm upgrade -i logstash elastic/logstash \
  -n logging --create-namespace \
  --version 8.5.1 \
  -f k8s/helm/logstash.yaml
```

## Filebeat

```bash
# Send logs to logstash -> Elasticsearch
helm upgrade -i filebeat elastic/filebeat \
  -n logging --create-namespace \
  --version 8.5.1 \
  -f k8s/helm/filebeat-to-logstash.yaml
```
