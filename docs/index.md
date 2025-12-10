# One Command to Trade: Deploy IBM Stock Trader on Azure with Terraform

Skip two days of wiring clusters, meshes, databases, and secrets—go from an empty Azure subscription to a secure, HTTPS Stock Trader app in under an hour with one `make apply`.

## Contents
- [At a glance](#at-a-glance)
- [Why we built a paved path](#why-we-built-a-paved-path)
- [What “done” looks like](#what-done-looks-like)
- [The 10-minute experience](#the-10-minute-experience)
- [Opinionated, but flexible](#opinionated-but-flexible)
- [Why this matters](#why-this-matters)
- [If you want to dig deeper](#if-you-want-to-dig-deeper)
- [Call to action](#call-to-action)

## At a glance
- **What:** Terraform blueprint for IBM Stock Trader on Azure AKS, with Istio, private PostgreSQL, Redis, and Key Vault + External Secrets.  
- **Why:** Eliminate yak-shaving; ship a production-ish demo path with sane defaults.  
- **How:** `precheck.sh`, a small `terraform.tfvars`, then `make init/plan/apply`, verify, and grab the URL.  
- **Extras:** Optional sentiment dashboard (Azure OpenAI + AI Search) and mesh toggle.  
- **See also:** [Terraform repo](https://github.com/IBMStockTrader/stocktrader-setup) · [Stock Trader app](https://github.com/IBMStockTrader)
- **Tested:** Terraform 1.x; AKS with Istio `asm-1-24`; observed deploy time: ~30–45 minutes end-to-end.

## Why we built a paved path
Stock Trader is a polyglot microservices app (Trader, Broker, Portfolio, Stock Quote, Cash Account, and friends). The app is fun; the plumbing is not. Clusters, meshes, databases, secrets, gateways, and dashboards can spiral into a yak-shave. We packaged all of that into Terraform so anyone with an Azure subscription can spin up a ready-to-demo environment with the same defaults we trust internally.

## What “done” looks like
- Azure AKS with Azure CNI overlay networking (keeps the VNet small, pods happy).  
- Istio service mesh with mTLS and an external gateway so `/trader` is reachable over HTTPS.  
- Managed PostgreSQL + Redis behind private endpoints and private DNS.  
- Azure Key Vault wired to Kubernetes via External Secrets Operator—no static creds in pods.  
- Stock Trader operator + Custom Resource applied, so the app just shows up.  
- Optional sentiment dashboard (Azure OpenAI + AI Search) for an AI-flavored demo.

![Architecture](https://raw.githubusercontent.com/IBMStockTrader/stocktrader-setup/main/azure/public/diagram.png)

## The 10-minute experience
1) `az login && bash precheck.sh` — refuse to continue if a dependency, name, or permission is off.  
2) Copy `terraform.tfvars.example`, fill in a handful of names and an OIDC secret.  
3) `make init && make plan && make apply` — Terraform provisions AKS, Istio, data services, secrets, and the app.  
4) `make postcheck` confirms Istio, pods, DB, Redis, and secrets are healthy.  
5) `make app-url` prints the HTTPS link and default credentials. You’re in.

## Opinionated, but flexible
- Toggle Istio off if you want a simple LoadBalancer instead of a mesh.  
- Reuse the same overlay CIDRs across environments to avoid IP math.  
- Bring your own Azure OpenAI/AI Search, or let Terraform create them.  
- Swap SKUs and sizes without breaking the private-endpoint/DNS wiring.

## Why this matters
- **Speed:** Empty subscription → working app before your coffee cools.  
- **Safety:** Private endpoints, mTLS, Workload Identity, and validated variables out of the box.  
- **Reproducibility:** Same modules we run; predictable outcomes for demos, POCs, and labs.  
- **Extensibility:** Structured for more clouds (AWS/GCP slots are waiting) and new managed services.

## If you want to dig deeper
- Walkthrough: `azure/README.md` in the [Terraform repo](https://github.com/IBMStockTrader/stocktrader-setup/blob/main/azure/README.md).  
- Architecture and networking: `ARCHITECTURE.md` (overlay + Istio + private endpoints).  
- Troubleshooting: common fixes for sidecars, IPs, and auth are baked into the docs.

## Call to action
Clone the repo, run `make apply`, and tell us how it went. Issues and PRs welcome—the goal is simple: fewer yak-shaves, more time experimenting with the app itself.
