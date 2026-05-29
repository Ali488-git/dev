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
| **AZ Policy Repo** | [metrolinx/az-policy-repo](https://github.com/metrolinx/az-policy-repo) | In Scope | Azure Storage (Templated) | P2 | Azure policy definitions. Policy-as-Code; management group scope |
| **Azure Products IaC** | [metrolinx/Azure-products-IaC](https://github.com/metrolinx/Azure-products-IaC) | Out of Scope | Terraform Cloud | P3 | Dev workspace for the team. Products and workload-specific deployments. Already migrated to TFC |
| **Ansible Terraform Cloud Integration** | [metrolinx/Ansible-TerraformCloud-Integration](https://github.com/metrolinx/Ansible-TerraformCloud-Integration) | Out of Scope | Terraform Cloud | P3 | Ansible playbooks for automatic Terraform Cloud resource provisioning for ServiceNow tickets. Existing TFC integration already operational |

---

## Terraform Cloud

**Link**: [app.terraform.io/app/metrolinx/workspaces](https://app.terraform.io/app/metrolinx/workspaces)

**Current Setup**: Terraform Cloud is already in use with a hierarchical organization structure:
- **Organization Level**: One TFC organization (metrolinx)
- **Project Level**: Per application → Each application gets its own project for logical grouping
- **Workspace Level**: Per resource → Each resource (e.g., VMs) gets its own dedicated workspace
  - VM workspaces organized by **purpose** (web servers, database servers, etc.)
  - Each workspace maintains its own **dedicated state file**
  - State file isolation: **Per Application × Per Environment × Per Resource Type**

**Example**: For a database application in production:
- Project: `database-app`
- Workspaces: `db-server-prod`, `db-backup-prod`, `db-cache-prod` (each with separate state)

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

## What Will and Will Not Change

### Largely Unchanged
- **Terraform code structure** - Existing patterns and modularity preserved
- **GitHub repositories and PR workflow** - Source control practices remain the same
- **Azure infrastructure** - No redesign of cloud architecture
- **State boundaries** - Logical resource groupings maintained
- **Ansible usage** - Continues for OS and application configuration

### Changing
- **Terraform execution platform** - GitHub Actions → Terraform Cloud
- **State backend location** - Azure Storage → Terraform Cloud
- **Credential and secret handling** - GitHub Secrets → TFC Variables (encrypted in TFC)
- **Variable management model** - Runtime injection → Declarative TFC Variables
- **Access control and approval workflows** - GitHub PR reviews → TFC + CloudOps approvals
- **Platform operations and governance** - Centralized visibility and audit trails

**Why This Matters**: This balance ensures minimal disruption while delivering meaningful control improvements, security enhancements, and operational visibility.

---

## Current Workflow with GitHub Actions

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                       │
│                      Developer Commits Code                          │
│                                                                       │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                                                                       │
│                   GitHub Actions Triggered                           │
│              (push or pull request event)                            │
│                                                                       │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                                                                       │
│            Terraform Validate & Format Check                         │
│              • terraform fmt                                         │
│              • terraform validate                                    │
│                                                                       │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                               ▼
                        ┌──────────────┐
                        │  Validation  │
                        │ Successful?  │
                        └─┬──────────┬─┘
                   No  ┌──┘          └──┐  Yes
                       │                 │
                       ▼                 ▼
              ┌─────────────────┐  ┌──────────────────────┐
              │  Fail & Notify  │  │ Terraform Plan       │
              └─────────────────┘  └──────────┬───────────┘
                                             │
                                             ▼
                        ┌─────────────────────────────────┐
                        │   PR Review & Approval          │
                        │   (manual approval required)    │
                        └──────────────┬──────────────────┘
                                      │
                                      ▼
                        ┌─────────────────────────────────┐
                        │   Terraform Apply               │
                        │   (via GitHub Actions)          │
                        │   Updates Azure Storage State   │
                        └──────────────┬──────────────────┘
                                      │
                                      ▼
                        ┌─────────────────────────────────┐
                        │   Resource Provisioned in Azure │
                        │   State stored in Azure Storage │
                        └─────────────────────────────────┘
```

---

## Ideal Workflow (Target State with Terraform Cloud)

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                       │
│            User Creates ServiceNow Ticket                            │
│                  (VM Request Details)                                │
│                                                                       │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                                                                       │
│              Manager Approval Required                               │
│                                                                       │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                                                                       │
│              Cloud OPS Validates Request                             │
│                                                                       │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                               ▼
                        ┌──────────────┐
                        │   Request    │
                        │   Valid?     │
                        └─┬──────────┬─┘
                   No  ┌──┘          └──┐  Yes
                       │                 │
                       ▼                 ▼
              ┌─────────────────┐  ┌──────────────────────┐
              │Reject & Notify  │  │ Payload → Ansible    │
              └─────────────────┘  └──────────┬───────────┘
                                             │
                                             ▼
                        ┌─────────────────────────────────┐
                        │   Ansible Validates Payload     │
                        └──────────────┬──────────────────┘
                                      │
                                      ▼
                        ┌─────────────────────────────────┐
                        │   TFC API Triggered             │
                        │   (Terraform Plan)              │
                        └──────────────┬──────────────────┘
                                      │
                        ┌─────────────┴──────────────┐
                        │                            │
                   Plan Failed              Plan Succeeded
                        │                            │
                        ▼                            ▼
              ┌─────────────────┐  ┌──────────────────────────┐
              │ SNOW Task Close │  │ Review & Approve Apply   │
              │ (with comment)  │  │ (Cloud OPS approval)     │
              └─────────────────┘  └──────────┬───────────────┘
                                             │
                                             ▼
                        ┌─────────────────────────────────┐
                        │   Resource Provisioned in Azure │
                        │   State centralized in TFC       │
                        └─────────────────────────────────┘
```

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

> **Note**: This is a **re-platforming of Terraform operations, not a redesign of Azure infrastructure**. Existing environments, Terraform code, GitHub repositories, and existing workflows remain largely unchanged.

### Key Outcomes
- **Centralized Terraform state and execution** - Single source of truth for all infrastructure state
- **Improved security and credential management** - Enhanced secret handling and access control
- **Stronger governance, auditability, and compliance posture** - Comprehensive audit trails and approval workflows
- **Scalable foundation for future self-service and platform capabilities** - Enterprise-grade infrastructure for growth

### Risk Profile
**Low** (incremental, non-production first approach)

### In Scope
- **Azure IaC** - Primary monorepo (heavy lift)
- **AZ Policy Repo** - Lighter lift for policy definitions

### Out of Scope (Already on Terraform Cloud)
- **Azure Products IaC** - Already migrated, serves as reference pattern
- **Ansible Terraform Cloud Integration** - Already operational with existing TFC integration

### Workspace Strategy: Addressing the 60+ Workload Directories

**Challenge**: Azure-IaC contains 60+ workload directories. A naive mapping (1 directory = 1 workspace × 4 environments) would create 240+ workspaces—unmanageable at scale.

**Solution**: Workspace consolidation strategy (to be validated with HashiCorp)
- **Option A**: Group related workloads by function/team → ~24-30 workspaces per environment (more manageable)
- **Option B**: Consolidated state per environment → 4 workspaces total (simpler but less granular)
- **Option C**: Hybrid approach → Balance granularity with manageability

**Current TFC Pattern (Reference)**: Azure Products IaC already uses the hierarchical model:
- Per application → Per environment → Per resource type (with dedicated state files)
- This pattern will inform workspace consolidation decisions for Azure IaC

**HashiCorp will help determine optimal workspace structure** during Phase 0 to ensure scalability without operational overhead.

### Migration Approach

The migration will be delivered in **controlled phases** to minimize risk, with targeted involvement from **HashiCorp** during the early stages to ensure alignment with current HashiCorp Validated Designs and best practices.

- **HashiCorp engagement**: Primarily during Phase 0 and Phase 1
- **Provides**: Validation, guidance, and initial enablement
- **Reduces**: Architectural risk while ensuring internal ownership and knowledge transfer

---

## Migration Phases & Execution Plan

### Phase 0 – Discovery & Validation (Weeks 1-3)
**Objective**: Map current infrastructure and validate migration strategy

**Internal Activities:**
- Inventory all Terraform states and repositories
- Identify current backends, variables, and secrets
- Determine optimal workspace mapping for 60+ workload directories
- Map each existing state to a future Terraform Cloud workspace
- Review existing TFC patterns from Azure Products IaC for consistency

**HashiCorp Responsibilities:**
- Review current Terraform execution model and backend usage
- Validate state architecture and proposed workspace mapping strategy
- Recommend optimal workspace consolidation approach
- Identify gaps or changes in recommended practices
- Compare Azure Products IaC TFC structure with Azure IaC requirements

**Outcome**: Clear inventory, validated migration roadmap, and workspace strategy defined

---

### Phase 1 – Foundation & Initial Migration (Weeks 4-9)
**Objective**: Establish Terraform Cloud platform and execute initial state migrations

**Internal Activities:**
- Configure TFC organization, teams, and permissions
- Create Terraform Cloud control repository for platform configuration as code
- Migrate existing Terraform state using Terraform CLI (one-time operation)
- Execute dry-runs for non-production workspaces
- Validate clean plans and expected behavior post-migration
- Document migration procedures and troubleshooting guides
- Ensure consistency with existing Azure Products IaC TFC structure

**HashiCorp Responsibilities:**
- Assist with TFC organization setup
- Configure or validate:
  - SSO integration
  - Project, team, and RBAC models
  - VCS connections and execution modes
- Support creation and review of Terraform Cloud control repository
- Provide guided oversight during initial state migrations
- Review dry-run results and approve go-live procedures

**Outcome**: Terraform Cloud platform operational; first state migrations validated; runbook documented

---

### Phase 2 – Pilot and Scale (Weeks 10-15)
**Objective**: Establish repeatable migration process and expand across environments

**Activities:**
- Migrate low-risk non-production workspace first (Dev environment)
- Document repeatable migration runbook and lessons learned
- Migrate SIT environment
- Migrate UAT environment
- Validate approval workflows work end-to-end
- **Production environments migrated last (Phase 3)**

**Outcome**: Standardized migration process proven; most non-prod environments in TFC

---

### Phase 3 – Production Migration (Weeks 16-21)
**Objective**: Migrate production environments with highest care and contingency

**Activities:**
- Conduct comprehensive backup of all production states
- Execute full production dry-run with rollback plan ready
- Migrate production workspaces in waves
- Maintain GitHub Actions as active fallback during cutover
- Comprehensive monitoring and validation post-cutover
- Gradual traffic cutover with instant rollback capability

**Outcome**: Production migrated with zero downtime; rollback tested and proven

---

### Phase 4 – Stabilization (Weeks 22-26)
**Objective**: Clean up legacy systems and formalize operations

**Activities:**
- Retire legacy GitHub Actions Terraform logic
- Remove old backend references and Azure Storage dependencies
- Normalize variables, naming, and access controls
- Formalize operating and support model
- Complete team training and knowledge transfer
- Consolidate with existing Azure Products IaC team practices

**Outcome**: Single source of truth (TFC); legacy systems retired; team self-sufficient

---

### Phase 5 – Optimization (Weeks 27+, Optional)
**Objective**: Enable advanced capabilities for future growth

**Potential Enhancements:**
- Private module registry for standardized components
- Dynamic Azure credentials with SSO
- Policy-as-code and automated approvals
- Drift detection and self-service provisioning
- Reusable workspace templates based on proven patterns

**Outcome**: Platform ready for self-service and advanced automation

---

## Timeline with Milestones

| Milestone | Target Week | Status |
|---|---|---|
| **Phase 0 Complete** - Workspace strategy defined | Week 3 | Discovery gate |
| **Phase 1 Complete** - TFC operational, dry-runs successful | Week 9 | Go-live approval gate |
| **Dev Environment Live** - First non-prod in production | Week 11 | Confidence checkpoint |
| **SIT + UAT Complete** - All non-prod migrated | Week 15 | Production readiness gate |
| **Production Live** - All environments migrated | Week 21 | Mission-critical gate |
| **Legacy Cleanup Complete** - Azure Storage backend retired | Week 26 | Completion gate |
| **Total Duration** | **~6 months** | On track |

---

## Effort Justification & Estimation

### Why This Investment?
1. **Operational Risk Reduction** - Centralized state eliminates fragmentation issues
2. **Security Posture** - Encrypted secrets management and audit trails
3. **Scalability** - Enterprise-grade platform for growth
4. **Compliance** - Better governance and auditability
5. **Team Efficiency** - Reduced manual ticket tracking and approvals

### Effort Breakdown

| Phase | Activity | Effort | Owner |
|---|---|---|---|
| **Phase 0** | Discovery & architecture validation | 80 hours | Internal + HashiCorp |
| **Phase 1** | TFC setup, control repo, dry-runs | 160 hours | Internal + HashiCorp |
| **Phase 2** | Non-prod migrations (3 environments) | 120 hours | Internal |
| **Phase 3** | Production migration + validation | 100 hours | Internal |
| **Phase 4** | Cleanup, training, documentation | 80 hours | Internal |
| **Phase 5** | Optimization (optional) | 40 hours | Internal |
| **Total** | **~580-600 hours over 6 months** | **~1.5 FTE** | Team + Partner |

**Cost-Benefit**: 600 hours upfront investment eliminates ongoing operational complexity, reduces incident response time (hours → minutes), and enables future self-service platform capabilities.

---

## Rollback / Contingency Plan

### Why Rollback Capability Matters
If critical issues arise during migration, we need the ability to quickly revert to GitHub Actions without data loss or extended downtime.

### Rollback Strategy

**Pre-Migration Safeguards**:
1. **Full State Backup** - All Azure Storage states backed up before migration
2. **TFC State Export** - Export all TFC states as backups
3. **Parallel Running** - GitHub Actions workflow remains operational during transition
4. **Dry-Run Validation** - Each workspace migrated in dry-run mode before cutover

**Rollback Procedures by Phase**:

| Phase | Rollback Trigger | Time to Rollback | Procedure |
|---|---|---|---|
| **Phase 1** | TFC setup issues | Immediate (same day) | Stop using TFC; resume GitHub Actions |
| **Phase 2** | Non-prod failures | Within 1 hour | Restore state from backup; resume GitHub Actions |
| **Phase 3** | Production issues | Within 15 minutes | Execute instant failover to GitHub Actions; investigate offline |
| **Post-Migration** | Critical bug discovered | Possible (manual process) | Restore specific workspace state from backup |

**Rollback Execution**:
- GitHub Actions workflows remain in version control and ready to activate
- Terraform state files retained in Azure Storage for 30 days post-migration
- Clear procedures documented for each service team
- On-call escalation plan for production incidents

**Data Loss Prevention**:
- All state migrations are additive (never destructive)
- Backup retention: 90 days minimum
- Version control backups: All TFC state exports retained in Git history

### Contingency Scenarios

| Scenario | Response | Prevention |
|---|---|---|
| **TFC authentication fails** | Revert to GitHub Actions immediately | SSO testing in Phase 1 |
| **State migration corruption** | Restore from backup; retry migration | Dry-run validation per workspace |
| **Approval workflow bottleneck** | Bypass TFC temporarily; use GitHub | User training and workflow tuning |
| **Variable management issues** | Revert to env vars in GitHub Actions | Comprehensive variable audit in Phase 0 |
| **Production performance degradation** | Instant failover; investigate offline | Load testing in Phase 2 |

---

## Terraform State Migration Principle

For each Terraform state, follow this consistent process:

1. **Create** a corresponding Terraform Cloud workspace
2. **Associate** it with the correct code path (Git repo and directory)
3. **Validate** workspace configuration (variables, integrations, permissions)
4. **Dry-run** migration with full test plan
5. **Migrate** state using Terraform CLI (one-time activity)
6. **Validate** with a clean plan to ensure no unexpected changes
7. **Cutover** with rollback plan ready
8. **Verify** post-migration for 7 days before declaring success

> **Critical**: After migration, Terraform Cloud becomes the authoritative source for execution and state. Azure Storage backups retained for contingency only.

---

## Risk Mitigation & Controls

| Risk | Control / Mitigation |
|---|---|
| **Incorrect state mapping** | Strict inventory and validation of all state files before migration |
| **Variable mismatches** | Full documentation of runtime inputs and variables across all environments |
| **Over-permissive access** | SSO-based least privilege access model with team-based RBAC |
| **Environment inconsistency** | Enforced naming standards and workspace configuration standards |
| **Over-engineering early phases** | Keep early phases minimal; focus on core functionality before optimization |
| **Production downtime** | Parallel running capability; instant rollback to GitHub Actions |
| **State corruption** | Comprehensive backup strategy; dry-run validation before cutover |
| **User adoption resistance** | Training program; clear documentation; support on-call team |

---

## Summary Statement

This migration addresses a known and growing operational risk around Terraform state consistency while modernizing governance, security, and auditability. By using a phased, low-risk approach with targeted HashiCorp involvement and comprehensive contingency planning, Metrolinx can adopt Terraform Cloud as a stable enterprise platform without disrupting day-to-day engineering workflows. The investment of ~600 hours over 6 months delivers immediate operational improvements and enables future self-service capabilities. The existing TFC patterns from Azure Products IaC and Ansible Terraform Cloud Integration serve as proven reference implementations for this migration.

---

## Questions?
