# One Command to Trade: How We Open-Sourced Stock Trader’s Terraform Playbook

Hello! I’m Anand from the Cloud Journey Optimization Team. We just took the Terraform that powers our IBM Stock Trader environments and opened it to the world. This isn’t another copy-paste install guide—think of it as the story behind the paved path we ship to teammates and partners who want a secure, production-ish Stock Trader in minutes, not weeks.

## Why we built a paved path
Stock Trader is a polyglot microservices app (Trader, Broker, Portfolio, Stock Quote, Cash Account, and friends). The app is fun; the plumbing is not. Clusters, meshes, databases, secrets, gateways, and dashboards can spiral into a yak-shave. We packaged all of that into Terraform so anyone with an Azure subscription can spin up a ready-to-demo environment with the same defaults we trust internally.

## What “done” looks like
- Azure AKS with overlay networking (keeps the VNet small, pods happy).
- Istio service mesh with mTLS and an external gateway so `/trader` is reachable over HTTPS.
- Managed PostgreSQL + Redis, locked behind private endpoints and private DNS.
- Azure Key Vault wired to Kubernetes via External Secrets Operator—no static creds in pods.
- Stock Trader operator + Custom Resource applied, so the app just shows up.
- Optional sentiment dashboard (Azure OpenAI + AI Search) for an AI-flavored demo.

## The 10-minute experience (from our perspective)
1) You log in to Azure and run the repo’s `precheck.sh`. It refuses to continue if a dependency, name, or permission is off.  
2) You tweak `terraform.tfvars` with a handful of names and an OIDC secret.  
3) `make init && make plan && make apply`—Terraform handles the rest.  
4) `make postcheck` tells you if Istio, pods, DB, Redis, and secrets are healthy.  
5) `make app-url` prints the HTTPS link and default credentials. You’re in.

## Opinionated, but flexible
- Toggle Istio off if you want a simple LoadBalancer instead of a mesh.  
- Reuse the same overlay CIDRs across environments to avoid IP math.  
- Bring your own Azure OpenAI/AI Search, or let Terraform create them.  
- Swap SKUs and sizes without breaking the private-endpoint/DNS wiring.

## Why this matters
- **Speed:** From empty subscription to working app before your coffee cools.  
- **Safety:** Private endpoints, mTLS, Workload Identity, and validated variables out of the box.  
- **Reproducibility:** Same modules we run; predictable outcomes for demos, POCs, and labs.  
- **Extensibility:** The repo is structured for more clouds (AWS/GCP slots are waiting) and new managed services.

## If you want to dig deeper
The repo has the gritty details—architecture diagrams, module-by-module READMEs, and troubleshooting tips (sidecars, IPs, auth). Start in `azure/README.md` for the walkthrough, or peek at `ARCHITECTURE.md` to see how the overlay + Istio + private endpoints fit together.

Thanks for reading! If you kick the tires, we’d love feedback or PRs. The goal is simple: fewer yak-shaves, more time spent experimenting with the app itself.

