# Speaker Notes - Cloud Infrastructure & Terraform Cloud Migration Overview

## Opening

Before we dive into the details of our migration plan, I want to assume this is our first meeting together and start from scratch to give you a high-level overview of our current cloud infrastructure.

---

## Current State Overview

### GitHub Repositories

Below are our GitHub repositories which mostly have their state management done through Azure storage blobs, with a few exceptions. All of these are in scope for migration to Terraform Cloud.

We do have our TFC organization setup for some repos, but it's not fully integrated yet. We need to consolidate and standardize the approach across all repositories.

---

## Repository Deep Dive

### Azure-IaC Repository
This is our biggest repository—the heavy lift, so to speak. It's structured as a monorepo with:

- **Environment-based workspaces** - Separate workspaces for Dev, SIT, UAT, and Prod environments
- **Infrastructure Foundation** - Core networking, security, and shared services
- **Modules** - Reusable Terraform modules for common infrastructure patterns
- **Workload-Specific Modules** - Specialized modules for different application types

One important thing to note: **Backend configuration is injected at runtime through GitHub Actions and is not version-controlled**. This is something we'll need to address as part of the migration.

**Architectural Question for HashiCorp**: Should workload directories map to one workspace per directory, one workspace per workload per environment, or consolidated workspaces by application team?

### Azure-Products-IaC Repository
This repository follows a **Product and workload deployment pattern** structure. 

Current challenge: We have a **split state issue** where state management is fragmented across different backends. However, there's a silver lining—this repo already has an existing Terraform Cloud implementation. We can use it as a migration reference pattern for other repositories.

### az-policy-repo
This one is relatively straightforward. It contains:

- **Policy definitions** - Azure Policy policy definitions
- **Policy assignments** - How those policies are assigned to resources and management groups

This is one of the lighter lifts in our migration scope.

### Ansible-Terraform-Integration Repository
This repository contains Ansible playbooks for infrastructure orchestration and integration with Terraform. It will be the critical integration point for triggering Terraform Cloud runs and orchestrating our target state workflow.

---

## Azure Subscription Mapping

These are our core Azure subscriptions. Each GitHub repository uses a combination of these subscriptions depending on which environments (Dev, SIT, UAT, Prod, DMZ, Sandbox) the Terraform code is deployed into.

This mapping ensures:
- **Consistent backend configuration** across all deployments
- **Cross-subscription references** - Resources in one subscription can reference resources in another
- **Proper separation of workloads** - Dev, test, and production workloads are isolated in their own subscriptions

The subscriptions span across:
- **Shared Infrastructure** (Connectivity, Management)
- **Environment-Specific** (Dev, SIT, UAT, Prod - each with core and DMZ variants)
- **Sandbox** (For testing and experimentation)

**Multi-Subscription Complexity**: We're managing 8+ subscriptions with dynamic lookup patterns and cross-subscription data source dependencies. Our Service Principal must have read access across all subscriptions. This adds complexity to our workspace mapping strategy that HashiCorp will help us navigate.

---

## Key Takeaways for Current State

1. **State Fragmentation** - Currently managed across Azure Storage with manual configuration injection via CI/CD
2. **Partial TFC Adoption** - Some repos already using Terraform Cloud, which gives us a reference pattern
3. **Monorepo Challenge** - Azure-IaC's size and complexity will be the main migration focus
4. **Policy Management** - Separate concern that needs specific handling
5. **Automation Ready** - Ansible-Terraform-Integration provides the foundation for our target state

---

## Terraform Cloud Migration Initiative - Detailed Speaker Notes

### Strategic Context

Metrolinx is planning a phased migration of Terraform execution from GitHub Actions to HCP Terraform (Terraform Cloud). This is fundamentally a **re-platforming initiative**, not an infrastructure redesign.

**Key Point to Emphasize**: This is a low-risk initiative because we're taking an incremental approach with production environments coming last. We're not redesigning Azure infrastructure or changing how our Terraform code is structured—we're simply centralizing execution and state management.

### What's Changing vs. What's Not

**What's Staying the Same:**
- Azure infrastructure design and patterns
- Terraform code structure and modules
- GitHub repositories as source control
- Ansible workflows (they'll be enhanced, but fundamentally similar)
- Environment separation (Dev, SIT, UAT, Prod)

**What's Changing:**
- Terraform *execution* location (GitHub Actions → Terraform Cloud)
- State *storage* location (Azure Storage → Terraform Cloud)
- Approval *workflows* (GitHub PR reviews → TFC + ServiceNow + Manager approvals)
- Credential *injection* (GitHub Secrets → TFC Variables)
- Central *visibility* (adding TFC audit logs and dashboards)

### The Core Problem We're Solving: Split State

This is the primary driver for our migration. Let me explain why it matters.

**Current Situation - State Consolidation Challenge:**
Some of our infrastructure resources are being managed by Terraform Cloud (when teams used it), while other resources—either pre-existing or legacy—remain tracked in Azure Storage state files. This creates a **split state ownership model** across 3 repositories currently using Azure Storage and 1 repository already on TFC.

**What This Looks Like:**
- Resource A might have its compute config tracked in TFC but its network config tracked in Azure Storage
- When we run Terraform, different attributes of the same resource come from different backends
- Over time, resources become harder to understand because their state is scattered

**The Downstream Impact:**
- **Configuration drift** - We lose track of what's actually deployed vs. what Terraform thinks is deployed
- **Inconsistent behavior** - Terraform operations sometimes work, sometimes don't, depending on which state file is authoritative
- **Audit nightmares** - When something breaks, we have to trace across multiple backends to understand what happened
- **Long-term risk** - As the infrastructure grows, this fragmentation becomes increasingly unmanageable

**Our Solution:**
Make Terraform Cloud the *single system of record* for all Terraform-managed infrastructure. Every workspace becomes the authoritative source for its state, eliminating partial ownership.

### Key Outcomes We're After

1. **Centralized Terraform state and execution** 
   - One place to see what's deployed, who changed it, and when
   - Easier troubleshooting and disaster recovery

2. **Improved security and credential management** 
   - Secrets live in Terraform Cloud, not in GitHub CI/CD secrets
   - Audit trail of who accessed what credentials

3. **Stronger governance, auditability, and compliance posture** 
   - TFC provides comprehensive audit logs
   - Approval gates can be enforced at the TFC level
   - Compliance teams get visibility into all infrastructure changes

4. **Scalable foundation for future self-service and platform capabilities** 
   - Once TFC is in place, we can build self-service provisioning
   - Teams can request resources through ServiceNow, which feeds into TFC
   - This enables the ServiceNow → Ansible → TFC workflow we discussed earlier

### Risk Profile: Why This Is Low-Risk

We're being intentional about minimizing risk:
- **Incremental approach** - We're doing this in phases, not all at once
- **Non-production first** - Development and testing environments go first
- **Reference patterns** - We already have some TFC implementations we can learn from
- **HashiCorp guidance** - HashiCorp will validate our approach in early phases
- **Parallel running** - We can run both GitHub Actions and TFC side-by-side during transition
- **Rollback capability** - We'll maintain Azure Storage as rollback backup during migration
- **Comprehensive backup strategy** - Dry-run procedures for each workspace before cutover

This is not a "big bang" cutover. It's a controlled, staged migration.

### Why HashiCorp Involvement Is Necessary

Terraform Cloud capabilities and recommended practices have evolved since our initial adoption. Our current adoption within the organization is partial and inconsistent. HashiCorp involvement helps reduce:

- **Architectural rework** - Avoiding costly redesigns mid-migration
- **Incorrect state or workspace mapping** - Critical decisions that are expensive to unwind
- **Early governance mistakes** - Setting RBAC, variable management, and approval workflows correctly from the start

### Engagement Model

**Short-term advisory and enablement** focused on Phase 0 and Phase 1 to:
- Validate our architecture against HashiCorp Validated Designs
- Provide guidance on workspace organization and team structure
- Review our backend configuration and secrets management approach
- Help establish best practices for state migration
- Identify and mitigate risks early

**No long-term dependency** on HashiCorp—strong emphasis on knowledge transfer to internal teams.

**Our Role (Throughout):**
- Own the implementation and day-to-day execution
- Build institutional knowledge so we don't depend on external advisors long-term
- Manage the actual state migration process
- Handle repository and workflow updates
- Drive team adoption and training

### In Scope: What We're Migrating

1. **Azure IaC** - Our primary monorepo (the heavy lift)
2. **Azure Products IaC** - Where we already have split state (this is the pain point we're solving)
3. **AZ Policy Repo** - Lighter lift for policy definitions
4. **Ansible–Terraform-Integration** - Ansible orchestration as critical integration point

**Note on scope**: We're being deliberate about *limiting* scope to reduce unnecessary work. We're not trying to migrate everything at once; we're focusing on these repositories where the value is highest and the risk is manageable.

### Key Discovery Gaps That HashiCorp Will Help Address

**Architectural Questions:**
- Should workload directories map to one workspace per directory, one workspace per workload per environment, or consolidated workspaces by application team?
- Should Terraform Cloud workspaces use multiple provider aliases?
- Should subscriptions be managed through workspace variables?
- Should states consolidate by environment or by function?

**Operational Questions:**
- Should we standardize provider versions or allow per-workspace flexibility?
- How should we manage variables—using Terraform Cloud Variables, HCL, or tfvars?
- What's the timeline for retiring Azure Storage backends?

**Integration Questions:**
- How should Ansible trigger Terraform Cloud runs?
- How should API tokens be managed securely?
- How should ServiceNow approvals integrate with Terraform Cloud?
- Should GitHub Actions workflows be deprecated or retained for non-Terraform tasks?

### Template Configuration Considerations

Our current backend configuration uses runtime variable injection through CI/CD. We need to validate the TFC integration approach and determine how workspace variables will be managed across our multi-subscription environment.

---

## The "Big Picture" Migration Flow

1. **Phase 0 – Discovery & Validation** (3 weeks)
   - HashiCorp engagement begins
   - Inventory all Terraform states and repositories
   - Validate our TFC organization structure
   - Define workspace naming conventions and team RBAC
   - Plan state migration strategy addressing architectural questions

2. **Phase 1 – Foundation & Initial Migration** (4–6 weeks)
   - Configure TFC organization, teams, and permissions
   - Create Terraform Cloud control repository for platform configuration as code
   - Migrate existing Terraform state using Terraform CLI (one-time operation)
   - Pilot with low-risk workspace first (Azure-Products-IaC as reference)
   - Test the GitHub → TFC integration with Ansible
   - Validate clean plans and expected behavior post-migration
   - Refine processes based on learnings

3. **Phase 2 – Expand to Test Environments** (4–6 weeks)
   - Scale up the migration process
   - SIT and UAT environments migrated
   - Validate approval workflows work end-to-end with ServiceNow integration
   - Document repeatable migration runbook

4. **Phase 3 – Production Migration** (4–6 weeks)
   - Move to production with highest care
   - Maintain GitHub Actions as fallback during cutover
   - Gradual migration with monitoring
   - Split state is eliminated

5. **Phase 4 – Cleanup and Optimization** (Ongoing)
   - Retire GitHub Actions Terraform workflows
   - Optimize TFC configuration
   - Establish ongoing governance and best practices
   - Enable self-service capabilities in future

### Expected Outcomes by Phase

- **After Phase 1**: We understand the migration process, have validated architecture with HashiCorp, and proven the approach with low-risk workspace
- **After Phase 2**: We're confident in the approach; most non-prod state is unified; repeatable process documented
- **After Phase 3**: Production is migrated; split state is eliminated; single system of record established
- **After Phase 4**: Terraform Cloud is fully optimized; team is self-sufficient; self-service capabilities can begin

---

## Migration Timeline

- **Phase 0 (Discovery)**: 3 weeks
- **Phase 1 (Foundation + Pilot)**: 4–6 weeks
- **Phase 2–3 (Scale & Production)**: 8–12 weeks
- **Phase 4 (Optimization)**: Ongoing
- **Total duration**: ~5–7 months

---

## Risk Mitigation Strategy

We're implementing comprehensive safeguards:
- **Comprehensive backup strategy** - All Azure Storage state files retained during migration
- **Dry-run procedures** - Test each workspace migration before cutover
- **Pilot-first approach** - Start with low-risk workspace (Azure-Products-IaC) to refine process
- **Parallel running** - Maintain both systems during transition period
- **Clear rollback plan** - Ability to revert to Azure Storage if critical issues arise

---

## Key Discussion Points for Stakeholders

**State Consolidation Challenge:**
- Currently split across multiple Azure Storage containers and TFC
- Need clear mapping strategy to avoid conflicts and data loss
- Reference implementation (Azure-Products-IaC) provides proven pattern

**Multi-Subscription Complexity:**
- 8+ subscriptions with dynamic lookup patterns
- Cross-subscription data source dependencies critical
- Service Principal access strategy must be validated

**Template Configuration:**
- Backend config uses runtime variable injection
- Need to validate TFC CI/CD integration approach
- Variables management strategy must scale across environments

**Migration Scope:**
- 3 repos moving from Azure Storage → TFC (high value, high risk)
- 1 repo already on TFC (reference implementation)
- Ansible orchestration as critical integration point

---

## Next Steps

1. HashiCorp engagement to address discovery gaps and validate architecture
2. Detailed review of each repository's current state and dependencies
3. Workspace mapping strategy aligned with team structure
4. Phase 0 kickoff with inventory and validation activities
5. Monthly checkpoints with stakeholders for progress tracking and risk review

The HashiCorp partnership will ensure we're aligned with industry best practices, and we'll maintain clear communication with all stakeholders throughout each phase. This structured approach minimizes risk while delivering the operational improvements we need to scale our infrastructure platform.
