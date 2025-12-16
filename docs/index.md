# From Diagram to Deploy: Stock Trader on AKS with Terraform

*The setup repo that makes the sample reproducible*

Microservices demos are fun until you try to recreate them.

At that point you are not really “running the sample.” You are setting up Kubernetes, networking, databases, secrets, ingress, DNS, and enough security to make it feel real. Then you hit the classic problem: Terraform completes, and the app still does not load.

That is the gap `stocktrader-setup` is meant to close. It turns the Stock Trader architecture into a repeatable, team-friendly deployment on Azure AKS, so you can spend your time learning the system (or extending it) instead of babysitting infrastructure.

By the end of this post, you will know what the repo sets up and how to get a working URL in minutes.

Who this is for: anyone who wants to run Stock Trader on Azure without assembling cloud resources by hand. If you are evaluating the architecture, contributing a service, or setting up a demo environment, this repo is a good starting point.

## Quick context: what is Stock Trader?

Stock Trader is a cloud-native sample made of multiple containerized microservices designed to run on Kubernetes. It was created at IBM and is now primarily maintained by Kyndryl (by many of the same folks who built it originally).

If you want an “off-the-shelf but realistic” playground for cloud-native patterns like service-to-service calls, state, optional integrations, and real deployment concerns, Stock Trader is a great choice.

## Why this repo exists: operators deploy apps, not platforms

Stock Trader has an umbrella operator that can install and configure the microservices. What it does not do is provision the prerequisite platform services (databases, messaging, and so on). Those are expected to exist already, with connection details provided.

That is why `stocktrader-setup` takes a straightforward approach:

> Build the platform layer first. Then install Stock Trader on top. Do it in a way you can repeat.

## Reading the architecture diagram like a story

This diagram is easiest to understand in two parts:

![Stock Trader on Azure AKS](https://raw.githubusercontent.com/IBMStockTrader/stocktrader-setup/main/azure/public/diagram.png)

### 1) What runs in AKS

This is the Kubernetes side: services like Trader, Broker, Portfolio, Trade History, Stock Quote, Account/Cash Account, plus supporting pieces like messaging and notifications.

Traffic comes in through the UI (browser or mobile). There is also a Looper microservice that can be enabled to generate activity for demos and testing.

### 2) What Azure provides

This is the managed layer underneath the cluster: data services, cache, secrets and identity, networking, and optional integrations such as eventing or serverless components.

Stock Trader is designed to plug into these kinds of cloud primitives. `stocktrader-setup` is what makes provisioning the Azure side predictable.

## A quick start flow that actually works

The repo keeps the happy path simple:

```bash
# 1) Auth + sanity
az login
bash precheck.sh

# 2) Deploy
make init
make plan
make apply

# 3) Verify + access
make postcheck
make app-url
```

A few small helpers make a big difference here:

- `precheck.sh` catches missing tools, misconfig, and name collisions early.  
- `make postcheck` validates what usually fails after “apply succeeded,” like cluster access, app readiness, and dependency connectivity.  
- `make app-url` prints the endpoint so you are not hunting for where the app is exposed.

## What stocktrader-setup provisions on Azure (in human terms)

The Azure implementation creates a production-leaning baseline for running Stock Trader on AKS. It is not meant to deploy every optional integration shown in the diagram. Think of it as a reliable starting point you can extend.

In practice, it sets up a baseline on Azure built around AKS plus core dependencies such as Postgres, Redis, and Key Vault.

At a high level, you can think of it as:

- **AKS cluster and networking**, which everything runs on  
- **Managed data services**, so you are not running stateful infrastructure by hand  
- **Secrets and identity wiring**, so credentials are not scattered across YAML files  
- **Bootstrap and app install**, so Stock Trader is deployed as part of the flow

One important note: the diagram shows the broader ecosystem Stock Trader can integrate with (Cosmos, SQL Server, eventing, and more). This repo focuses on the baseline needed to deploy Stock Trader reliably on Azure and reach a working endpoint.

The repo is also structured to grow into other clouds over time. Azure is the complete implementation today, with room for more providers later.

## The choices that save you time

### 1) Catch problems early

A lot of Terraform repos stop at “apply succeeded.” This one leans into prechecks and post-deploy verification because Kubernetes problems often show up after infrastructure is technically created.

### 2) Istio support without forcing it

Service mesh can add value, but it can also introduce friction. The setup supports running with or without Istio and still aims to give you a clear, usable application URL either way.

### 3) Collaboration that does not fall apart

If more than one person touches an environment, remote state matters. The repo recommends a shared backend so you avoid the common failure modes: two laptops applying changes, state files scattered across machines, and drift nobody can explain.

## Where this fits in the larger Stock Trader workflow

Stock Trader is most valuable when you can iterate quickly. Make a change, deploy it, validate, and repeat. That is how you learn microservices patterns for real.

`stocktrader-setup` gives you the stable environment needed to do that on Azure: cluster, wiring, prerequisites, and a workflow you can re-run with confidence.

## Closing

Microservices demos are easy to watch and hard to recreate.

`stocktrader-setup` makes the architecture deployable in a repeatable, verifiable way. It helps you get to the part that actually matters: understanding the system, extending services, testing integrations, and shipping changes.

If you want to try it, start with the Quick Start in the setup repo and use `make app-url` to get your first working endpoint.

## References

- Terraform setup: <https://github.com/IBMStockTrader/stocktrader-setup>  
- Application code: <https://github.com/IBMStockTrader>  
- Azure docs and architecture: `azure/README.md` and `azure/ARCHITECTURE.md` in the setup repo.
