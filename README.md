# airflow

Apache Airflow deployed with the [official Helm chart](https://airflow.apache.org/docs/helm-chart/stable/index.html) on a local Kubernetes cluster (this repo targets [OrbStack](https://orbstack.dev/) Kubernetes).

## Prerequisites

- [OrbStack](https://orbstack.dev/) installed
- In OrbStack: enable **Kubernetes** (Settings → Kubernetes → turn on)
- [kubectl](https://kubernetes.io/docs/tasks/tools/) and [Helm 3.10+](https://helm.sh/docs/intro/install/) on your PATH

Verify the cluster and context (OrbStack usually registers a context such as `orbstack`):

```bash
kubectl cluster-info
kubectl config current-context
```

## Run locally (OrbStack Kubernetes)

1. Add the Apache Airflow chart repository (once per machine):

```bash
helm repo add apache-airflow https://airflow.apache.org
helm repo update
```

2. Install or upgrade the release using this repo’s `values.yaml` (release name `airflow`, namespace `airflow`):

```bash
helm upgrade --install airflow apache-airflow/airflow \
  --namespace airflow \
  --create-namespace \
  -f values.yaml
```

3. Wait until workloads are ready:

```bash
kubectl get pods -n airflow -w
```

The chart provisions PostgreSQL, Redis (for Celery), workers, scheduler, triggerer, etc. First startup can take several minutes while images pull and jobs complete.

## Access the Airflow UI (API server)

Forward the chart’s API server service to your machine. With the default release name `airflow`, the service is typically named `airflow-api-server`:

```bash
kubectl port-forward svc/airflow-api-server 8080:8080 -n airflow
```

Then open [http://localhost:8080](http://localhost:8080). Initial login uses the chart’s default admin user (see [`webserver.defaultUser`](https://airflow.apache.org/docs/helm-chart/stable/parameters-ref.html#webserver) in the chart reference) unless you override it in `values.yaml`.

If the API service name differs from `airflow-api-server`, list services:

```bash
kubectl get svc -n airflow
```

## Update the Helm chart

1. Refresh chart versions from the repo:

```bash
helm repo update apache-airflow
```

2. See what would change (optional):

```bash
helm diff upgrade airflow apache-airflow/airflow -n airflow -f values.yaml
```

(Requires the [helm-diff](https://github.com/databus23/helm-diff) plugin; skip if you do not use it.)

3. Apply the upgrade:

```bash
helm upgrade airflow apache-airflow/airflow --namespace airflow -f values.yaml
```

To pin a specific chart version:

```bash
helm search repo apache-airflow/airflow --versions | head
helm upgrade airflow apache-airflow/airflow --namespace airflow -f values.yaml --version <chart-version>
```

Official upgrade notes: [Upgrading the deployed Helm Chart](https://airflow.apache.org/docs/helm-chart/stable/index.html#upgrading-the-deployed-helm-chart).

## Uninstall

```bash
helm uninstall airflow --namespace airflow
```

Some hook-created resources may remain; see the chart docs for cleanup details.
