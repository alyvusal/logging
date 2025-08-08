# Elasticsearch, Fluentd (FluentBit) and Kibana (EFK)

FluentD and FluentBit can be used together

:warning: AWS Fargate does not support daemonsets, so EFK and ELK [can not be installed on fargate profiles](https://discuss.elastic.co/t/elasticsearch-eck-on-aws-eks-hosted-on-fargate/332477/2). Injecting agent as sidecat to pods should work.

## [Fluentd](https://docs.fluentd.org/)

```bash
curl 127.0.0.1:24220/api/plugins.json
```

### [Installation](https://docs.fluentd.org/installation)

### [Configration](https://docs.fluentd.org/configuration)

- Multiple files can be used with `@include` directive
- Config file could be specified with `-c` or `--config` directive
- Check config file with `--dry-run` : `fluentd --dry-run -c fluent.conf` (Also see [Config validator](https://docs.fluentd.org/configuration/config-file#config-validator))
- `-s` or `--setup` create sample config
- [YAML syntax](https://docs.fluentd.org/configuration/config-file-yaml)

#### [Labels](https://docs.fluentd.org/configuration/plugin-common-parameters#label)

- Processing of event is done by label
- Useful for creating `match` ruels for multiple sources
- Reserved labels: `@ERROR`, `@FLUENT_LOG`

### [CLI](https://docs.fluentd.org/deployment/command-line-option)

## [FluentBit](https://docs.fluentbit.io/manual)

- Used for low resource environments
- Config file format is not XML style, `[]` used instead of `<>`
- Directives in FluentD is called sections in FluentBit
- We can also use CLI argument to replace config file

Comparison

|Fluentd|FluentBit|
|-|-|
|`[INPUT]`|`<source>`|
|`[SERVICE]`|`<system>`|
|`[FILTER]`|`<filter>`|
|`[OUTPUT]`|`<match>`|

:warning: **INFO**
The only input plugin that does NOT assign tags is Forward input. This plugin speaks the Fluentd wire protocol called Forward where every Event already comes with a Tag associated. Fluent Bit will always use the incoming Tag set by the client.

### [Tag](https://docs.fluentbit.io/manual/concepts/key-concepts#tag)

- Tags used but not inside the `[OUTPUT]` or `[FILTER]` sections.
- The replacement is `Match` and the key that is being matched.
- You can use `*` as the wildcard for the match.

### Plugins

- Uses `Name` instead of `@type` (For example: `Name tail`)

## [Docker config](https://docs.docker.com/engine/daemon/#configuration-file)

Add to `/etc/docker/daemon.json`

```json
{
"log-driver" : "fluentd",
"log-level": "debug",
"raw-logs": true,
"log-opts": {
    "env": "os,customer",
    "labels": "production_status,dev",
    "fluentd-retry-wait": "1s",
    "fluentd-max-retries": "600",
    "fluentd-sub-second-precision": "false",
    "tag": "{{.ID}}-{{.ImageID}}",
    "fluentd-address": "w.x.y.z:28080"
    }
}
```

## Lab

### k8s

Change to k8s folder

```bash
cd k8s
```

#### Kustomize

Kustomize will install all components of EFK stack
:warning: This method is not fully compatible yet, for example: kibana preinstall pod also installed at the same time with kibana pod etc.

```bash
kubectl create ns logging
kubectl kustomize --enable-helm kustomize/overlays | kubectl -n logging apply -f -
```

Cleanup

```bash
kubectl -n logging kustomize --enable-helm kustomize/overlays | kubectl -n logging delete -f -
```

#### Elasticsearch

```bash
# Operator
helm upgrade -i -n observability elastic-operator elastic/eck-operator
kubectl apply -f examples/Elasticsearch.yaml

# Helm
kubectl apply -f base-for-helm.yml
helm upgrade -i es elastic/elasticsearch --version 8.5.1 --namespace logging --create-namespace -f kustomize/base/helm/es-values.yml
```

Cleanup

```bash
helm uninstall es -n logging
```

#### Kibana

```bash
helm upgrade -i kibana elastic/kibana --version 8.5.1 --namespace logging
```

Cleanup

```bash
helm uninstall kibana -n logging
kubectl delete configmap kibana-kibana-helm-scripts -n logging
kubectl delete serviceaccount pre-install-kibana-kibana -n logging
kubectl delete roles pre-install-kibana-kibana -n logging
kubectl delete rolebindings pre-install-kibana-kibana -n logging
kubectl delete job pre-install-kibana-kibana -n logging
kubectl delete secrets kibana-kibana-es-token -n logging
```

##### Dashboard

[Dashboards](https://github.com/marandalucas/kibana-dashboard/tree/master)

#### Fluent-Bit

Consider increasing below sysctl settings

```ini
vm.max_map_count = 262144
fs.inotify.max_user_watches = 524288
fs.file-max = 2000000
fs.inotify.max_user_instances = 1500
```

```bash
helm upgrade -i fluent-bit fluent/fluent-bit --version 0.47.9 --namespace=logging -f k8s/kustomize/base/helm/fluent-bit-values.yml
```

Cleanup

```bash
helm uninstall fluent-bit -n logging
```

#### Fluentd

```bash
# kubectl create configmap custom-fluentd-config --from-file=examples/custom-fluentd.conf

helm upgrade -i fluentd fluent/fluentd \
  --namespace=logging \
  --version 0.5.2 \
  -f k8s/kustomize/base/helm/fluentd-values.yaml
  # --set configMap=custom-fluentd-config

```

:warning: Fluentd has following issue

```bash
failed to flush the buffer. retry_times=7 next_retry_time=2024-09-17 13:52:59 +0000 chunk="62250ef02f8387b5099c204771154dd2" error_class=Fluent::Plugin::ElasticsearchOutput::RecoverableRequestFailure error="could not push logs to Elasticsearch cluster ({:host=>\"elasticsearch-master\", :port=>9200, :scheme=>\"https\", :user=>\"elastic\", :password=>\"obfuscated\"}): [400] {\"error\":{\"root_cause\":[{\"type\":\"illegal_argument_exception\",\"reason\":\"Action/metadata line [1] contains an unknown parameter [_type]\"}],\"type\":\"illegal_argument_exception\",\"reason\":\"Action/metadata line [1] contains an unknown parameter [_type]\"},\"status\":400}"
```

adding `suppress_type_name true` not helped

Cleanup

```bash
helm uninstall fluentd -n logging
```

##### sidecar

[sidecar or docker log files](https://github.com/vipin-k/deploy-fluentd-sidecar-on-Kubernetes)

#### Log simulator

```bash
kubectl apply -f log-simulator.yml
```

## Prometheus

Set Up Prometheus to scrape Fluentd Metrics: Modify your Prometheus configuration (prometheus-config.yaml) to include Fluentd metrics:

```yaml
- job_name: 'fluentd'
  static_configs:
  - targets: ['fluentd.kube-system:24231']
```

## REFERENCE

- [Kubernetes Monitoring: A Complete Guide](https://www.kubecost.com/kubernetes-monitoring/)
