# Speaker Notes - Cloud Infrastructure & Terraform Cloud Migration Overview

## Opening

Before we dive into the details of our migration plan, I want to assume this is our first meeting together and start from scratch to give you a high-level overview of our current cloud infrastructure.

---

## Current State Overview

### GitHub Repositories

Below are our GitHub repositories which mostly have their state management done through Azure storage blobs, with a few exceptions. All of these are in scope for migration to Terraform Cloud.

We do have our TFC organization setup for some repos, but it's not fully integrated yet. For a detailed overview of that, we're unfortunately missing John and Abbas today, but we can cover the high-level approach.

---

## Repository Deep Dive

### Azure-IaC Repository
This is our biggest repository—the heavy lift, so to speak. It's structured as a monorepo with:

- **Environment-based workspaces** - Separate workspaces for Dev, SIT, UAT, and Prod environments
- **Infrastructure Foundation** - Core networking, security, and shared services
- **Modules** - Reusable Terraform modules for common infrastructure patterns
- **Workload-Specific Modules** - Specialized modules for different application types

One important thing to note: **Backend configuration is injected at runtime through GitHub Actions and is not version-controlled**. This is something we'll need to address as part of the migration.

### Azure-Products-IaC Repository
This repository follows a **Product and workload deployment pattern** structure. 

Current challenge: We have a **split state issue** where state management is fragmented across different backends. However, there's a silver lining—this repo already has an existing Terraform Cloud implementation. We can use it as a migration reference pattern for other repositories.

### az-policy-repo
This one is relatively straightforward. It contains:

- **Policy definitions** - Azure Policy policy definitions
- **Policy assignments** - How those policies are assigned to resources and management groups

This is one of the lighter lifts in our migration scope.

### Ansible-Terraform-Integration & Ansible-TerraformCloud-Integration Repositories
The Ansible-TerraformCloud-Integration repository is particularly interesting because it's already using Terraform Cloud. It has:

- **Existing Terraform Cloud integration** - Already set up and partially operational
- **Workspace configuration** - TFC workspaces are configured
- **Requires inventory of existing workspaces** - We need to document what's already there

Ruban can shed more light on this if we need deeper details on the current Ansible-TFC setup.

---

## Azure Subscription Mapping

These are our core Azure subscriptions. Each GitHub repository uses a combination of these subscriptions depending on which environments (Dev, SIT, UAT, Prod, DMZ, Sandbox) the Terraform code is deploying into.

This mapping ensures:
- **Consistent backend configuration** across all deployments
- **Cross-subscription references** - Resources in one subscription can reference resources in another
- **Proper separation of workloads** - Dev, test, and production workloads are isolated in their own subscriptions

The subscriptions span across:
- **Shared Infrastructure** (Connectivity, Management)
- **Environment-Specific** (Dev, SIT, UAT, Prod - each with core and DMZ variants)
- **Sandbox** (For testing and experimentation)

---

## Key Takeaways for Current State

1. **State Fragmentation** - Currently managed across Azure Storage with manual configuration injection
2. **Partial TFC Adoption** - Some repos already using Terraform Cloud, which gives us a reference pattern
3. **Monorepo Challenge** - Azure-IaC's size and complexity will be the main migration focus
4. **Policy Management** - Separate concern that needs specific handling
5. **Automation Ready** - Ansible-TFC integration provides the foundation for our target state

---

## Terraform Cloud Migration Initiative - Detailed Speaker Notes

### Strategic Context

Metrolinx is planning a phased migration of Terraform execution from GitHub Actions to HCP Terraform (Terraform Cloud). This is fundamentally a **re-platforming initiative**, not an infrastructure redesign. We're moving the *how* we execute Terraform, not the *what* we're deploying.

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

**Current Situation:**
Some of our infrastructure resources are being managed by Terraform Cloud (when teams used it), while other resources—either pre-existing or legacy—remain tracked in Azure Storage state files. This creates a **split state ownership model**.

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
- **Partner support** - A HashiCorp partner will validate our approach in early phases
- **Parallel running** - We can run both GitHub Actions and TFC side-by-side during transition
- **Rollback capability** - If something goes wrong, we can revert to GitHub Actions

This is not a "big bang" cutover. It's a controlled, staged migration.

### In Scope: What We're Migrating

1. **Azure IaC** - Our primary monorepo (the heavy lift)
2. **Azure Products IaC** - Where we already have split state (this is the pain point we're solving)
3. **AZ Policy Repo** - Lighter lift for policy definitions
4. **Ansible-TerraformCloud-Integration** - Already using TFC, but needs standardization

**Note on scope**: We're being deliberate about *limiting* scope to reduce unnecessary work. We're not trying to migrate everything at once; we're focusing on these repositories where the value is highest and the risk is manageable.

### Migration Approach: How We'll Execute This

We're partnering with a certified HashiCorp partner to guide the process. Here's the engagement model:

**Partner's Role (Primarily Phases 0-1):**
- Validate our architecture against HashiCorp Validated Designs
- Provide guidance on workspace organization and team structure
- Review our backend configuration and secrets management approach
- Help us establish best practices for state migration
- Identify and mitigate risks early

**Our Role (Throughout):**
- Own the implementation and day-to-day execution
- Build institutional knowledge so we don't depend on external partners long-term
- Manage the actual state migration process
- Handle repository and workflow updates
- Drive team adoption and training

**Why This Model Works:**
- Reduces architectural risk by validating with experts upfront
- Ensures we're following industry best practices
- Transfers knowledge to our team so we're self-sufficient afterward
- Keeps costs manageable (partner involvement is focused, not ongoing)

### The "Big Picture" Migration Flow

1. **Phase 0** - Setup and validation
   - HashiCorp partner engages
   - We validate our TFC organization structure
   - Define workspace naming conventions and team RBAC
   - Plan state migration strategy

2. **Phase 1** - Pilot migration (non-production)
   - Start with dev environments
   - Migrate Azure Products IaC as reference
   - Test the GitHub → TFC integration
   - Refine processes based on learnings

3. **Phase 2** - Expand to test environments
   - SIT and UAT environments
   - Scale up the migration process
   - Validate approval workflows work end-to-end

4. **Phase 3** - Production migration
   - Move to prod with highest care
   - Maintain GitHub Actions as fallback
   - Gradual cutover with monitoring

5. **Phase 4** - Cleanup and optimization
   - Retire GitHub Actions Terraform workflows
   - Optimize TFC configuration
   - Establish ongoing governance and best practices

### Expected Outcomes by Phase

- **After Phase 1**: We understand the migration process and have a reference pattern
- **After Phase 2**: We're confident in the approach; most non-prod state is unified
- **After Phase 3**: Production is migrated; split state is eliminated
- **After Phase 4**: Terraform Cloud is fully optimized; self-service capabilities can begin

---

## Next Steps

We'll walk through each repository in detail, discuss the current state vs. ideal state workflows, and outline the migration strategy to consolidate state management in Terraform Cloud while maintaining governance and approval workflows.

The partner will help us ensure we're aligned with HashiCorp best practices, and we'll maintain clear communication with all stakeholders throughout each phase.
