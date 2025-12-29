# GitOps-Driven Canary Release with Argo CD & Istio (Minikube)

## Overview

This project demonstrates a **production-style GitOps workflow** using **Argo CD** and **Istio** to perform a **canary release (90/10 traffic split)** on Kubernetes, running locally on **Minikube**.

The focus of this project is:
- Declarative application delivery using GitOps
- Traffic management using Istio without redeploying workloads
- Real-world observability and troubleshooting
- Understanding common GitOps pitfalls and their resolution

---

## Architecture

Git Repository
↓
Argo CD (Auto-sync enabled)
↓
Kubernetes (Minikube)
↓
Istio Service Mesh
↓
Application Traffic (Canary Release)

### Design Principles
- Argo CD runs **outside the Istio mesh**
- Application workloads run **inside the Istio mesh**
- Traffic shifting is controlled **entirely via Istio**
- Git is the **single source of truth**

---

## Technology Stack

- Kubernetes: Minikube (Docker driver)
- GitOps: Argo CD
- Service Mesh: Istio (demo profile)
- Observability:
  - Kiali (service mesh visualization)
  - Prometheus (metrics)
  - Jaeger (distributed tracing)
- Application: Istio Bookinfo sample

---

## Repository Structure

bookinfo-gitops/
├── base/
│ ├── bookinfo.yaml
│ ├── destination-rule.yaml
├── canary/
│ └── reviews-traffic-90-10.yaml
└── argocd/
└── application.yaml


---

## Implementation Details

### 1. GitOps Application Management
- Created an Argo CD `Application` resource pointing to this repository
- Enabled:
  - Automated sync
  - Self-healing
  - Resource pruning
- Verified sync state via Argo CD UI and CLI

---

### 2. Istio Service Mesh Setup
- Enabled automatic sidecar injection for application namespace
- Deployed Bookinfo services into the mesh
- Configured Istio Gateway and Ingress
- Verified Envoy sidecar injection (`READY 2/2`)

---

### 3. Canary Release (90/10 Traffic Split)
- Defined `DestinationRule` with subsets:
  - `reviews-v1`
  - `reviews-v3`
- Applied `VirtualService` to split traffic:
  - 90% → `reviews-v1`
  - 10% → `reviews-v3`
- Traffic changes applied **only via Git commits**
- No application redeployment required

---

### 4. Observability & Validation
- Kiali used to visualize live traffic flow
- Prometheus used to validate traffic distribution numerically
- Jaeger available for request tracing
- Argo CD Diff and History used to verify applied changes

---

## Real-World Debugging & Lessons Learned

### Key Issue Identified
Argo CD initially showed the application as **Synced**, but traffic changes were not applied.

### Root Cause
- Argo CD Application was using **Directory mode**
- **Directory recursion was disabled**
- Manifests inside subdirectories (`base/`, `canary/`) were silently ignored

### Resolution
- Enabled `directory.recurse: true`
- Argo CD immediately detected missing manifests
- Auto-sync applied Istio configuration correctly
- Traffic behavior updated instantly

### Key Insight
> **“Argo CD ‘Synced’ does not guarantee all manifests are applied unless directory recursion is configured correctly.”**

This is a common real-world GitOps pitfall.

---

## Verification Checklist

- Argo CD Application status: `Synced`, `Healthy`
- VirtualService weights verified via `kubectl`
- Envoy sidecars injected in application pods
- Traffic observed in Kiali graph
- Metrics confirmed in Prometheus

---

## Outcome

- Implemented a fully GitOps-driven canary release
- Demonstrated safe traffic shifting without redeployments
- Built a production-relevant reference project
- Captured and resolved a real GitOps configuration issue

---

## Next Steps

- Convert manifests into a Helm chart
- Parameterize traffic weights via `values.yaml`
- Use Argo CD with Helm source
- Implement progressive rollout (5% → 25% → 50%)
- Add rollback automation

---

## Why This Project Matters

This project reflects how modern platform teams deploy applications:
- Git as the source of truth
- Declarative traffic management
- Observability-driven validation
- Minimal manual intervention

---

## Author

**Naresh Maran**  
Cloud / Platform / SRE Engineer  
