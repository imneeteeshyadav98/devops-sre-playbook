# 🚀 DevOps SRE Playbook

<p align="center">
  <b>Public DevOps/SRE notes, Kubernetes upgrade learnings, incident reviews, Kubepreflight ideas, and Platform Engineering interview prep.</b>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Focus-Kubernetes%20Upgrade%20Readiness-blue" />
  <img src="https://img.shields.io/badge/AWS-EKS-orange" />
  <img src="https://img.shields.io/badge/SRE-Production%20Reliability-green" />
  <img src="https://img.shields.io/badge/Platform%20Engineering-DevOps-purple" />
  <img src="https://img.shields.io/badge/IaC-Terraform-623CE4" />
  <img src="https://img.shields.io/badge/CI%2FCD-Automation-black" />
</p>

---

## 🎯 Current Goal

Become a stronger **Senior SRE / Platform Engineer** with deep hands-on expertise in:

- Kubernetes/EKS upgrade readiness
- API deprecation and removed API detection
- Admission webhook and PDB upgrade blockers
- EKS add-ons and lifecycle management
- Production reliability and observability
- Terraform, CI/CD, automation, and platform tooling

---

## 🧑‍💻 What I Am Building

| Project | Purpose | Focus |
|---|---|---|
| **Kubepreflight** | Kubernetes upgrade preflight and readiness tool | Upgrade blockers, risk scoring, action plan |
| **AWS EOL Monitor** | AWS service lifecycle and upgrade-risk tracking | EKS, RDS, OpenSearch, Lambda, ElastiCache |
| **DevOps/SRE Playbook** | Public learning and interview-prep repo | Notes, incidents, checklists, product ideas |

---

## 🧭 Why This Repo Exists

Kubernetes upgrades can fail because of hidden production risks.

| Risk | Production Impact |
|---|---|
| **Removed APIs** | Deployments, Helm, GitOps, or Terraform apply can fail |
| **Deprecated APIs** | Future upgrade risk |
| **Admission webhooks** | API writes can get blocked |
| **PodDisruptionBudgets** | Node drain and upgrade can get stuck |
| **Unhealthy workloads** | Upgrade starts from an already unstable state |
| **Node version skew** | Worker nodes may become incompatible |
| **EKS add-on mismatch** | DNS, CNI, proxy, storage, or autoscaling can break |
| **Weak rollback plan** | Recovery becomes slow during outage |

> This repo captures my daily learning and product thinking around these real production problems.

---

## 📚 Topics Covered

| Area | Notes |
|---|---|
| **Kubernetes** | Upgrade basics, API deprecations, webhooks, PDBs, version skew |
| **AWS EKS** | Upgrade planning, add-ons, lifecycle, node groups |
| **SRE** | Incident learning, reliability, observability, post-upgrade validation |
| **Kubepreflight** | Product ideas, rules backlog, readiness score, action plan |
| **EOL Monitor** | AWS lifecycle monitoring and service upgrade risk |
| **Interview Prep** | Practical answers for DevOps/SRE/Platform roles |
| **Job Search** | Resume keywords, target profile, remote/hybrid DevOps roles |

---

## 🗂 Repository Structure

```text
devops-sre-playbook/
├── kubernetes/
│   ├── upgrade-basics.md
│   ├── api-deprecation-lifecycle.md
│   ├── admission-webhooks.md
│   ├── pdb-and-node-drain.md
│   └── version-skew.md
│
├── eks/
│   ├── eks-upgrade-checklist.md
│   ├── eks-addons.md
│   └── eks-lifecycle.md
│
├── incidents/
│   └── reddit-piday-outage.md
│
├── kubepreflight/
│   ├── product-ideas.md
│   ├── rules-backlog.md
│   ├── readiness-score.md
│   └── action-plan.md
│
├── eol-monitor/
│   ├── aws-eol-monitoring.md
│   └── service-lifecycle-checklist.md
│
├── interview-prep/
│   ├── kubernetes-upgrade-answers.md
│   ├── sre-answers.md
│   └── aws-devops-answers.md
│
└── job-search/
    ├── target-profile.md
    └── resume-keywords.md
```

---

## 🛠 Kubepreflight Thinking

Kubepreflight should help engineers answer one important question:

> **Can we safely start this Kubernetes/EKS upgrade?**

Planned and active rule ideas:

| Rule | Purpose | Priority |
|---|---|---|
| **API-001** | Detect removed APIs before target upgrade | P1 |
| **API-002** | Detect deprecated APIs still in use | P2 |
| **WH-001** | Detect fail-closed webhook blockers | P1 |
| **PDB-001** | Detect node drain risks | P2 |
| **WORKLOAD-001** | Detect unhealthy workloads before upgrade | P4 |
| **LABEL-001** | Detect old `node-role.kubernetes.io/master` label usage | P2 |
| **ADDON-001** | Detect EKS add-on compatibility risk | P2 |

---

## 📊 Upgrade Readiness Model

Kubepreflight should not only show findings. It should provide a clear decision summary.

```text
Upgrade Readiness Score: 72/100

Verdict: NOT READY

Blockers: 1
Warnings: 4
Info: 6

Top reasons:
1. Fail-closed admission webhook may block API writes
2. Removed API is still in use
3. PDB may block node drain during upgrade
```

Core idea:

```text
Score gives confidence.
Blocker decides readiness.
Priority decides what to fix first.
```

---

## 🔥 Incident Learnings

I study real production incidents and convert them into SRE/platform engineering lessons.

| Incident | Learning |
|---|---|
| **Reddit Pi-Day Outage** | Kubernetes upgrade broke hidden Calico/node-label dependency |

Each incident note includes:

- Root cause
- Production impact
- Detection signal
- Recovery steps
- How Kubepreflight could prevent or reduce the risk

---

## 🎤 Interview Preparation

This repo helps me prepare practical answers for questions like:

- How do you plan a safe EKS upgrade?
- What are the biggest risks during Kubernetes upgrades?
- How do admission webhooks block deployments?
- How do PDBs affect node drain?
- How do you handle deprecated APIs before upgrade?
- How do you design production-ready DevOps automation?
- How do you validate a cluster after upgrade?

---

## ✅ Current Progress

| Date | Progress |
|---|---|
| 2026-07-11 | Created DevOps SRE Playbook repository |
| 2026-07-11 | Added Kubernetes upgrade, API deprecation, Kubepreflight, and incident learning structure |
| 2026-07-11 | Started documenting production Kubernetes upgrade risks |

---

## 🧑‍💼 Target Profile

I am positioning myself toward:

```text
Senior SRE / Platform Engineer
focused on AWS EKS, Kubernetes upgrade automation,
Terraform, CI/CD, observability, and production reliability.
```

My core profile:

- AWS / EKS
- Kubernetes / Docker / Helm
- Terraform / Ansible
- CI/CD automation
- Prometheus / Grafana / Loki
- Production support and incident handling
- Kubernetes upgrade and EOL readiness automation

---

## 📌 Note

This is a living repository.  
I update it as I learn, build, test, and improve my understanding of real-world DevOps/SRE problems.

The goal is simple:

> **Build proof that I understand production Kubernetes, SRE thinking, and platform automation beyond just using tools.**