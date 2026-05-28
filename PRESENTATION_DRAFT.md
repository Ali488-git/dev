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

## Terraform Cloud Migration Initiative

### Overview
Metrolinx is planning a phased migration of Terraform execution from GitHub Actions to HCP Terraform (Terraform Cloud). The goal is to establish Terraform Cloud as the single, authoritative platform for Terraform execution, state management, approvals, and auditability.

> **Note**: This is a **re-platforming of Terraform operations, not a redesign of Azure infrastructure**. Existing environments, Terraform code, GitHub repositories, and Ansible workflows remain largely unchanged.

### Key Outcomes
- **Centralized Terraform state and execution** - Single source of truth for all infrastructure state
- **Improved security and credential management** - Enhanced secret handling and access control
- **Stronger governance, auditability, and compliance posture** - Comprehensive audit trails and approval workflows
- **Scalable foundation for future self-service and platform capabilities** - Enterprise-grade infrastructure for growth

### Risk Profile
**Low** (incremental, non-production first approach)

### In Scope
- **Azure IaC** - Primary monorepo (heavy lift)
- **Azure Products IaC** - Experiencing split state issues (primary pain point)
- **AZ Policy Repo** - Lighter lift for policy definitions
- **Ansible–TerraformCloud-Integration** - Already using TFC, needs standardization

### The Core Problem: Split Terraform State

A primary driver for this migration is the current **split state model**:

- Resources managed via Terraform Cloud store state in Terraform Cloud
- Pre-existing or legacy resources retain state in Azure Storage
- This creates **partial state ownership** where different attributes of the same resource are tracked in different backends

**Consequences of split state:**
- Increased configuration drift
- Inconsistent execution behavior
- Harder troubleshooting and audits
- Long-term operational risk

**Migration objective**: Move all active Terraform state into Terraform Cloud so each workspace becomes the single system of record, eliminating partial state ownership.

### Migration Approach

The migration will be delivered in **controlled phases** to minimize risk, with targeted involvement from **HashiCorp** during the early stages to ensure alignment with current HashiCorp Validated Designs and best practices.

- **HashiCorp engagement**: Primarily during Phase 0 and Phase 1
- **Provides**: Validation, guidance, and initial enablement
- **Reduces**: Architectural risk while ensuring internal ownership and knowledge transfer

---

## Migration Phases & Execution Plan

### Phase 0 – Discovery & Validation
**Objective**: Map current infrastructure and validate migration strategy

**Internal Activities:**
- Inventory all Terraform states and repositories
- Identify current backends, variables, and secrets
- Map each existing state to a future Terraform Cloud workspace

**HashiCorp Responsibilities:**
- Review current Terraform execution model and backend usage
- Validate state architecture and proposed workspace mapping strategy
- Identify gaps or changes in recommended practices

**Outcome**: Clear inventory and validated migration roadmap

---

### Phase 1 – Foundation & Initial Migration
**Objective**: Establish Terraform Cloud platform and execute initial state migrations

**Internal Activities:**
- Configure TFC organization, teams, and permissions
- Create Terraform Cloud control repository for platform configuration as code
- Migrate existing Terraform state using Terraform CLI (one-time operation)
- Validate clean plans and expected behavior post-migration

**HashiCorp Responsibilities:**
- Assist with TFC organization setup
- Configure or validate:
  - SSO integration
  - Project, team, and RBAC models
  - VCS connections and execution modes
- Support creation and review of Terraform Cloud control repository
- Provide guided oversight during initial state migrations

**Outcome**: Terraform Cloud platform operational; first state migrations validated

---

### Phase 2 – Pilot and Scale
**Objective**: Establish repeatable migration process and expand across environments

**Activities:**
- Migrate low-risk non-production workspace first
- Document repeatable migration runbook
- Migrate remaining environments in risk-based waves
- **Production environments migrated last**

**Outcome**: Standardized migration process; most non-prod environments in TFC

---

### Phase 3 – Stabilization
**Objective**: Clean up legacy systems and formalize operations

**Activities:**
- Retire legacy GitHub Actions Terraform logic
- Remove old backend references and Azure Storage dependencies
- Normalize variables, naming, and access controls
- Formalize operating and support model

**Outcome**: Single source of truth (TFC); legacy systems retired

---

### Phase 4 – Optimization (Optional, Future)
**Objective**: Enable advanced capabilities for future growth

**Potential Enhancements:**
- Private module registry for standardized components
- Dynamic Azure credentials with SSO
- Policy-as-code and automated approvals
- Drift detection and self-service provisioning

**Outcome**: Platform ready for self-service and advanced automation

---

## Terraform State Migration Principle

For each Terraform state, follow this consistent process:

1. **Create** a corresponding Terraform Cloud workspace
2. **Associate** it with the correct code path (Git repo and directory)
3. **Migrate** state using Terraform CLI (one-time activity)
4. **Validate** with a clean plan to ensure no unexpected changes

> **Critical**: After migration, Terraform Cloud becomes the authoritative source for execution and state. There is no fallback to distributed backends.

---

## Risk Mitigation & Controls

| Risk | Control / Mitigation |
|---|---|
| **Incorrect state mapping** | Strict inventory and validation of all state files before migration |
| **Variable mismatches** | Full documentation of runtime inputs and variables across all environments |
| **Over-permissive access** | SSO-based least privilege access model with team-based RBAC |
| **Environment inconsistency** | Enforced naming standards and workspace configuration standards |
| **Over-engineering early phases** | Keep early phases minimal; focus on core functionality before optimization |

---

## Timeline and Next Steps

### Timeline
- **Foundation & migration**: 8–12 weeks
- **Optimization and maturity**: 12–16 weeks
- **Total duration**: ~5–7 months

### Next Steps
1. **Approve** this migration plan
2. **Engage** HashiCorp for guidance and validation
3. **Schedule** migration kickoff meeting
4. **Establish** monthly checkpoints for progress tracking and risk review

---

## Summary Statement

This migration addresses a known and growing operational risk around Terraform state consistency while modernizing governance, security, and auditability. By using a phased, low-risk approach with targeted HashiCorp involvement, Metrolinx can adopt Terraform Cloud as a stable enterprise platform without disrupting day-to-day engineering workflows.

---

## Questions?
