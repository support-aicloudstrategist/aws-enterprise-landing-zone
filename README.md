# AWS Enterprise Landing Zone — Reference Architecture

A reference implementation of a secure, multi-account **AWS Landing Zone** built with
Terraform — modeled on the patterns I use for enterprise cloud transformation and
migration programs (AWS Control Tower + Organizations, multi-account governance,
network baselines, IAM/RBAC, logging, and guardrails).

> Reference architecture / patterns repository — not a production deployment. Account
> IDs, CIDRs, and names are placeholders.

## Why a Landing Zone

A landing zone gives enterprises a **governed, repeatable foundation** before workloads
land in the cloud: account structure, identity, network, security baselines, logging,
and cost controls — codified as Infrastructure as Code so every account is consistent
and auditable.

## Architecture

```
                          ┌─────────────────────────────┐
                          │      AWS Organization        │
                          │      (Control Tower)         │
                          └──────────────┬──────────────┘
            ┌──────────────┬─────────────┼─────────────┬──────────────┐
        ┌───▼───┐      ┌───▼────┐   ┌────▼────┐    ┌────▼────┐    ┌────▼────┐
        │  Mgmt │      │Security│   │ Shared  │    │  Prod   │    │  Non-   │
        │ (root)│      │  / Log │   │ Services│    │   OU    │    │ Prod OU │
        └───────┘      │ Archive│   │   OU    │    └─────────┘    └─────────┘
                       └────────┘   └─────────┘
   Guardrails (SCPs) • CloudTrail org trail • Config aggregator • GuardDuty •
   Centralized logging (S3+KMS) • Transit Gateway hub • IAM Identity Center (SSO)
```

## What this codifies

| Domain | Pattern |
|--------|---------|
| **Account structure** | Management, Security/Log-Archive, Shared-Services, Prod / Non-Prod OUs via AWS Organizations |
| **Guardrails** | Service Control Policies (SCPs) — deny root, region lockdown, prevent CloudTrail/Config tampering |
| **Identity** | IAM Identity Center (SSO) permission sets; least-privilege IAM roles, no long-lived keys |
| **Network** | Hub-and-spoke with Transit Gateway, segmented VPCs, centralized egress, PrivateLink |
| **Logging & audit** | Organization CloudTrail, AWS Config aggregator, centralized S3 log archive with KMS + Object Lock |
| **Threat detection** | GuardDuty + Security Hub delegated to the Security account |
| **Cost** | Budgets, cost allocation tags, consolidated billing |

## Repo layout

```
.
├── modules/
│   ├── organizations/      # OUs + SCPs
│   ├── log-archive/         # centralized logging (S3 + KMS + Object Lock)
│   ├── network-hub/         # Transit Gateway + shared VPCs
│   └── identity-center/     # SSO permission sets
├── envs/
│   ├── management/
│   └── security/
├── main.tf
├── variables.tf
└── versions.tf
```

## Usage

```bash
terraform init
terraform plan  -var-file=envs/management/terraform.tfvars
terraform apply -var-file=envs/management/terraform.tfvars
```

## Well-Architected alignment

Maps to the **Security**, **Operational Excellence**, and **Cost Optimization** pillars of
the AWS Well-Architected Framework: least-privilege identity, immutable audit logs,
guardrails-as-code, and tag-driven cost governance.

---
Maintained by **Rajiv Das Gupta** — Cloud & Platform Architecture Leader · AWS/Azure/GCP ·
[linkedin.com/in/rajiv-dg-cloud](https://www.linkedin.com/in/rajiv-dg-cloud/)
