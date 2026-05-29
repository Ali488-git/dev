# SPEAKER NOTES - TERRAFORM CLOUD MIGRATION PRESENTATION
## Aligned with 10-Slide Concise Presentation

---

## SLIDE 1: TITLE SLIDE
### Terraform Cloud Migration Initiative

**Opening Statement:**
"Good morning, everyone. Thank you for being here. Today I want to walk you through Metrolinx's strategic initiative to migrate our Terraform infrastructure management from GitHub Actions to Terraform Cloud. This is a significant but manageable transformation that will modernize how we manage infrastructure, improve security, and establish a foundation for future self-service capabilities."

**Speaker Notes:**
- This presentation covers a 6-month phased migration program
- The initiative is low-risk because we're taking an incremental approach with non-production environments first
- We have a HashiCorp partner supporting us throughout the early phases
- By the end of this presentation, you'll understand what's changing, why it matters, and how we'll execute it safely
- This isn't a redesign of our Azure infrastructure—it's a modernization of how we execute and manage Terraform

---

## SLIDE 2: CURRENT STATE OVERVIEW
### Where We Are Today

**Slide Context:**
"Let's start by understanding what we're working with today. We have four key repositories in our infrastructure ecosystem. Two of them are moving to Terraform Cloud as part of this initiative, and two are already there—which actually gives us proven reference patterns to learn from."

**In-Scope Repositories (Moving to TFC):**

1. **Azure IaC (Priority P1) - The Heavy Lift**
   - This is our primary monorepo containing 60+ workload directories
   - Manages Dev, SIT, UAT, and Prod environments
   - Currently uses Azure Storage backends with templated backend configurations
   - State is fragmented across multiple containers
   - GitHub Actions triggers deployments on every commit
   - This repository will be our primary focus—it's the most complex migration

2. **AZ Policy Repo (Priority P2) - Lighter Lift**
   - Contains Azure Policy definitions and policy assignments
   - Manages policies across management groups
   - Fewer dependencies, lower complexity
   - Good secondary target after establishing patterns with Azure IaC

**Out-of-Scope Repositories (Already on TFC):**

1. **Azure Products IaC (Priority P3)**
   - Already successfully migrated to Terraform Cloud
   - Products and workload-specific deployments for the team
   - Serves as our reference implementation for workspace structure
   - We can learn from their project/resource/workspace hierarchy
   - Provides proof that TFC works well for our use cases

2. **Ansible TerraformCloud Integration (Priority P3)**
   - Already operational with existing TFC integration
   - Contains Ansible playbooks for automatic TFC resource provisioning
   - Handles ServiceNow ticket-to-infrastructure automation
   - Shows us the end-to-end integration possibilities
   - We'll use this as a baseline for our approval workflow design

**Current Challenge:**
"Today, our Terraform state is distributed across Azure Storage containers. Each repository manages its own backend, leading to operational complexity, manual variable injection, and difficulty tracking what's deployed where. This fragmentation is our primary motivation for this migration."

---

## SLIDE 3: CURRENT WORKFLOW (CI/CD)
### How GitHub Actions Works Today

**Visual Reference:** [Image 1 - CI/CD Workflow]

**Detailed Walkthrough:**

1. **Developer Commits Code**
   - Engineer makes changes to Terraform code in GitHub repository
   - Could be infrastructure changes, module updates, or variable changes
   - Changes are pushed to the repository (either directly or via PR)

2. **GitHub Actions Triggered**
   - Our workflow file activates on push or pull request events
   - GitHub runners spin up to execute the defined workflow
   - This is managed entirely within GitHub's infrastructure
   - We pay for GitHub Actions runtime minutes

3. **Terraform Validation**
   - `terraform fmt` checks code formatting consistency
   - `terraform validate` ensures syntax correctness
   - These run before any actual infrastructure changes
   - Fail early on code quality issues

4. **Validation Decision Point**
   - If validation fails: Pipeline stops, developer is notified, must fix code
   - If validation succeeds: Move forward to planning

5. **Terraform Plan**
   - Shows what infrastructure changes would occur
   - Creates a preview without making changes
   - Plan output is posted in the PR for visibility

6. **PR Review & Approval**
   - Manual review step where team members must approve
   - This is our primary governance mechanism today
   - Someone must click "Approve and Merge" in GitHub
   - No additional checks or secondary approvals

7. **Terraform Apply**
   - GitHub Actions executes the terraform apply command
   - Changes are deployed to Azure
   - State is updated in Azure Storage containers
   - This is a direct, synchronous operation

8. **Resource Provisioned in Azure**
   - Infrastructure is now live in Azure
   - State file contains the resource definitions
   - Ready for operations teams to verify

**Current Limitations:**
"While this process works, it has several pain points: state is scattered across multiple Azure Storage accounts, we have no centralized audit trail, secrets are stored in GitHub (increasing exposure), and there's limited visibility into infrastructure across teams. Additionally, if something breaks, troubleshooting requires access to multiple systems."

---

## SLIDE 4: TARGET WORKFLOW (TFC)
### How Terraform Cloud Workflow Will Work

**Visual Reference:** [Image 2 - TFC Workflow]

**Detailed Walkthrough:**

1. **User Creates ServiceNow Ticket**
   - Users request infrastructure (e.g., new VMs) through self-service
   - ServiceNow ticket captures all required parameters
   - Not a Git commit—much more user-friendly for non-technical requesters
   - Ticket includes purpose, environment, sizing, and compliance tags

2. **Manager Approval Required**
   - First approval gate: Direct manager must approve the request
   - Ensures budgetary awareness and alignment with team priorities
   - Takes time but adds governance rigor
   - Approval workflows through ServiceNow

3. **Cloud OPS Validates Request**
   - Our cloud operations team validates technical feasibility
   - Checks for resource conflicts, naming conventions, compliance
   - Ensures request aligns with our infrastructure standards
   - May require clarifications or adjustments

4. **Request Valid? (Decision Point)**
   - If No: Request is rejected with feedback, user can resubmit
   - If Yes: Proceed to automation

5. **Payload → Ansible**
   - ServiceNow payload is converted to Ansible input
   - Ansible reads parameters and constructs Terraform variables
   - Prepares the infrastructure request for Terraform Cloud
   - This is the orchestration layer we've built

6. **TFC API Triggered**
   - Ansible calls Terraform Cloud API
   - TFC creates a workspace if needed (or uses existing one)
   - Terraform plan is executed showing what will be created
   - This is fully automated—no manual Git commits required

7. **Plan Succeeded? (Decision Point)**
   - If No: Plan failed with validation errors, user is notified, request ends
   - If Yes: Move to approval for apply

8. **Review & Approve Apply**
   - Cloud OPS team reviews the plan output
   - This is a second approval gate for governance
   - Only after approval does the actual infrastructure get created
   - Provides a critical "stop" point if something looks wrong

9. **Resource Provisioned in Azure**
   - Terraform apply is executed by TFC
   - Infrastructure is created in Azure
   - State is immediately stored in Terraform Cloud (not Azure Storage)
   - User is notified through ServiceNow of completion

**Key Differences:**
"The critical shift here is from code-driven (Git commits) to request-driven (ServiceNow tickets), from distributed state (Azure Storage) to centralized state (TFC), from single approval (GitHub PR) to multi-stage approvals (Manager → CloudOps → TFC). This creates a more formal, auditable, and user-friendly process."

---

## SLIDE 5: WHAT'S CHANGING vs. WHAT'S NOT
### Minimal Disruption, Maximum Benefit

**What Stays the Same - Reassurance Point:**

"I want to be very clear: we're not redesigning your infrastructure. We're not changing how Terraform code is structured. We're not breaking your deployment patterns."

1. **Terraform Code Structure**
   - Your modules remain exactly as they are
   - Module organization stays the same
   - Variable definitions unchanged
   - Resource groupings preserved
   - Engineers can continue working with familiar patterns

2. **GitHub Repositories & PR Workflow**
   - GitHub remains our source of truth for code
   - PRs are still how code changes are proposed and reviewed
   - Git history is preserved
   - Collaboration workflow unchanged
   - We're not giving up version control benefits

3. **Azure Infrastructure Design**
   - No resource reorganization
   - Subscription structure stays the same
   - Networking architecture unchanged
   - Security groups and policies preserved
   - VNets, subnets, resource groups all remain as-is

4. **Ansible Usage**
   - Ansible continues managing OS and application configuration
   - Integration deepens, but fundamentally similar
   - Scripts, playbooks, inventory—all largely the same
   - Ansible becomes the bridge between ServiceNow and TFC

**What's Changing - The Transformations:**

1. **Terraform Execution Platform: GitHub Actions → Terraform Cloud**
   - Why: TFC is built specifically for Terraform execution at scale
   - Benefit: Better resource management, no more GitHub runner bottlenecks
   - Impact: Faster, more reliable deployments

2. **State Backend Location: Azure Storage → Terraform Cloud**
   - Why: Single source of truth, better security, built-in versioning
   - Benefit: Centralized state, no more manual state management
   - Impact: Easier troubleshooting, better collaboration

3. **Credential & Secret Handling: GitHub Secrets → TFC Variables**
   - Why: TFC has superior secret management and encryption
   - Benefit: Audit trails for who accessed what, encryption in transit and at rest
   - Impact: Significantly improved security posture

4. **Variable Management Model: Runtime Injection → Declarative TFC Variables**
   - Why: TFC provides superior variable organization and validation
   - Benefit: Variables are visible in TFC UI, easier to audit, no shell script parsing
   - Impact: Reduced complexity, fewer errors from malformed variables

5. **Access Control & Approvals: GitHub PR Reviews → TFC + CloudOps + Manager**
   - Why: Multi-stage approvals provide better governance
   - Benefit: Business ownership (manager) + technical validation (CloudOps) + system level (TFC)
   - Impact: Better control, compliance-friendly, audit-friendly

6. **Platform Operations & Governance: Manual → Centralized Visibility**
   - Why: TFC provides dashboards, audit logs, cost estimation, policy enforcement
   - Benefit: Leaders have visibility into infrastructure changes in real-time
   - Impact: Better compliance, faster incident response, data-driven decisions

**Why This Balance Matters:**
"By preserving what works and only changing what needs improvement, we minimize disruption to your daily work while delivering real operational benefits. This isn't a wholesale replacement—it's a thoughtful modernization that respects the expertise you've built."

---

## SLIDE 6: TERRAFORM CLOUD SETUP
### Current Architecture (Reference Pattern)

**Context:**
"Let me show you how our existing TFC setup is organized. Azure Products IaC has already successfully implemented this pattern, and we'll follow a similar approach for Azure IaC."

**Hierarchical Organization:**

**Level 1: Organization**
```
metrolinx (Single TFC Organization)
├─ All Metrolinx infrastructure under one roof
├─ Centralized billing and user management
├─ Single set of policies and governance rules
└─ Shared backend infrastructure
```

"We have one Terraform Cloud organization called 'metrolinx'. This is our top-level container. All teams, all projects, all workspaces live under this organization. This gives us centralized visibility and control."

**Level 2: Projects**
```
database-app (Project)
├─ Dev environment project
├─ SIT environment project
├─ UAT environment project
└─ Prod environment project
```

"Within that organization, we create projects per application or business function. For example, if you have a database application, you'd have a project called 'database-app'. This is purely organizational—it helps us keep things logically grouped. Each project can have its own access controls and team assignments."

**Level 3: Workspaces (Per Resource Type)**
```
database-app / Prod / Workspaces:
├─ db-server-prod
│  └─ State file containing only DB server configuration
├─ db-backup-prod
│  └─ State file containing backup infrastructure
└─ db-cache-prod
   └─ State file containing cache infrastructure
```

"The critical part: each workspace has its own dedicated state file. This means a VM workspace only tracks VM state, a backup workspace only tracks backup infrastructure, etc. State is never mixed. This reduces blast radius—a problem in one workspace doesn't affect another."

**Real Example: Database Application**

"Let's walk through a concrete example. Say you have a database application serving production. Here's how it's organized:

- **Project Name**: database-app
- **Workspaces in Prod**:
  - `db-server-prod` → Manages the database VMs, disks, network interfaces
  - `db-backup-prod` → Manages backup storage, backup policies, recovery vaults
  - `db-cache-prod` → Manages Redis/cache infrastructure, cache networking

Each workspace is independent. If there's an issue with backup infrastructure, it doesn't affect the database servers because they're in different workspaces with separate state files."

**State File Isolation Principle:**
```
Isolation = Application × Environment × Resource Type
```

"This isolation pattern is what makes TFC powerful. You get granularity without chaos. Engineers can work on one thing without worrying about unintentionally affecting others."

**Why This Matters for Azure IaC:**
"With 60+ workload directories, we need this hierarchical approach. We can't have 240 workspaces (60 dirs × 4 envs) all at the same level—that's overwhelming. Instead, we group by application or function, then by environment, then by resource type. This gives us manageability and clarity."

---

## SLIDE 7: MIGRATION PHASES & TIMELINE
### 6-Month Phased Approach

**Overall Philosophy:**
"We're taking a measured, phased approach. Non-production environments first, production last. This allows us to learn, refine our processes, and build confidence before touching production infrastructure."

**Phase 0: Discovery & Validation (Weeks 1-3)**

**Objectives:**
- Map every single Terraform state currently in use
- Document all backends, backends configs, and variable injection patterns
- Understand dependencies between workspaces
- Design optimal workspace consolidation strategy

**Key Activities:**
- Inventory all 60+ workload directories in Azure IaC
- Catalog all policy definitions in AZ Policy Repo
- Review secrets management patterns
- Design workspace mapping (we'll use HashiCorp guidance here)

**HashiCorp's Role:**
- Review our current state architecture
- Validate proposed workspace mapping against best practices
- Recommend consolidation patterns that balance granularity and manageability
- Identify any gaps or antipatterns in our current setup

**Deliverable:** Clear, validated migration roadmap with workspace strategy documented

**Phase 1: Foundation & Initial Migration (Weeks 4-9)**

**Objectives:**
- Stand up Terraform Cloud infrastructure
- Configure organization, teams, and access controls
- Perform initial state migrations
- Validate clean plans and dry-runs

**Key Activities:**
- Create TFC organization and project structure
- Set up SSO integration with directory services
- Build "Terraform Cloud Control Repository" (IaC for TFC itself)
- Migrate first batch of state files using Terraform CLI
- Execute dry-run plans to verify correctness
- Test all variable handling and secret substitution

**Why Dry-Runs Matter:**
"When we migrate state, we don't want to accidentally trigger infrastructure changes. Dry-runs prove that after migration, a `terraform plan` shows no changes—meaning the state is correct and intact."

**HashiCorp's Role:**
- Guide TFC organization setup
- Validate RBAC models and access patterns
- Review control repository design
- Provide oversight during initial migrations
- Approve dry-run results before cutover

**Deliverable:** TFC platform operational, first migrations tested and validated

**Phase 2: Pilot and Scale (Weeks 10-15)**

**Objectives:**
- Establish repeatable migration process
- Expand to all non-production environments
- Validate end-to-end workflows

**Key Activities:**
- Migrate Dev environment (lower risk, good learning ground)
- Document lessons learned and refine procedures
- Migrate SIT environment
- Migrate UAT environment
- Full end-to-end workflow testing: ServiceNow → Ansible → TFC → Azure

**De-Risk Strategy:**
"Each environment migration follows the same pattern: plan → dry-run → cutover → 7-day verification. By the time we finish UAT, the process is well-established and confidence is high."

**Deliverable:** Standardized migration process proven across 3 environments

**Phase 3: Production Migration (Weeks 16-21)**

**Objectives:**
- Migrate production environments with zero downtime
- Maintain rollback capability throughout
- Minimize risk through gradual cutover

**Key Activities:**
- Comprehensive backup of all production states
- Full production dry-run with rollback plan prepared
- Wave-based cutover (migrate one application at a time, not all at once)
- Maintain GitHub Actions as active fallback
- Comprehensive post-migration monitoring (7 days per wave)
- Gradual traffic cutover with instant rollback capability

**Production Governance:**
"For production, we're extra careful. We have backups, we have rollback plans, we test everything twice. We migrate in waves so if something goes wrong, it's limited to one application, not everything."

**Deliverable:** Production migrated with zero downtime, rollback capability proven

**Phase 4: Stabilization (Weeks 22-26)**

**Objectives:**
- Retire legacy systems
- Normalize configuration across all workspaces
- Establish operational steady-state

**Key Activities:**
- Remove GitHub Actions Terraform workflows
- Clean up old backend configurations
- Remove Azure Storage dependencies (data retention for 90 days as archive)
- Standardize variable naming and defaults across all workspaces
- Complete team training and documentation
- Align team practices with Azure Products IaC patterns

**Knowledge Transfer:**
"This phase is critical for self-sufficiency. We document everything, we train the team, and by the end of this phase, we're operating TFC independently without HashiCorp support."

**Deliverable:** Single source of truth (TFC), legacy systems retired, team self-sufficient

**Phase 5: Optimization (Weeks 27+, Optional)**

**Objectives:**
- Build advanced capabilities for future growth

**Enhancements:**
- Private Terraform module registry (reusable components across teams)
- Dynamic Azure credentials with OIDC (instead of static service principals)
- Policy-as-Code enforcement in TFC (automatic compliance checks)
- Drift detection (detect when Azure resources change outside Terraform)
- Self-service provisioning enhancement (users can request resources without tickets)

"Phase 5 is optional because the core migration is complete by Phase 4. But these optimizations enable even better efficiency and governance."

**Timeline Visualization:**

```
Total Duration: ~6 Months

Week 1-3:   Phase 0 (Discovery)                    ✓ Gate: Strategy Approved
Week 4-9:   Phase 1 (Foundation)                   ✓ Gate: Platform Live
Week 10-15: Phase 2 (Pilot & Scale)                ✓ Gate: Non-Prod Complete
Week 16-21: Phase 3 (Production Migration)         ✓ Gate: Production Live
Week 22-26: Phase 4 (Stabilization)                ✓ Gate: Mission Complete
Week 27+:   Phase 5 (Optimization, Optional)
```

---

## SLIDE 8: 60+ WORKLOAD DIRECTORIES SOLUTION
### Workspace Consolidation Strategy

**The Challenge Explained:**

"Azure IaC has 60+ workload directories. If we naively said 'one directory = one workspace', and we have 4 environments (Dev, SIT, UAT, Prod), that's 240+ workspaces. Imagine managing 240 workspaces—that's chaos. We'd have workspaces called:
- app-one-dev
- app-two-dev
- app-three-dev
- ... (240 times)

That's unmanageable. You can't keep track of them, you can't govern them effectively, and operational overhead explodes."

**Solution Options (Validated with HashiCorp):**

**Option A: Group by Function/Team**
```
~24-30 workspaces per environment
Organized by business function or team:
├─ Networking Workspaces (Dev, SIT, UAT, Prod)
├─ Database Workspaces (Dev, SIT, UAT, Prod)
├─ Compute Workspaces (Dev, SIT, UAT, Prod)
├─ Storage Workspaces (Dev, SIT, UAT, Prod)
└─ Security Workspaces (Dev, SIT, UAT, Prod)
```

**Benefits:**
- Manageable number of workspaces (30-40 total for 4 environments)
- Logical grouping by domain
- Each team owns their function's workspaces
- Easier to apply governance policies

**Trade-offs:**
- Less granular isolation (more resources per workspace)
- Larger blast radius if something breaks (affects entire function)
- Requires careful dependency management

**Option B: Consolidated Per Environment**
```
4 workspaces total (one per environment):
├─ dev-infrastructure
├─ sit-infrastructure
├─ uat-infrastructure
└─ prod-infrastructure
```

**Benefits:**
- Simplest to manage and understand
- Minimal workspace governance overhead
- Clear environment separation
- Good for smaller teams

**Trade-offs:**
- Everything in one workspace per environment
- Very large blast radius (changes affect all infrastructure in that environment)
- Harder to parallelize deployments
- Less suitable for large teams with independent domains

**Option C: Hybrid Approach**
```
Balance between granularity and manageability:
├─ Core Infrastructure (Networking, Security - shared across teams)
├─ Application A (All environments)
├─ Application B (All environments)
├─ Application C (All environments)
└─ Shared Services (Monitoring, Logging, etc.)
```

**Benefits:**
- Core shared infrastructure isolated from application infrastructure
- Applications can be deployed independently
- Teams own their application workspaces
- 10-15 workspaces total (very manageable)

**Trade-offs:**
- Requires application-level organization clarity
- Some infrastructure still shared per workspace
- Needs clear ownership model

**Recommended Approach:**

"Based on HashiCorp's recommendations and Azure Products IaC's proven pattern, we recommend **Option A with elements of Option C**: Group related workloads by function/team, isolate core infrastructure separately, and target 24-30 total workspaces across all environments. This gives us:
- Manageability (not too many workspaces)
- Governance clarity (by domain/team)
- Scalability (easy to add new applications)
- Safety (reasonable blast radius)"

**Reference Pattern (Already Proven):**

"Azure Products IaC demonstrates this pattern works. They use project-per-application, workspace-per-resource-type. We'll follow a similar model for Azure IaC but at a higher level: projects for application clusters, workspaces for infrastructure domains."

**Decision Timeline:**

"HashiCorp will help us finalize this strategy during Phase 0. We'll document the exact mapping, and all team leads will review and sign off before we begin migration."

---

## SLIDE 9: EFFORT, RISK & ROLLBACK
### Investment Required & Safety Measures

**Effort Estimation - Transparency:**

"Let me be transparent about the effort this requires. We're not underselling it, but we're also not overcomplicating it."

**Phase Breakdown (Total ~600 hours = 1.5 FTE over 6 months):**

1. **Phase 0 - Discovery (80 hours)**
   - 2-3 engineers × 4 weeks of deep analysis
   - Inventory all states, backends, variables
   - Design workspace structure with HashiCorp

2. **Phase 1 - Foundation (160 hours)**
   - Most time-intensive phase
   - TFC setup, organization structure, SSO integration
   - Initial state migrations and dry-runs
   - Control repository development and testing
   - 3-4 engineers × 6 weeks

3. **Phase 2 - Non-Prod Migrations (120 hours)**
   - Dev, SIT, UAT environments (3 × 30-40 hours each)
   - Applying established processes from Phase 1
   - Workflow testing and validation
   - 2-3 engineers × 6 weeks

4. **Phase 3 - Production Migration (100 hours)**
   - Highest care and caution
   - Backups, dry-runs, careful cutover
   - Monitoring and validation
   - Rollback testing
   - 2-3 engineers × 6 weeks

5. **Phase 4 - Stabilization (80 hours)**
   - Team training and knowledge transfer
   - Documentation and runbooks
   - Cleanup and normalization
   - Process formalization
   - 1-2 engineers × 5 weeks

6. **Phase 5 - Optimization (40 hours, optional)**
   - Advanced features, policy-as-code, module registry
   - Self-service enhancements
   - 1 engineer × 5 weeks (if pursued)

**Cost-Benefit Analysis:**

"This investment of 600 hours upfront delivers:
- Elimination of manual state management (saves 10+ hours/month ongoing)
- Reduced incident response time (from hours to minutes when things break)
- Audit compliance readiness (automatic, no manual log gathering)
- Foundation for self-service (enables future capabilities without additional migration)
- Improved security (secrets management, audit trails, encryption)

ROI: Within 3-4 months, the operational efficiency gains pay back the migration investment."

**Risk Profile - Why Low Risk:**

1. **Incremental Approach**
   - We're not doing a big-bang cutover
   - Each phase builds confidence for the next
   - Non-production first means we practice before doing it for real

2. **Reference Patterns Available**
   - Azure Products IaC already uses TFC
   - Ansible TFC Integration is already operational
   - We have working examples to follow

3. **Production Last**
   - Production doesn't migrate until we've done it successfully 3 times (Dev, SIT, UAT)
   - By week 21, we're experts at the process

4. **Parallel Running**
   - GitHub Actions workflows stay active during entire migration
   - We can switch back to GitHub if something goes wrong
   - No forced cutover date; we go when ready

5. **HashiCorp Support**
   - Expert guidance during critical early phases
   - Validation of our approach against best practices
   - Reduces chance of architectural mistakes

**Risk Assessment: LOW**

"We're not doing anything novel here. We're following established patterns, learning from proven implementations already in our organization, and moving carefully. This isn't a risky migration—it's a well-planned evolution."

**Rollback Capability - Safety Valve:**

"I want to address the obvious question: 'What if something goes wrong?' We have comprehensive rollback procedures."

**Phase 1-2 Rollback (Dev/SIT/UAT):**
- **Timeframe**: Immediate to 1 hour
- **Procedure**: Stop TFC, switch GitHub Actions workflows back on, redeploy from GitHub
- **Data Loss**: Zero—Azure Storage state files are retained during transition
- **Decision**: Can rollback without any data loss

**Phase 3 Rollback (Production):**
- **Timeframe**: Within 15 minutes
- **Procedure**: Instant failover to GitHub Actions using pre-prepared workflows
- **Data Loss**: Zero—TFC states are exported and backed up
- **Decision**: Even in production, rollback is quick and safe

**Post-Migration Contingency:**
- **Backup Retention**: 90 days (Azure Storage states kept as archive)
- **State Exports**: All TFC states exported to Git for version control
- **Runbooks**: Clear procedures documented for every contingency scenario
- **On-Call**: Designated team on standby during cutover windows

**Contingency Scenarios:**

| Scenario | Response | Prevention |
|----------|----------|-----------|
| TFC authentication fails | Revert to GitHub Actions immediately | SSO testing in Phase 1 |
| State migration corrupted | Restore from backup, retry migration | Dry-run validation per workspace |
| Approval workflow blocked | Bypass TFC temporarily | User training, workflow tuning |
| Production performance degrades | Instant failover to GitHub Actions | Load testing in Phase 2 |
| Variable mapping errors | Revert, fix mapping, retry | Comprehensive audit Phase 0 |

"For each risk, we have a prevention strategy and a remediation plan. That's what makes this low-risk."

---

## SLIDE 10: KEY OUTCOMES & NEXT STEPS
### What We're Achieving & Path Forward

**Key Outcomes - The Real Wins:**

1. **Centralized State & Execution (Single Source of Truth)**
   - Right now: State scattered across Azure Storage, GitHub Actions scattered across repos
   - After Migration: All state in TFC, all execution through TFC
   - Benefit: One place to look, one place to audit, one place to manage
   - Real Impact: When something breaks, we troubleshoot in minutes not hours

2. **Improved Security (Encrypted Secrets & Audit Trails)**
   - Right now: Secrets in GitHub (exposed to anyone with repo access), limited logging
   - After Migration: Secrets encrypted in TFC, comprehensive audit trails
   - Benefit: Compliance teams have visibility, security team has control
   - Real Impact: Passes security audits, reduces vulnerability surface

3. **Stronger Governance & Compliance**
   - Right now: Manual approval through GitHub PRs, limited audit history
   - After Migration: Multi-stage approvals (Manager → CloudOps → System), complete audit logs
   - Benefit: Business owners involved in infrastructure decisions, regulators get visibility
   - Real Impact: Meets compliance requirements, enables future regulatory certifications

4. **Foundation for Self-Service Provisioning**
   - Right now: Users need to create Git commits or contact ops team
   - After Migration: Users request through ServiceNow, automatic fulfillment
   - Benefit: Faster provisioning, self-service reduces bottlenecks
   - Real Impact: Teams can iterate faster, ops team focuses on complex tasks

**Business Value Summary:**

"This migration isn't just technical—it's transformational for how we operate. We move from a fragmented, manual process to a centralized, automated, auditable system. Operations teams focus on strategy instead of firefighting. Engineering teams get self-service access. Security and compliance teams get visibility they need."

**Next Steps - Immediate Actions:**

1. **Approve the Migration Plan**
   - Executive alignment on phasing and timeline
   - Budget approval for HashiCorp partner engagement
   - Stakeholder sign-off on scope (in-scope: Azure IaC + AZ Policy; out-of-scope: already migrated repos)

2. **Engage HashiCorp Partner**
   - Formal engagement kickoff meeting
   - HashiCorp reviews our architecture and current state
   - Design Phase 0 discovery activities

3. **Schedule Kickoff Meeting**
   - All team leads and technical stakeholders
   - Clarify roles and responsibilities
   - Establish communication cadence

4. **Establish Monthly Checkpoints**
   - Monthly steering committee review
   - Progress tracking against timeline
   - Risk assessment and mitigation updates
   - Course correction if needed

5. **Phase 0 Starts (Immediate)**
   - Begin infrastructure inventory
   - Start HashiCorp engagement
   - Design workspace consolidation strategy
   - Target completion: Week 3

**Success Criteria - How We Know It Worked:**

- ✓ All Azure IaC state migrated to TFC without data loss
- ✓ All AZ Policy Repo state migrated to TFC
- ✓ Zero production downtime during migration
- ✓ Full audit trail available in TFC for all infrastructure changes
- ✓ Team operates independently without HashiCorp support (Phase 4)
- ✓ New resource requests through ServiceNow → Ansible → TFC workflow
- ✓ Incident response time improved (measured and tracked)

**Final Message:**

"This migration represents a strategic investment in operational excellence. We're taking proven patterns from Azure Products IaC and Ansible TFC Integration and scaling them across the entire infrastructure. With careful planning, expert guidance, and a measured approach, we'll emerge with a modern, secure, scalable platform that serves Metrolinx for years to come.

I'm confident in this plan. I'm confident in our team. And I'm confident that in 6 months, you'll see why this was the right investment to make.

Thank you. Questions?"

---

## END OF SPEAKER NOTES
