# APM-OTEL-Solarwinds
Lab Guidance: Modern Polyglot Application on K8s with SolarWinds Observability

## Introduction

This lab guidance allows you to deploy a modern, cloud-native polyglot application on Kubernetes. We will explore the fundamentals of OpenTelemetry (OTEL) using the community standard demo, and then demonstrate how to transition to a production-grade observability solution using **SolarWinds Observability SaaS (SWO)**.

The application used is based on the [OpenTelemetry Astronomy Shop Demo](https://opentelemetry.io/docs/demo/architecture/), a microservices-based application written in Java, Python, .NET, Go, Rust, and more.

### Lab Overview
1.  **Part 1: The OpenTelemetry Standard:** Deploying the vanilla OTEL demo to understand traces, metrics, Jaeger, and Grafana.
2.  **Part 2: SolarWinds Integration:** Deploying the **SolarWinds fork** of the demo for seamless SaaS integration, APM, and full-stack Kubernetes monitoring.
3.  **Part 3: Operational Use Cases:** Setting up Alerts, Incident Management (Squadcast), and Service Level Objectives (SLOs).

---

## Prerequisites

* **Kubernetes Cluster**: EKS, AKS, GKE, or a local cluster (Minikube/Kind).
* **Tools**: `kubectl`, `helm`, and `git` installed locally.
* **SolarWinds Account**: A valid SolarWinds Observability SaaS account.
* **API Token**: An **Ingestion** API Token from your SolarWinds settings.

---

## Part 1: The Standard OpenTelemetry Demo

In this section, we deploy the upstream OpenTelemetry demo. This helps us understand how microservices interact and how raw OTEL data is visualized using open-source tools.

### 1. Create a Namespace
We will keep this environment isolated in its own namespace.

```bash
kubectl create namespace otel-demo
```

### 2. Install via Helm
We use the official OpenTelemetry Helm chart to deploy the application and the OTEL Collector.

```helm repo add open-telemetry [https://open-telemetry.github.io/opentelemetry-helm-charts](https://open-telemetry.github.io/opentelemetry-helm-charts)
helm repo update

helm install my-otel-demo open-telemetry/opentelemetry-demo --namespace otel-demo
```

### 3. Expose the Application
To access the "Astronomy Shop" from the internet (e.g., via AWS LoadBalancer), we update the service type for the frontendproxy.

```
kubectl patch svc my-otel-demo-frontendproxy -n otel-demo -p '{"spec": {"type": "LoadBalancer"}}'
```
Wait for the External IP to be assigned:

```
kubectl get svc -n otel-demo my-otel-demo-frontendproxy --watch
```

### 4. Explore Open Source Tools



