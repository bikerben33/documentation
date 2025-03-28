---
title: CloudPrem
description: Learn how to deploy and manage Datadog CloudPrem, a self-hosted log solution for cost-effective log ingestion, indexing, and search capabilities
further_reading:
- link: "/logs/explorer/"
  tag: "Documentation"
  text: "Log Explorer"
- link: "/logs/log_configuration/"
  tag: "Documentation"
  text: "Log Configuration"
---

## Overview

Datadog CloudPrem is a self-hosted log solution which provides cost-effective log ingestion, indexing, and search capabilities in your own infrastructure. Designed to address data residency, security or high-volume requirements, CloudPrem seamlessly integrates with Datadog platform to offer comprehensive log analysis, visualization, and alerting while ensuring that your sensitive log data remains within your own infrastructure.

{{< img src="path/to/your/image-name-here.png" alt="CloudPrem architecture diagram" style="width:100%;" >}}

## Architecture

CloudPrem employs a decoupled architecture which separates the processes of indexing, searching, and data storage. This allows for independent scaling and optimization of different cluster components based on workload demands.

{{< img src="path/to/your/image-name-here.png" alt="CloudPrem component diagram" style="width:100%;" >}}

### Components

The CloudPrem cluster, typically deployed on Kubernetes (EKS), consists of several components:

**Indexers**
: Responsible for receiving log from Datadog agents, they process, index, and store logs in index files called splits to the object storage (ex: Amazon S3).

**Searchers**
: Handle incoming search queries from the Datadog UI via an API. They read metadata from the Metastore and relevant index data directly from the object storage.

**Metastore**
: Stores metadata about the indexes, including splits locations on the object storage. CloudPrem uses PostgreSQL for this purpose.

**Janitor / Control Plane**
: Responsible for tasks like indexing tasks scheduling and delete tasks.

## Get started

### Prerequisites

- AWS account
- Kubernetes cluster (EKS preferred)
- AWS Load Balancer Controller
- S3 bucket
- PostgreSQL database (RDS preferred)
- Datadog agent
- Kubernetes command line tool (`kubectl`)
- Helm command line tool (`helm`)

### Installation steps

1. Install and configure [AWS Load Balancer Controller][1]
2. Create S3 buckets
3. Install CloudPrem Helm chart
4. Set up DNS records
5. Configure Datadog agents
6. Configure your Datadog account

## Install

### Add Datadog helm repository

```bash
helm repo add datadog https://helm.datadoghq.com
helm repo update
```

### Create and customize your configuration file `datadog-values.yaml`

```bash
helm show values datadog/cloudprem
```

### Create a secret to store your metastore URI

```bash
kubectl create secret generic cloudprem-metastore-uri --from-literal QW_METASTORE_URI=postgres://<username>:<password>@<host>:5432/<db>
```

### Deploy CloudPrem

```bash
helm install <deployment name> datadog/cloudprem -f datadog-values.yaml
```

Here is an example of a configuration file `datadog-values.yaml`:

```yaml
image:
  tag: edge
  
serviceAccount:
  create: true
  name: cloudprem-demo
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::386798578047:role/cloudprem-demo
    eks.amazonaws.com/sts-regional-endpoints: "true"

config:
  default_index_root_uri: s3://cloudprem-demo/indexes
  metastore_uri: s3://cloudprem-demo/metastore

  storage:
    s3:
      region: us-east-1

ingress:
  public:
    create: true
  internal:
    create: true

indexer:
  replicaCount: 1
  resources:
    limits:
      cpu: "1"
      memory: "1Gi"
    requests:
      cpu: "1"
      memory: "1Gi"

searcher:
  replicaCount: 1
  resources:
    limits:
      cpu: "1"
      memory: "1Gi"
    requests:
      cpu: "1"
      memory: "1Gi"
```

### Configure your Datadog account

Currently, you need to ask Datadog's team to update your account so that you can search into your CloudPrem cluster from Datadog UI.

### Searching CloudPrem logs in the Logs Explorer

[Coming soon]

### Uninstall

```bash
helm uninstall <deployment name>
```

## Cluster sizing

In this section, we give recommendations to size your CloudPrem cluster.

### Indexers

- **Performance:** To index 5MB/s of logs, CloudPrem needs approximately 1 vCPU and 2GB of RAM.
- **Recommended Pod Sizes:** We recommend deploying indexer pods with either:
  - 2 vCPUs and 4GB of RAM
  - 4 vCPUs and 8GB of RAM
  - 8 vCPUs and 16GB of RAM
- **Storage:** An indexer stores temporary data and requires persistent storage (e.g., AWS EBS).
  - Minimum: 100GB per pod
  - Recommendation (for pods > 4 vCPUs): 200GB per pod
- **Example Calculation:** To index 1 TB per day (~11.6 MB/s):
  - Required vCPUs: `(11.6 MB/s / 5 MB/s/vCPU) ≈ 2.3 vCPUs`
  - Rounding up, you might start with one indexer pod configured with 3 vCPUs and 6GB RAM, requiring a 100GB EBS volume. (Adjust based on observed performance and redundancy needs)

### Searchers

- **Performance:** Search performance depends heavily on the workload (query complexity, concurrency, data scanned)
- **Rule of Thumb:** A general starting point is to provision roughly double the total number of vCPUs allocated to Indexers
- **Memory:** We recommend 4GB of RAM per searcher vCPU. Provision more RAM if you expect many concurrent aggregation requests

### Other services

The following components are typically lightweight:

- **Control Plane:** 1 vCPU, 2GB RAM
- **Metastore:** 1 vCPU, 2GB RAM
- **Janitor:** 1 vCPU, 2GB RAM

### Postgres Metastore backend

- **Instance Size:** For most use cases, a PostgreSQL instance with 1 vCPU and 4GB of RAM is sufficient
- **AWS RDS Recommendation:** If using AWS RDS, the `t4g.medium` instance type is a suitable starting point
- **High Availability:** Enable Multi-AZ deployment with one standby replica for high availability

<div class="alert alert-info">
Remember that these are starting recommendations. Monitor your cluster's performance and resource utilization closely and adjust sizing as needed.
</div>

## Monitoring

CloudPrem integrates directly with Datadog for monitoring. Deploy the Datadog agent on your k8s cluster with OTLP ingestion and Prometheus scraping enabled to start monitoring CloudPrem in Datadog. Datadog will then provide prebuilt dashboards to monitor CloudPrem.

## Metrics

CloudPrem exposes key metrics in the Prometheus format on the `/metrics` endpoint. You can use any front-end that supports Prometheus to examine the behavior of CloudPrem visually.

## Further reading

{{< partial name="whats-next/whats-next.html" >}}

[1]: https://kubernetes-sigs.github.io/aws-load-balancer-controller/latest/deploy/installation/ 