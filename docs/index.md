# From Diagram to Deploy: Stock Trader on AKS with Terraform

*The setup repo that makes the sample reproducible*

Microservices demos are fun… right up until you try to recreate them.

Suddenly you’re not “running the sample” anymore — you’re stitching together Kubernetes, networking, databases, secrets, ingress, DNS, and just enough security to not feel like a toy. And then you hit the classic trap: Terraform finishes successfully… and the app still isn’t reachable.

That’s the exact gap `stocktrader-setup` exists to close. It turns the Stock Trader architecture into a repeatable, team-friendly deployment — starting with Azure + AKS — so you can spend time learning the system (or extending it), not babysitting infrastructure.

By the end of this post, you’ll know what the repo sets up and how to get to a working URL in minutes.

Who this is for: anyone who wants to run Stock Trader on Azure without hand-assembling cloud resources.

If you’re evaluating the architecture, contributing a service, or setting up a demo environment, this repo is your starting point.

## Quick context: what is Stock Trader?

Stock Trader is a cloud-native sample made of multiple containerized microservices designed to run on Kubernetes. It was created at IBM and is now primarily maintained by Kyndryl (by the folks who originally built it).

If you want an “off-the-shelf but realistic” playground for cloud-native patterns — service-to-service calls, state, optional integrations, and real deployment concerns — this is the one.

## Why this repo exists: “operators deploy apps”, not platforms

Stock Trader has an umbrella operator that can install and configure the microservices — but it does **not** provision the prerequisite platform services (databases, messaging, etc.). It expects you to bring those, then provide connection details.

So `stocktrader-setup` takes a strong stance:

> First build the platform layer. Then wire Stock Trader on top. Repeatably.

## Reading the architecture diagram like a story (not a wiring schematic)

This architecture is easiest to understand as two worlds:

![Stock Trader on Azure AKS](https://raw.githubusercontent.com/IBMStockTrader/stocktrader-setup/main/azure/public/diagram.png)

### 1) What runs in AKS (the “trading floor”)

This is the Kubernetes side: services like Trader, Broker, Portfolio, Trade History, Stock Quote, Account/Cash Account, plus supporting pieces like messaging/notifications.

Traffic comes in from the UI (browser/mobile), and a Looper microservice can be enabled to simulate activity and test the system under load.

### 2) What Azure provides (the platform backbone)

This is the managed layer: data services, cache, secrets/identity, networking, and optional integrations (eventing, functions, alternate data stores, etc.).

The punchline: Stock Trader is built to plug into hyperscaler primitives. `stocktrader-setup` makes provisioning those primitives on Azure predictable.

## The 60-second path: a “boring” workflow that actually works

TL;DR: authenticate → precheck → init/plan/apply → verify → get URL.

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

That last step is the difference between a demo and a deployment mindset:

- `precheck.sh` is your bouncer: it catches missing tools, misconfig, and “this name already exists” problems before you waste time provisioning.  
- `make postcheck` is the real MVP: it validates what you normally discover the hard way (cluster connectivity, app readiness, key dependencies reachable, routing behaving).  
- `make app-url` removes the “ok but where is it exposed?” guessing game.

## What stocktrader-setup provisions on Azure (in human terms)

At a high level, the Azure implementation creates a production-leaning baseline for running Stock Trader on AKS — not a giant everything-and-the-kitchen-sink build. Think of it as a baseline on Azure (AKS + Postgres + Redis + Key Vault) that you can rely on and extend.

Think of it as:

- **AKS cluster + networking** (the foundation)  
- **Managed data services** (so you’re not hand-rolling stateful infra)  
- **Secrets/identity wiring** (so credentials aren’t scattered across YAML files)  
- **Bootstrap + app install** (so Stock Trader is actually deployed, not “ready for you to deploy”)

One important note: the diagram shows the broader ecosystem Stock Trader can integrate with (Cosmos/SQL/Eventing/etc.). This repo focuses on the baseline you need to get a reliable deployment up and running on Azure.

The repo is also structured to grow into other clouds over time: Azure is the complete implementation today, with room for more providers later.

## The “opinionated” parts that save you hours

### 1) Fast failure beats slow debugging

A lot of Terraform repos stop at “apply succeeded.”

This one leans into prechecks + post-deploy verification, because in Kubernetes land, the painful failures are usually after infrastructure is technically created.

### 2) Istio is supported — but it’s not a trap

Service mesh is where demos go to die:

- sidecars not injected  
- wrong labels / wrong revision  
- gateways exist but routing doesn’t  

The setup supports running **with or without** the mesh and gives you a clear “this is your URL” outcome either way.

### 3) Team-ready habits (so it doesn’t collapse after day 1)

If more than one person will touch the environment, remote state is the difference between collaboration and chaos.

The repo recommends a shared backend so you don’t end up with two laptops applying changes, state files living in random folders, and drift nobody can explain.

## Where this fits in the bigger Stock Trader lifecycle

Stock Trader is most valuable when you can iterate:

> change code → build/push → deploy → validate → repeat

That’s how you learn microservices patterns for real — not by staring at diagrams.

`stocktrader-setup` is what makes the environment stable enough to support that loop on Azure: cluster, wiring, prerequisites, and a workflow you can re-run without crossing your fingers.

## Closing: a setup repo that makes “try the sample” mean something

A microservices demo is easy to watch and painfully hard to recreate.

`stocktrader-setup` makes the diagram deployable — in a way that’s repeatable, verifiable, and shaped for real teams — so you can focus on what matters: understanding the system, extending services, testing integrations, and shipping changes.

If you want to try it now, start here: run the Quick Start in the setup repo, then use `make app-url` to get your first working endpoint.

## References

- Terraform setup: <https://github.com/IBMStockTrader/stocktrader-setup>  
- Application code: <https://github.com/IBMStockTrader>  
- Azure-specific docs and architecture: `azure/README.md` and `azure/ARCHITECTURE.md` in the setup repo.
