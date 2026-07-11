# DevOps SRE Playbook

<p align="center">
  <b>My public learning and engineering notes for becoming a stronger Senior SRE / Platform Engineer.</b>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Focus-Kubernetes%20Upgrade%20Readiness-blue" />
  <img src="https://img.shields.io/badge/AWS-EKS-orange" />
  <img src="https://img.shields.io/badge/SRE-Production%20Reliability-green" />
  <img src="https://img.shields.io/badge/Platform%20Engineering-DevOps-purple" />
</p>

---

## Current Goal

Become a stronger **Senior SRE / Platform Engineer** with deep hands-on expertise in:

- Kubernetes/EKS upgrade readiness
- API deprecation and removed API detection
- Admission webhook and PDB upgrade blockers
- EKS add-ons and lifecycle management
- Production reliability and observability
- Terraform, CI/CD, automation, and platform tooling

I am actively working on:

| Project | Purpose |
|---|---|
| **Kubepreflight** | Kubernetes upgrade preflight and readiness tool |
| **AWS EOL Monitor** | AWS service lifecycle and upgrade-risk tracking |
| **DevOps/SRE Playbook** | Public notes, incident reviews, interview prep, and production checklists |

---

## Why This Repo Exists

Kubernetes upgrades can fail because of hidden production risks:

| Risk | Production Impact |
|---|---|
| Removed APIs | Deployments, Helm, GitOps, or Terraform apply can fail |
| Deprecated APIs | Future upgrade risk |
| Admission webhooks | API writes can get blocked |
| PodDisruptionBudgets | Node drain and upgrade can get stuck |
| Unhealthy workloads | Upgrade starts from an already unstable state |
| Node version skew | Worker nodes may become incompatible |
| EKS add-on mismatch | DNS, CNI, proxy, storage, or autoscaling can break |
| Weak rollback plan | Recovery becomes slow during outage |

This repo captures my daily learning and product thinking around these problems.

---

## Topics Covered

| Area | Notes |
|---|---|
| Kubernetes | Upgrade basics, API deprecations, webhooks, PDBs, version skew |
| AWS EKS | Upgrade planning, add-ons, lifecycle, node groups |
| SRE | Incident learning, reliability, observability, post-upgrade validation |
| Kubepreflight | Product ideas, rules backlog, readiness score, action plan |
| EOL Monitor | AWS lifecycle monitoring and service upgrade risk |
| Interview Prep | Practical answers for DevOps/SRE/Platform roles |
| Job Search | Resume keywords, target profile, remote/hybrid DevOps roles |

---

## Repository Structure

```text
devops-sre-playbook/
├── kubernetes/
│   ├── upgrade-basics.md
│   ├── api-deprecation-lifecycle.md
│   ├── admission-webhooks.md
│   ├── pdb-and-node-drain.md
│   └── version-skew.md
├── eks/
│   ├── eks-upgrade-checklist.md
│   ├── eks-addons.md
│   └── eks-lifecycle.md
├── incidents/
│   └── reddit-piday-outage.md
├── kubepreflight/
│   ├── product-ideas.md
│   ├── rules-backlog.md
│   ├── readiness-score.md
│   └── action-plan.md
├── eol-monitor/
│   ├── aws-eol-monitoring.md
│   └── service-lifecycle-checklist.md
├── interview-prep/
│   ├── kubernetes-upgrade-answers.md
│   ├── sre-answers.md
    └── aws-devops-answers.md