**######################################################################################################################**

**1.Core Foundations:-**

**######################################################################################################################**	

**Linux**

•	Linux FS, perms, processes: ls, cd, cat, ps, top, kill, chmod, chown.

•	Linux system admin basics: systemctl, journald, log analysis.

•	Editors: vim or nano. SSH keys, ~/.ssh/config.

•	Cron \& systemd basics.

•	Bash scripting for automation: variables, loops, functions, exit codes, set -euo pipefail.

•	SSH

•	Networking: HTTP, DNS, TCP/UDP, firewalls, routing tables.

**Git**

•	Understand Git deeply (branching, merging, rebasing, tagging).

•	Git: init/clone/branch/merge/rebase; clean commit messages; PR workflow, Git hooks.

**Etc**

•	Python essentials for automation (venv, argparse, requests, json, logging).

•	Networking: OSI vs practical, IP/DNS/HTTP/TLS, curl/wget, ss/netstat, tcpdump basics.

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

**Projects**

	Automation Scripts: Bash/Python to automate GCP tasks



**######################################################################################################################**

**2.Containers:-** 

**######################################################################################################################**

**Docker**

•	Docker: images, volumes, networking, multi-stage builds.

•	Push to Artifact Registry.

•	Dockerfile (multi stage), .dockerignore, tagging, docker run/exec/logs.

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

Docker Projects:-

•	Containerize a simple web app (Node.js or Nginx static). Push to Artifact Registry.

•	Stretch: Deploy to Cloud Run with HTTPS managed cert



**######################################################################################################################**

**3.CI/CD:-**

**######################################################################################################################**

**Cloud Build**

•	Learn YAML workflow syntax, triggers, and runners.

•	Compare with Cloud Build (create one Cloud Build trigger for same repo).

**GitHub Actions**

•	GitHub Actions: build/test/deploy pipeline for a static site.

**Jenkins**: 

•	Simple pipeline + GCP deployment.

•	Set up self hosted runner on a GCE VM (systemd service, least privilege SA).

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

**CI/CD Projects:-**

	GCP CI/CD pipeline with GitHub Actions.

•	Create a small e2e pipeline: build → test → push (Artifact Registry) → deploy (Cloud Run).

	CI/CD pipeline with GitHub Actions

	Deploy app to GKE using Cloud Build pipeline.

	Secure CI/CD Pipelines: Secrets management in GitHub Actions, Artifact Registry security.

•	One repo, two services: frontend on Cloud Run, backend on GCE MIG + LB.

•	CI/CD to both, dashboards + alerts + budget.

•	Debugging GKE networking issues, pod crashes, failed pipelines.

•	CI/CD: GitHub Actions or Cloud Build to apply plans w/ manual approval.



**######################################################################################################################**

**4.Compute on GCP:-**

**######################################################################################################################**

**Cloud Run**

•	Revisions \& traffic split, min/max instances, concurrency, CPU allocation.

•	Env vars, secrets via Secret Manager. Health checks

**Compute Engine**

Compute Engine (instance templates, MIG, autoscaling, startup scripts, health checks.)

GCS (storage classes, signed URLs, lifecycle rules, CMEK encryption.)

•	GCP Ops Agent for VMs.

**Load Balancing**

**Cloud DNS**

**Secret Manager**

**Pub/Sub**

**Cloud Functions**

**Cloud SQL**

**BigQuery basics**

**IAM**

**SSL certs**

SSL policies.



**######################################################################################################################**

**5.Kubernetes (GKE):-**

**######################################################################################################################**

**GKE choices:**

	Autopilot vs Standard

•	Ingress (GKE managed) or Gateway API; Google managed cert; HTTPS LB

**Objects**

&nbsp;	Core: Pods, Deployments, Services, ConfigMaps, Secrets, Ingress/Gateway, HPA, StatefulSets, Service (ClusterIP/NodePort), Probes, ReplicaSet,

&nbsp;	Requests/limits

•	ConfigMaps, Secrets; Workload Identity (no SA keys in pods!).

•	RBAC (least privilege), NetworkPolicy (isolate namespaces), Pod Security.

•	PV/PVC with Persistent Disk; StatefulSet basics.

•	Logs/metrics in GKE; kubectl debugging patterns; backups/snapshots.

**Helm basics**

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

**Kubernetes Projects:-**

	Dockerized app → GKE

	Multi-Environment Deployment: Dev, staging, prod with separate configs in Terraform \& GitHub Actions

	Simulate and resolve a GKE pod crash issue.

	Blue-Green \& Canary Deployments : Implement in GKE using Istio or native Ingress.

	Production-Ready App: Deploy a scalable, secure web app with full monitoring and CI/CD on GKE.

•	Deploy Docker app to GKE via GitHub Actions/Terraform.

•	Prometheus + Grafana setup in GKE.

•	Microservice app on GKE: API + worker + DB (Cloud SQL via private IP or Cloud SQL Auth Proxy).

•	HTTPS ingress, HPA, Workload Identity, NetworkPolicies, SLO 99% with alert.

	Alert on high CPU usage in GKE.

•	Choose scenario (e.g., 3 tier app: frontend Cloud Run, API on GKE, async worker via Pub/Sub; DB on Cloud SQL; Terraformed infra; CI/CD to each tier).

•	Draw architecture + define SLOs, rollout, rollback.



**######################################################################################################################**

**6.Networking on GCP:-**

**######################################################################################################################**

**Networking** 

&nbsp;	VPC, subnets, peering, Routes, Cloud NAT, firewall rules, load balancers, Private Google Access, Internal/External, LB patterns.

&nbsp;	VPC design, Shared VPC, Peering, Private Service Connect, Cloud DNS.

•	Private Google Access, Cloud NAT patterns, internal/external LBs, Cloud DNS, bastion via IAP.

•	VPC peering vs Cloud VPN (theory) + routing considerations.



**######################################################################################################################**

**7.IaC:-**

**######################################################################################################################**

**Terraform**

&nbsp;	google provider, modules, environments, remote state

•	Provider setup, state, variables/outputs, locals, data sources.

•	Remote state in GCS; state locking strategy; fmt/validate/plan/apply.

&nbsp;	Infra pipelines

&nbsp;	GCP: VPC, VM, GKE, Cloud SQL automation.

•	Create reusable modules (vpc, gke, cloudrun, monitoring).



**Cloud Deployment Manager**

•	Terraform on 

•	Modules \& workspaces.

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

**Terraform Projects:-**

	Terraform + GKE app deployment (multi-environment).

	Terraform → GCP infra

•	Write scripts for automating VM creation, backup, and log collection on GCP.

•	Deploy a GCP VM + firewall with Terraform.

•	Terraform a platform: VPC, GKE, AR, Secret Manager, Monitoring alerts, DNS + HTTPS LB.

•	Environments: dev/stage/prod via workspaces or directory layout; CI for terraform plan on PR.

•	Implement infra (Terraform) and app deployments; add GitHub Actions/Cloud Build + Cloud Deploy for progressive delivery.

•	2 technical mocks (K8s + Terraform). 1 system design for CI/CD on GCP.



**######################################################################################################################**

**8.Observability \& SRE:-**

**######################################################################################################################**

Cloud Logging/Monitoring

•	Cloud Logging \& structured logs, Cloud Monitoring dashboards, Uptime checks, alerting policies.

•	Structured logging best practices; alert fatigue fixes.

•	Error Reporting basics; budget alerts in Billing.

•	Incident response workflows.

•	Error budgets, SLOs, SLIs.

•	Prometheus + Grafana

•	Trace/Profiler

•	Runbooks

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

**Projects**

	GCP monitoring dashboard (Cloud Monitoring + Grafana).

•	Create custom metrics, log-based alerts.



**######################################################################################################################**

**9.Security \& Compliance:-**

**######################################################################################################################**

**IAM design**

•	IAM (custom roles, service accounts, workload identity)

•	IAM least privilege, VPC Service Controls, GCP security best practices.

•	IAM advanced concepts: custom roles, org policies, workload identity federation.

•	IAM strategy: SA per workload, custom roles, org policies (overview).

•	CI/CD security scanning (Trivy, Snyk).

•	Secret Manager, KMS for encryption at rest; AR vulnerability scans; Cloud Armor basics.

•	Container SBOM basics; AR vulnerability policy gates; canary/blue green releases.

•	WAF with Cloud Armor (managed rules) ahead of LB.

•	SA keys avoidance (Workload Identity)

•	Artifact Registry vulnerability scans



**######################################################################################################################**

**10.Cost \& Reliability:-**

**######################################################################################################################**

Autoscaling

Preemptibles

Rightsizing

Budget alerts

Postmortems.



