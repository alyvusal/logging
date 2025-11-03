# [Elastic Cloud on Kubernetes (ECK)](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s)

## [Deploy an orchestrator](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/deploy-an-orchestrator)

```bash
helm repo add elastic https://helm.elastic.co
helm repo update

# Install CRDs separately
helm upgrade -i elastic-operator-crds elastic/eck-operator-crds \
  -n elastic-system --create-namespace \
  --version 3.2.0

# Install operator
helm upgrade -i elastic-operator elastic/eck-operator \
  -n elastic-system --create-namespace \
  --version 3.2.0 \
  -f k8s/helm/eck-operator.yaml
```

- [Required RBAC permissions](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/required-rbac-permissions)
- [Configure operator](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/configure)

## [Manage deployments](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/manage-deployments)

Deployment could be done via `eck-stack` chart or via installing components individually

### Deploy with eck-stack

`eck-stack` chart can deploy following components. Get settings from individual chart values

- `eck-elasticsearch`
- `eck-kibana`
- `eck-agent`
- `eck-fleet-server`
- `eck-beats`
- `eck-logstash`
- `eck-apm-server`
- `eck-enterprise-search`

```bash
# This example install with disabled auth
helm upgrade -i elastic-stack elastic/eck-stack \
  -n logging --create-namespace \
  --version 0.17.0 \
  -f k8s/helm/eck-stack.yaml
```

### Deploy indivudually

#### [Deploy an Elasticsearch cluster](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/elasticsearch-deployment-quickstart)

```bash
# Create NS and secret
kubectl apply -f k8s/secret.yaml

# Install elasticsearch cluster
helm upgrade -i elasticsearch elastic/eck-elasticsearch \
  -n logging --create-namespace \
  --version 0.17.0 \
  -f k8s/helm/eck-elasticsearch.yaml

# List credentials
kubectl -n logging get secret -l eck.k8s.elastic.co/credentials=true
USER="elastic"
PASSWORD=$(kubectl -n logging get secret elasticsearch-eck-demo-es-elastic-user -o go-template='{{.data.elastic | base64decode}}')

# Check URL
curl -k -u "$USER:$PASSWORD" -k "https://elasticsearch-192.168.0.100.nip.io"
```

#### [Deploy a Kibana instance](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/kibana-instance-quickstart)

```bash
helm upgrade -i kibana elastic/eck-kibana \
  -n logging --create-namespace \
  --version 0.17.0 \
  -f k8s/helm/eck-kibana.yaml
```

Use Login credentials from elastic deployment. UI: [https://kibana-192.168.0.100.nip.io](https://kibana-192.168.0.100.nip.io)

## [Elastic Stack configuration policies](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/elastic-stack-configuration-policies)

This **requires a valid Enterprise license or Enterprise trial license**. Check the license documentation for more details about managing licenses.

examples are in [stack-config](./stack-config/) folder

```bash
kubectl get stackconfigpolicy
kubectl get -n b scp test-err-stack-config-policy -o jsonpath="{.status}" | jq .
```

## Beats

```bash
helm upgrade -i beats elastic/eck-beats \
  -n logging --create-namespace \
  --version 0.17.0 \
  -f k8s/helm/eck-beats.yaml

kubectl -n logging get beat
```

## REFERENCE

- [Github](https://github.com/elastic/cloud-on-k8s)
  - [Example elastichsearch values](https://github.com/elastic/cloud-on-k8s/blob/main/deploy/eck-stack/examples/elasticsearch/hot-warm-cold.yaml)
- [Elascticsearch configuration](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/elasticsearch-configuration)
  - [Settings managed by ECK](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/settings-managed-by-eck)
  - [Custom configuration files and plugins](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/custom-configuration-files-plugins)
- [Kibana configuration](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/kibana-configuration)
  - [Install Kibana plugins](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/k8s-kibana-plugins)
- [Beats configuration](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/beats)
  - [Examples](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/configuration-examples-beats)
    - [Filebeat with autodiscover](https://raw.githubusercontent.com/elastic/cloud-on-k8s/3.2/config/recipes/beats/filebeat_autodiscover.yaml)
    - [Filebeat with autodiscover for metadata: Labels](https://raw.githubusercontent.com/elastic/cloud-on-k8s/3.2/config/recipes/beats/filebeat_autodiscover_by_metadata.yaml)
    - [Filebeat without autodiscover](https://raw.githubusercontent.com/elastic/cloud-on-k8s/3.2/config/recipes/beats/filebeat_no_autodiscover.yaml)
    - [Metricbeat](https://raw.githubusercontent.com/elastic/cloud-on-k8s/3.2/config/recipes/beats/metricbeat_hosts.yaml)
    - [Elasticsearch and Kibana Stack Monitoring](https://raw.githubusercontent.com/elastic/cloud-on-k8s/3.2/config/recipes/beats/stack_monitoring.yaml)
- [Customize pods](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/customize-pods)
- [Propagate labels and annotations](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/propagate-labels-annotations)
- [Connect to external Elastic resources](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/connect-to-external-elastic-resources)
- [API Doc](https://www.elastic.co/docs/api/doc/elasticsearch/)
- [Autoscale](https://www.elastic.co/docs/deploy-manage/autoscaling/autoscaling-in-eck)
- [Stack monitoring](https://www.elastic.co/docs/deploy-manage/monitor/stack-monitoring/eck-stack-monitoring)
  - [Monitoring settings in Kibana](https://www.elastic.co/docs/reference/kibana/configuration-reference/monitoring-settings)
