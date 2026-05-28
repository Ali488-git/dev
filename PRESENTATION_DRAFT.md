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