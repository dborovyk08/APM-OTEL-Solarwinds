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
Once the app is running, generate traffic by clicking around the website. Then, use port-forwarding to inspect the data locally.
* **Jaeger (Traces)**:
```
kubectl port-forward svc/my-otel-demo-jaeger-query 16686:16686 -n otel-demo
```
Open http://localhost:16686 in your browser.
* **Grafana (Metrics)**:
```
kubectl port-forward svc/my-otel-demo-grafana 3000:3000 -n otel-demo
```
Open http://localhost:3000 in your browser.

* **Load-Balancing**
Better way is to use exposed IP from cloud load-balancer so you do need port-forwarding.

--

## Part 2: SolarWinds Observability Integration

For the second part of the lab, we switch to SolarWinds Observability.
Why use the SolarWinds Fork? For the best interoperability, we use the specific [SolarWinds Fork](https://github.com/solarwinds/opentelemetry-demo/tree/swo/kubernetes).

* It removes references to the generic pre-built OTEL collector.
* It automatically points all traffic to your SolarWinds SaaS account.
* It includes optimized instrumentation for APM (Java, Python, .NET, etc.).

### 1.Preparation
Create a new namespace for the production-style deployment.

```
kubectl create namespace swo-lab
```

### 2. Create the Secret
The SolarWinds fork expects your API token to be available as a Kubernetes secret. Replace <YOUR_SWO_TOKEN> with your actual token.

```
kubectl create secret generic swo-api-token \
  --from-literal=SWO_API_TOKEN=<YOUR_SWO_TOKEN> \
  --namespace swo-lab
```

### 3. Install SolarWinds Infrastructure Agent
To monitor the Kubernetes layer (Nodes, Pods, Cluster health) and act as a gateway, install the SolarWinds Agent.
Use values.yaml from this git.

```
helm repo add solarwinds [https://helm.solarwinds.io/](https://helm.solarwinds.io/)
helm repo update

helm repo add solarwinds https://helm.solarwinds.com && helm install -f values.yaml swo-k8s-collector solarwinds/swo-k8s-collector --namespace swo-lab --atomic
```

### 4. Deploy the SolarWinds Polyglot App
We will clone the specific branch configured for SolarWinds.

```
# Clone the SolarWinds fork
git clone -b swo/kubernetes [https://github.com/solarwinds/opentelemetry-demo.git](https://github.com/solarwinds/opentelemetry-demo.git)

# Navigate to the kubernetes directory
cd opentelemetry-demo/kubernetes

# Apply the manifests

kubectl apply -f ./ -n swo-lab
```

### 5. Add Digital Experience Monitoring (DEM)
To monitor the frontend as a real website and perform synthetic testing:

Expose the Service:

```
kubectl patch svc frontendproxy -n swo-lab -p '{"spec": {"type": "LoadBalancer"}}'
```

Configure in SaaS:

* Go to Digital Experience > Websites.

* Add the LoadBalancer URL found via kubectl get svc -n swo-lab.

* Create a Synthetic Check (Availability or Transaction) to ping the site every minute.
