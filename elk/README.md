# Elasticsearch, Logstash and Kibana (ELK)

For installation of eleasticsearch and Kibana check [EFK](../efk/README.md#lab)

:warning: AWS Fargate does not support daemonsets, so EFK and ELK [can not be installed on fargate profiles](https://discuss.elastic.co/t/elasticsearch-eck-on-aws-eks-hosted-on-fargate/332477/2). Injecting agent as sidecat to pods should work.
