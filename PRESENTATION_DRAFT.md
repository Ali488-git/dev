# Cloud Infrastructure Overview

## High-Level Overview of Current Cloud Infrastructure

### How It Works

- **Infrastructure as Code (IaC)**: All infrastructure is defined and managed through Terraform code stored in GitHub repositories
- **Version Control**: Every infrastructure change is tracked, reviewed, and approved through Git workflows
- **CI/CD Pipeline**: GitHub Actions automatically triggers on code changes to validate and deploy infrastructure
- **Azure Deployment**: Infrastructure is provisioned and managed in Azure cloud platform
- **State Management**: Terraform maintains infrastructure state either in Azure Storage or Terraform Cloud for consistency and collaboration
- **Policy Enforcement**: Azure Policies are applied to ensure compliance and governance across all deployments
- **Multi-Environment Support**: Single codebase supports Dev, SIT, UAT, and Prod environments through workspace management

---

## GitHub Repositories & Infrastructure Management

| Repository | Link | Status | Backend Type | Priority | Description |
|---|---|---|---|---|---|
| **Azure IaC** | [metrolinx/Azure-IaC](https://github.com/metrolinx/Azure-IaC) | In Scope | Azure Storage (Templated) | P1 | Main repository for Dev, SIT, UAT, Prod Terraform code and GitHub Actions deployment pipeline. Shared infrastructure; multi-environment; 60+ workload directories |
| **Azure Products IaC** | [metrolinx/Azure-products-IaC](https://github.com/metrolinx/Azure-products-IaC) | In Scope | Mixed (Azure Storage + TFC) | P1 | Dev workspace for the team. Products/workloads; split state detected; some Terraform Cloud already in use |
| **AZ Policy Repo** | [metrolinx/az-policy-repo](https://github.com/metrolinx/az-policy-repo) | In Scope | Azure Storage (Templated) | P2 | Azure policy definitions. Policy-as-Code; management group scope |
| **Ansible Terraform Integration** | [metrolinx/Ansible-Terraform-Integration](https://github.com/metrolinx/Ansible-Terraform-Integration) | In Scope | Azure Storage (Templated) | P2 | Ansible playbooks for infrastructure orchestration and integration with Terraform |
| **Ansible Terraform Cloud Integration** | [metrolinx/Ansible-TerraformCloud-Integration](https://github.com/metrolinx/Ansible-TerraformCloud-Integration) | In Scope | Terraform Cloud (Remote) | P3 | Ansible playbooks for automatic Terraform Cloud resource provisioning for ServiceNow tickets |

---

## Terraform Cloud

**Link**: [app.terraform.io/app/metrolinx/workspaces](https://app.terraform.io/app/metrolinx/workspaces)

**Description**: Remote state management and workspace orchestration platform used across multiple repositories for centralized Terraform state management, VCS integration, and automated deployments. Enables team collaboration and provides enhanced security for sensitive infrastructure variables.

---

## Azure Subscription Mapping

> **Note**: This section outlines our Azure subscription mapping. Each repository utilizes a combination of these subscriptions based on which environments it is being deployed to.

| Subscription Display Name | Subscription Reference | Purpose | Environments |
|---|---|---|---|
| it-sub-sharedconnect-001 | data.azurerm_subscriptions.infraconnect | Shared connectivity | prod |
| it-sub-sharedmgmt-001 | data.azurerm_subscriptions.mgmt | Management plane | prod |
| it-sub-dev-001 | data.azurerm_subscriptions.infradev | Development workloads | dev |
| it-sub-devdmz-001 | data.azurerm_subscriptions.infradevdmz | Dev DMZ | dev |
| it-sub-sit-001 | data.azurerm_subscriptions.infrasit | SIT workloads | sit |
| it-sub-sitdmz-001 | data.azurerm_subscriptions.infrasitdmz | SIT DMZ | sit |
| it-sub-uat-001 | data.azurerm_subscriptions.infrauat | UAT workloads | uat |
| it-sub-uatdmz-001 | data.azurerm_subscriptions.infrauatdmz | UAT DMZ | uat |
| it-sub-prd-001 | data.azurerm_subscriptions.infraprd | Production workloads | prd |
| it-sub-prddmz-001 | data.azurerm_subscriptions.infraprddmz | Production DMZ | prd |
| it-sub-sbox-cloudteam-001 | data.azurerm_subscriptions.sbox | Sandbox/testing | sbox |

---

## Current Workflow with GitHub Actions

```
                                    ┌─────────────┐
                                    │    START    │
                                    └──────┬──────┘
                                           │
                                           ▼
    ┌──────────────────────────────────────────────────┐
    │   Developer commits Terraform code to GitHub     │
    │   (Feature branch or direct to main)             │
    └──────────────────┬───────────────────────────────┘
                       │
                       ▼
    ┌──────────────────────────────────────────────────┐
    │   GitHub Actions Workflow Triggered              │
    │   - Runs on push/pull request events             │
    └──────────────────┬───────────────────────────────┘
                       │
                       ▼
    ┌──────────────────────────────────────────────────┐
    │   Terraform Validate & Format Check              │
    │   - terraform fmt                                │
    │   - terraform validate                           │
    └──────────────────┬───────────────────────────────┘
                       │
                       ▼
                   ◇─────────◇
                  ╱           ╲
                 ╱  Validation ╲
                ╱   Successful?  ╲
               ◇                  ◇
               │ No           Yes │
               │                  │
               └─────┬────────────┘
                     │
                     ▼
    ┌──────────────────────────────────────────────────┐
    │   Terraform Plan                                 │
    │   - Generates execution plan                     │
    │   - Stores plan artifact                         │
    └──────────────────┬───────────────────────────────┘
                       │
                       ▼
    ┌──────────────────────────────────────────────────┐
    │   PR Review & Approval (if PR workflow)           │
    │   - Team reviews changes                         │
    │   - Manual approval required                     │
    └──────────────────┬───────────────────────────────┘
                       │
                       ▼
    ┌──────────────────────────────────────────────────┐
    │   Terraform Apply via GitHub Actions             │
    │   - Applies plan to Azure                        │
    │   - Updates Azure Storage state file             │
    └──────────────────┬───────────────────────────────┘
                       │
                       ▼
    ┌──────────────────────────────────────────────────┐
    │   Resource provisioned in Azure                  │
    │   - State stored in Azure Storage Account        │
    └──────────────────┬───────────────────────────────┘
                       │
                       ▼
                   ◇─────────◇
                   │    END    │
                   └─────────┘
```

### Current Workflow Characteristics

| Aspect | Details |
|---|---|
| **Trigger** | Developer push or pull request to GitHub repository |
| **Validation** | Terraform fmt, validate, and plan stages |
| **Approval Process** | GitHub PR reviews and manual approval before apply |
| **Execution** | GitHub Actions runner executes Terraform apply |
| **State Management** | Stored in Azure Storage Account (templated) |
| **Integration** | Direct Git to Azure, no external CI/CD platform |
| **Scaling** | Limited concurrent runs, runner resource constraints |
| **Limitations** | No centralized workspace management, state management complexity with 24-30 workspaces |

---

## Ideal Workflow (Target State with Terraform Cloud)

```
                                    ┌─────────────┐
                                    │    START    │
                                    └──────┬──────┘
                                           │
                                           ▼
    ┌──────────────────────────────────────────────────┐
    │   User enters the details of VM in ServiceNow    │
    │                                                  │
    │  ◄─────── Reject the ticket with reason ◄──────┐
    │                                                  │
    └──────────────────┬───────────────────────────────┘
                       │
                       ▼
    ┌──────────────────────────────────────────────────┐
    │   Approval from Requestor's Manager             │
    └──────────────────┬───────────────────────────────┘
                       │
                       ▼
    ┌──────────────────────────────────────────────────┐
    │   Cloud OPS Validates the Request                │
    └──────────────────┬───────────────────────────────┘
                       │
                       ▼
                   ◇─────────◇
                  ╱           ╲
                 ╱   Valid?    ╲
                ╱               ╲
               ◇                 ◇
               │ No          Yes │
               │                 │
               └─────────┬────────┘
                         │
                         ▼
    ┌──────────────────────────────────────────────────┐
    │   Payload sent to Ansible                        │
    └──────────────────┬───────────────────────────────┘
                       │
                       ▼
    ┌──────────────────────────────────────────────────┐
    │   Validate the payload                           │
    └──────────────────┬───────────────────────────────┘
                       │
                       ▼
    ┌──────────────────────────────────────────────────┐
    │   Job Trigger in Ansible via REST API            │
    └──────────────────┬───────────────────────────────┘
                       │
                       ▼
    ┌──────────────────────────────────────────────────┐
    │   TFC API triggered from Ansible with            │
    │   the necessary payload                          │
    └──────────────────┬───────────────────────────────┘
                       │
                       ▼
                   ◇─────────────◇
                  ╱               ╲
                 ╱   Plan         ╲
                ╱   Succeeded?     ╲
               ◇                    ◇
               │ Yes            No │
               │                    │
               ├─────┬─────────────┘
               │     │
               │     ▼
               │  ┌──────────────────────────────────┐
               │  │  SNOW Task closes with comment   │
               │  └──────────────────────────────────┘
               │
               ▼
    ┌──────────────────────────────────────────────────┐
    │   Review and Approve Apply CI/Cloud OPS          │
    └──────────────────┬───────────────────────────────┘
                       │
                       ▼
    ┌──────────────────────────────────────────────────┐
    │   Resource provisioned in Azure                  │
    └──────────────────┬───────────────────────────────┘
                       │
                       ▼
                   ◇─────────────◇
                   │    STOP     │
                   └─────────────┘
```

### Ideal Workflow Characteristics (Target State)

| Aspect | Details |
|---|---|
| **Trigger** | ServiceNow ticket creation with VM specifications |
| **Validation** | Cloud OPS manual validation + Ansible payload validation |
| **Approval Process** | Manager approval + Cloud OPS approval + TFC plan review + Apply approval |
| **Execution** | Terraform Cloud API execution via Ansible orchestration |
| **State Management** | Centralized in Terraform Cloud (remote backend) |
| **Integration** | ServiceNow → Ansible → Terraform Cloud → Azure (end-to-end automation) |
| **Scaling** | Unlimited concurrent runs, enterprise-grade state locking |
| **Benefits** | Centralized workspace management, enhanced collaboration, audit trails, cost estimation |

---

## Current vs. Ideal Workflow Comparison

| Feature | Current (GitHub Actions) | Ideal (Terraform Cloud) | Gap/Benefit |
|---|---|---|---|
| **Trigger Mechanism** | Git commit/PR | ServiceNow ticket + Approval workflow | User-friendly, audit-compliant |
| **State Management** | Azure Storage (distributed) | Terraform Cloud (centralized) | Better consistency, easier collaboration |
| **Workspace Management** | Manual, dispersed | Centralized in TFC | Simplified management for 24-30 workspaces |
| **Approval Workflow** | GitHub PR review | Multi-stage (Manager → CloudOps → TFC) | Enhanced governance and compliance |
| **ServiceNow Integration** | Manual ticket tracking | Automated end-to-end integration | Reduced manual effort, improved tracking |
| **Ansible Integration** | Not currently used | Ansible orchestrates TFC API | Greater automation and flexibility |
| **Concurrency & Scaling** | GitHub runner limitations | Enterprise-grade TFC | Better performance and reliability |
| **Cost Estimation** | No built-in capability | TFC provides cost estimates | Cost visibility before deployment |
| **Audit Trail** | GitHub Actions logs | TFC audit logs + Ansible + ServiceNow | Comprehensive compliance tracking |
| **Notification & Updates** | Slack/email alerts | Automated SNOW task updates | Real-time status visibility |

---

## Migration Path: GitHub Actions → Terraform Cloud
