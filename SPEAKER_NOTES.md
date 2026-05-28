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

## Key Takeaways for Migration

1. **State Fragmentation** - Currently managed across Azure Storage with manual configuration injection
2. **Partial TFC Adoption** - Some repos already using Terraform Cloud, which gives us a reference pattern
3. **Monorepo Challenge** - Azure-IaC's size and complexity will be the main migration focus
4. **Policy Management** - Separate concern that needs specific handling
5. **Automation Ready** - Ansible-TFC integration provides the foundation for our target state

---

## Next Steps

We'll walk through each repository in detail, discuss the current state vs. ideal state workflows, and outline the migration strategy to consolidate state management in Terraform Cloud while maintaining governance and approval workflows.
