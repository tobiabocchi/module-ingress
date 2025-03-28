# Ingress NGINX Single

<!-- <SD-DOCS> -->

Ingress NGINX is an Ingress Controller for [NGINX][nginx-page] web server and reverse-proxy, it manages NGINX in a Kubernetes native manner. This package deploys Ingress Controller for clusters created with `kubeadm`.

## Requirements

- Kubernetes >= `1.28.0`
- Kustomize >= `v5.6.0`
- `cert-manager`

## Image repository and tag

- Ingress NGINX image: `k8s.gcr.io/ingress-nginx/controller:v1.12.1`
- Ingress NGINX repo: [https://github.com/kubernetes/ingress-nginx](https://github.com/kubernetes/ingress-nginx)

## Configuration

Ingress NGINX single is deployed with the following default configuration:

- Maximum allowed size of the client request body: `10m`
- HTTP status code used in redirects: `301`
- Metrics are scraped by Prometheus every `10s`
- Validating Admission webhook that validates an ingress definition does not break NGINX configuration.

## Deployment

> ⚠️ You'll need to have deployed [`cert-manager`](../cert-manager/) first, the validating webhook needs to have TLS communication with the API server.

1. Add the module to your `Furyfile.yml`:

```yaml
bases:
  - name: ingress/nginx
    version: "v4.0.0
```

> See `furyctl` [documentation][furyctl-repo] for additional details about `Furyfile.yml` format.

2. Execute `furyctl vendor -H` to download the packages

3. Inspect the download packages under `./vendor/katalog/ingress/dual-nginx`.

4. Define a `kustomization.yaml` that includes the `./vendor/katalog/ingress/nginx` directory as resource.

```yaml
resources:
  - ./vendor/katalog/ingress/nginx
```

5. Apply the necessary patches. You can find a list of common customization [here](#common-customizations).

6. To deploy the packages to your cluster, execute:

```bash
kustomize build . | kubectl apply -f -
```

You are now ready to expose your applications using Kubernetes `Ingress` objects.

## Common customizations

`nginx` package is deployed by default as a `DaemonSet`, meaning that it will deploy 1 ingress-controller pod on every worker node.

This is probably NOT what you want, standard Fury clusters have at least 1 `infra` node (nodes that are dedicated to Fury infrastructural components, like Prometheus, OpenSearch, and the Ingress Controllers).

If your cluster has `infra` nodes you should patch the DaemonSet adding the `NodeSelector` for the `infra` nodes to the Ingress `DaemonSet`. You can do this using the following kustomize patch:

```yaml
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ingress-nginx-controller
spec:
  template:
    spec:
      nodeSelector:
        node-kind.sighup.io/infra: ""
```

If you don't have infra nodes and you don't want to run ingress controllers on all your worker nodes, you should probably label some nodes and adjust the previous `NodeSelector` accordingly.

## Alerts

Followings Prometheus [alerts][prometheus-alerts] are already defined for this package.

### NGINX Ingress Controller Alert Rules

| Parameter                           | Description                                                                                                                                         | Severity | Interval |
| ----------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | :------: |
| `NginxIngressDown`                  | This alert fires if Prometheus target discovery was not able to reach ingress-nginx-metrics in the last 15 minutes.                                 | critical |   15m    |
| `NginxIngressFailureRate`           | This alert fires if the failure rate (the rate of 5xx responses) measured on a time window of 2 minutes was higher than 10% in the last 10 minutes. | critical |   10m    |
| `NginxIngressFailedReload`          | This alert fires if the ingress' configuration reload failed in the last 10 minutes.                                                                | warning  |   10m    |
| `NginxIngressLatencyTooHigh`        | This alert fires if the ingress 99th percentile latency was more than 5 seconds in the last 10 minutes.                                             | warning  |   10m    |
| `NginxIngressLatencyTooHigh`        | This alert fires if the ingress 99th percentile latency was more than 10 seconds in the last 10 minutes.                                            | critical |   10m    |
| `NginxIngressCertificateExpiration` | This alert fires if the certificate for a given host is expiring in less than 7 days.                                                               | warning  |          |
| `NginxIngressCertificateExpiration` | This alert fires if the certificate for a given host is expiring in less than 1 day.                                                                | critical |          |

<!-- Links -->

[furyctl-repo]: https://github.com/sighupio/furyctl
[nginx-page]: https://nginx.org
[prometheus-alerts]: https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/

<!-- </SD-DOCS> -->

## License

For license details please see [LICENSE](../../LICENSE)
