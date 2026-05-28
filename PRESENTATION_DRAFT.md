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

## Ideal Workflow for Infrastructure Provisioning

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

### Workflow Stages

| Stage | Description |
|---|---|
| **ServiceNow** | User initiates VM provisioning request with detailed specifications |
| **Ansible** | Validates payload and triggers Terraform Cloud API via REST |
| **Terraform Cloud (TFC)** | Executes plan and applies infrastructure changes |
| **Azure** | Resources are provisioned and configured in the target subscription |

### Key Features

- **Approval Gates**: Manager and Cloud OPS validation ensure compliance
- **Automated Validation**: Ansible validates payloads before TFC execution
- **ServiceNow Integration**: Automated task updates with plan results
- **Approval Workflow**: TFC plan approval required before resource deployment
- **Audit Trail**: Full visibility across all stages from request to deployment
