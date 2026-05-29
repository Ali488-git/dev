# SPEAKER NOTES - TERRAFORM CLOUD MIGRATION PRESENTATION
## Aligned with 26-Slide Structure

---

## SLIDE 1: TITLE SLIDE
### Terraform Cloud Migration Initiative

**Opening:**
"Good morning. Today we're walking through Metrolinx's strategic initiative to migrate our Terraform infrastructure from GitHub Actions to Terraform Cloud. This is a phased, low-risk transformation that will modernize operations, improve security, and establish a foundation for self-service capabilities."

---

## SLIDE 2: HIGH-LEVEL OVERVIEW OF CURRENT CLOUD INFRASTRUCTURE

**Key Points:**
- All infrastructure defined as code via Terraform in GitHub repositories
- Every change tracked, reviewed, and approved through Git workflows
- GitHub Actions automates validation and deployment
- Azure cloud platform hosts all infrastructure
- Multi-environment support (Dev, SIT, UAT, Prod) from single codebase
- Azure Policies enforce compliance and governance

---

## SLIDE 3: GITHUB REPOSITORIES & INFRASTRUCTURE MANAGEMENT

**In-Scope Repositories (Migration targets):**
- **Azure IaC (P1)** - Primary monorepo, 60+ workload directories, Dev/SIT/UAT/Prod, currently Azure Storage backend
- **AZ Policy Repo (P2)** - Azure Policy definitions and assignments, lighter complexity

**Out-of-Scope (Already on TFC):**
- **Azure Products IaC (P3)** - Reference implementation for workspace design
- **Ansible TerraformCloud Integration (P3)** - Already operational, shows integration patterns

**Current State:**
Terraform state fragmented across Azure Storage. Each repo manages its own backend. This fragmentation creates operational complexity.

---

## SLIDE 4: CURRENT WORKFLOW vs IDEAL WORKFLOW

**Current Workflow (GitHub Actions):**
Developer commits → GitHub Actions triggered → Terraform validate/format → PR review → Terraform apply → Resources in Azure (state in Azure Storage)

**Ideal Workflow (Terraform Cloud):**
ServiceNow ticket → Manager approval → Cloud OPS validates → Ansible → TFC API → Plan → Review/approve → Resources in Azure (state in TFC)

**Key Differences:**
- Code-driven → Request-driven
- Single approval → Multi-stage approvals
- Distributed state → Centralized state

---

## SLIDE 5: CURRENT vs IDEAL WORKFLOW COMPARISON TABLE

**Use table as visual reference. Key points to emphasize:**
- Trigger mechanism: Git commit → ServiceNow ticket (user-friendly)
- State management: Distributed → Centralized (consistency)
- Approval workflow: Single → Multi-stage (governance)
- Audit trail: GitHub logs → TFC + Ansible + ServiceNow (comprehensive)
- Cost estimation: None → Built-in TFC capability

"Notice the improvements span user experience, compliance, and operational visibility."

---

## SLIDE 6: AZURE SUBSCRIPTION MAPPING

**Context:**
"Each repository uses a combination of these subscriptions depending on target environments."

**Key Subscriptions:**
- Shared Infrastructure: Connectivity, Management (prod)
- Environment-Specific: Dev, SIT, UAT, Prod (each with core + DMZ)
- Sandbox: Testing and experimentation

**Why This Matters:**
Ensures consistent backend configuration across deployments and proper workload isolation.

---

## SLIDE 7: WHAT WILL AND WILL NOT CHANGE

**Staying the Same:**
- Terraform code structure
- GitHub repositories and PR workflow
- Azure infrastructure design
- Ansible for OS/app configuration

**Changing:**
- Execution platform (GitHub Actions → TFC)
- State backend (Azure Storage → TFC)
- Secrets management (GitHub → TFC)
- Variable management (Runtime → Declarative)
- Approval workflow (GitHub → Multi-stage)
- Governance model (Manual → Centralized visibility)

"Minimal disruption, maximum benefit."

---

## SLIDE 8: TERRAFORM CLOUD

**Link:** app.terraform.io/app/metrolinx/workspaces

**Description:** Remote state management and workspace orchestration platform for centralized Terraform state, VCS integration, and automated deployment.

**Current TFC Setup (Reference Pattern):**
- Organization: metrolinx (single)
- Projects: Per application/function
- Workspaces: Per resource type per environment
- State files: Dedicated per workspace

"Each workspace has isolated state. This prevents blast radius and enables team independence."

---

## SLIDE 9: WORKSPACE STRATEGY

**Challenge:**
"Azure IaC has 60+ workload directories. If we do 1:1 mapping, that's 240+ workspaces (60 directories × 4 environments). Unmanageable."

**Recommended Solution (Validated with HashiCorp):**
Group by function/team. Target 24-30 total workspaces across all environments.

**Example:**
- Networking workspaces (Dev/SIT/UAT/Prod)
- Database workspaces (Dev/SIT/UAT/Prod)
- Compute workspaces (Dev/SIT/UAT/Prod)
- Storage workspaces (Dev/SIT/UAT/Prod)
- Security workspaces (Dev/SIT/UAT/Prod)

**Benefits:**
- Manageable number
- Logical domain grouping
- Clear team ownership
- Easier governance

**Reference:** Azure Products IaC successfully uses this pattern.

---

## SLIDE 10: TERRAFORM CLOUD MIGRATION INITIATIVE

**Overview:**
Metrolinx is migrating Terraform execution from GitHub Actions to Terraform Cloud to establish centralized, authoritative infrastructure state management.

**Key Principle:**
This is **re-platforming of operations, not redesign of infrastructure**. Existing environments, code, repositories, and workflows remain unchanged.

**Key Outcomes:**
1. Centralized state and execution (single source of truth)
2. Improved security and credential management
3. Stronger governance, auditability, and compliance
4. Scalable foundation for self-service provisioning

**Risk Profile:** LOW (incremental, non-production first, reference patterns available, HashiCorp partner support)

---

## SLIDE 11: MIGRATION APPROACH

**Partnership Model:**
HashiCorp partner engagement primarily during Phases 0-1.

**Partner's Role:**
- Validate architecture against best practices
- Provide guidance on workspace organization and team structure
- Review backend configuration and secrets management
- Help establish state migration best practices
- Identify and mitigate risks early

**Our Role:**
- Own implementation and day-to-day execution
- Build institutional knowledge for independence
- Manage state migration process
- Handle repository and workflow updates
- Drive team adoption and training

**Why This Model Works:**
Reduces architectural risk while ensuring internal ownership and knowledge transfer.

---

## SLIDE 12: PHASE 0 – DISCOVERY & VALIDATION (WEEKS 1-3)

**Objective:** Map current infrastructure and validate migration strategy.

**Internal Activities:**
- Inventory all Terraform states and repositories
- Identify current backends, variables, and secrets
- Map each existing state to future TFC workspace

**HashiCorp Responsibilities:**
- Review current Terraform execution model and backend usage
- Validate state architecture and proposed workspace mapping
- Identify gaps or changes in recommended practices

**Deliverable:** Clear inventory and validated migration roadmap.

**Effort:** ~80 hours

---

## SLIDE 13: PHASE 1 – FOUNDATION & INITIAL MIGRATION (WEEKS 4-9)

**Objective:** Establish Terraform Cloud platform and execute initial state migrations.

**Internal Activities:**
- Configure TFC organization, teams, and permissions
- Create Terraform Cloud control repository
- Migrate existing Terraform state using CLI
- Validate clean plans post-migration

**HashiCorp Responsibilities:**
- Assist TFC organization setup
- Configure SSO integration, project/team RBAC, VCS connections
- Support control repository creation and review
- Provide oversight during initial state migrations

**Deliverable:** TFC platform operational; first state migrations validated.

**Effort:** ~160 hours (most time-intensive)

---

## SLIDE 14: PHASE 2 – PILOT AND SCALE (WEEKS 10-15)

**Objective:** Establish repeatable migration process and expand across environments.

**Activities:**
- Migrate low-risk non-production workspace first
- Document repeatable migration runbook
- Migrate remaining environments in risk-based waves
- Production environments migrate last

**Key Focus:**
Establish confidence through repetition before touching production.

**Deliverable:** Standardized migration process; most non-prod environments in TFC.

**Effort:** ~120 hours

---

## SLIDE 15: PHASE 3 – PRODUCTION MIGRATION (WEEKS 16-21)

**Objective:** Migrate production environments with zero downtime.

**Activities:**
- Comprehensive backup of all production states
- Full production dry-run with rollback plan prepared
- Wave-based cutover (one app at a time, not all at once)
- Maintain GitHub Actions as active fallback
- Comprehensive post-migration monitoring (7 days per wave)
- Gradual traffic cutover with instant rollback capability

**Key Principle:**
Production doesn't migrate until non-prod is 100% successful.

**Deliverable:** Production migrated with zero downtime, rollback capability proven.

**Effort:** ~100 hours

---

## SLIDE 16: PHASE 4 – STABILIZATION (WEEKS 22-26)

**Objective:** Retire legacy systems and formalize operations.

**Activities:**
- Remove GitHub Actions Terraform workflows
- Clean up old backend configurations
- Remove Azure Storage dependencies (retain 90-day backup)
- Standardize variable naming across all workspaces
- Complete team training and documentation
- Establish operational steady-state

**Knowledge Transfer:**
Team operates TFC independently without HashiCorp support.

**Deliverable:** Single source of truth (TFC); legacy systems retired; team self-sufficient.

**Effort:** ~80 hours

---

## SLIDE 17: PHASE 5 – OPTIMIZATION (WEEKS 27+, OPTIONAL)

**Objective:** Enable advanced capabilities for future growth.

**Potential Enhancements:**
- Private module registry for reusable components
- Dynamic Azure credentials with OIDC (vs. static service principals)
- Policy-as-Code enforcement (automatic compliance checks)
- Drift detection (detect when Azure resources change outside Terraform)
- Enhanced self-service provisioning

**Note:** Optional because core migration is complete by Phase 4.

**Effort:** ~40 hours (if pursued)

---

## SLIDE 18: TIMELINE WITH MILESTONES

**Overall Duration:** ~6 months

**Milestone Timeline:**
- **Week 3:** Workspace strategy defined ✓
- **Week 9:** TFC operational, dry-runs successful ✓
- **Week 15:** All non-prod environments migrated ✓
- **Week 21:** Production migrated ✓
- **Week 26:** Legacy cleanup complete ✓

**Phase Breakdown:**
- Phase 0: Weeks 1-3 (Discovery)
- Phase 1: Weeks 4-9 (Foundation)
- Phase 2: Weeks 10-15 (Pilot & Scale)
- Phase 3: Weeks 16-21 (Production)
- Phase 4: Weeks 22-26 (Stabilization)
- Phase 5: Weeks 27+ (Optional Optimization)

"This phased approach allows us to learn and refine before each major step."

---

## SLIDE 19: EFFORT JUSTIFICATION & ESTIMATION

**Total Effort: ~600 hours over 6 months (1.5 FTE)**

**Phase Breakdown:**
- Phase 0: 80 hours (discovery and inventory)
- Phase 1: 160 hours (most intensive—platform setup)
- Phase 2: 120 hours (non-prod migrations)
- Phase 3: 100 hours (production migration with safeguards)
- Phase 4: 80 hours (stabilization and training)
- Phase 5: 40 hours (optional optimization)

**Cost-Benefit Analysis:**
- **One-Time Cost:** 600 hours
- **Ongoing Savings:** Eliminates 10+ hours/month of manual state management
- **Incident Response:** Reduced from hours to minutes when troubleshooting
- **ROI:** Investment paid back within 3-4 months

**Additional Benefits:**
- Improved security posture (compliance-ready)
- Audit trails for all infrastructure changes
- Foundation for self-service provisioning
- Team operational efficiency

---

## SLIDE 20: ROLLBACK / CONTINGENCY PLAN - ROLLBACK STRATEGY

**Pre-Migration Safeguards:**

1. **Backup Strategy:**
   - All Azure Storage states exported before migration
   - Exported states stored in Git for version control
   - 90-day retention policy on all pre-migration backups

2. **GitHub Actions Preservation:**
   - GitHub Actions workflows remain active throughout migration
   - Can switch back to GitHub at any time during Phases 1-2
   - Production switch back capability within 15 minutes

3. **Dry-Run Validation:**
   - Every migrated workspace validated with clean plan
   - No unexpected infrastructure changes after migration
   - Proof that state is correct before cutover

4. **Monitoring Setup:**
   - Post-migration monitoring during 7-day verification window
   - Alert thresholds defined for anomalies
   - On-call team during cutover windows

---

## SLIDE 21: ROLLBACK EXECUTION & DATA LOSS PREVENTION

**Rollback Timeframes:**

**Non-Prod Rollback (Phases 1-2):**
- **Timeframe:** Immediate to 1 hour
- **Procedure:** Stop TFC, switch GitHub Actions workflows on, redeploy
- **Data Loss:** Zero—Azure Storage state files retained during transition
- **Complexity:** Low—straightforward reversal

**Production Rollback (Phase 3):**
- **Timeframe:** Within 15 minutes
- **Procedure:** Instant failover to GitHub Actions using pre-prepared workflows
- **Data Loss:** Zero—TFC states exported and backed up
- **Complexity:** Medium—requires coordination but proven process

**Post-Migration Safety:**
- All TFC states exported to Git (version-controlled)
- State backups retained for 90 days minimum
- Runbooks documented for every rollback scenario
- Designated on-call team during cutover windows

"We've designed this so there's never a scenario where we lose data or can't recover."

---

## SLIDE 22: CONTINGENCY SCENARIOS

**Scenario: TFC Authentication Fails**
- Response: Revert to GitHub Actions immediately
- Prevention: SSO testing in Phase 1
- Recovery Time: < 5 minutes

**Scenario: State Migration Corrupted**
- Response: Restore from backup, retry migration
- Prevention: Dry-run validation per workspace
- Recovery Time: 30 minutes

**Scenario: Approval Workflow Blocked**
- Response: Bypass TFC temporarily, revert to manual approval
- Prevention: User training, workflow tuning
- Recovery Time: Immediate

**Scenario: Production Performance Degrades**
- Response: Instant failover to GitHub Actions
- Prevention: Load testing in Phase 2
- Recovery Time: 15 minutes

**Scenario: Variable Mapping Errors**
- Response: Revert, fix mapping, retry migration
- Prevention: Comprehensive audit in Phase 0
- Recovery Time: 1-2 hours

"Each risk has a prevention strategy and a documented recovery plan."

---

## SLIDE 23: RISK MITIGATION & CONTROLS

**Risk: Incorrect state mapping**
- Control: Strict inventory and validation of all state files before migration

**Risk: Variable mismatches**
- Control: Full documentation of runtime inputs and variables across all environments

**Risk: Over-permissive access**
- Control: SSO-based least privilege access model with team-based RBAC

**Risk: Environment inconsistency**
- Control: Enforced naming standards and workspace configuration standards

**Risk: Over-engineering early phases**
- Control: Keep early phases minimal; focus on core functionality before optimization

**Risk Profile:** LOW (incremental approach, reference patterns available, expert guidance, parallel running capability)

---

## SLIDE 24: TERRAFORM STATE MIGRATION PRINCIPLE

**Migration Process (Consistent across all states):**

1. **Create** a corresponding Terraform Cloud workspace
2. **Associate** it with correct code path (Git repo and directory)
3. **Migrate** state using Terraform CLI (one-time activity)
4. **Validate** with clean plan to ensure no unexpected changes

**Critical Principle:**
"After migration, Terraform Cloud becomes the authoritative source for execution and state. There is no fallback to distributed backends."

**Validation Checklist:**
- ✓ Workspace created and properly named
- ✓ VCS connection established (GitHub repo linked)
- ✓ Variables populated correctly
- ✓ Secrets configured in TFC
- ✓ Clean plan verified (no changes after migration)
- ✓ Team access configured

---

## SLIDE 25: SUMMARY STATEMENT

"This migration addresses a known and growing operational risk around Terraform state consistency while modernizing governance, security, and auditability. By taking a phased, low-risk approach with HashiCorp partner support, we'll transition from fragmented, manual infrastructure operations to a centralized, automated, auditable system.

Operations teams will focus on strategy instead of firefighting. Engineering teams gain self-service access. Security and compliance teams get the visibility they need.

This is an investment in operational excellence and a foundation for future growth. With careful planning and a measured approach, we emerge with a modern, secure, scalable platform."

---

## SLIDE 26: Q & A

**Anticipated Questions & Answers:**

**Q: How long will this take?**
A: 6 months total. Phases 0-4 are required (~5 months); Phase 5 is optional optimization. Non-production environments go first (Phases 1-2), production migration happens in Phase 3.

**Q: Will this impact our current infrastructure?**
A: No. We're not redesigning infrastructure, just modernizing how we execute and manage Terraform.

**Q: What if something goes wrong?**
A: We have comprehensive rollback procedures. Non-prod can revert in 1 hour. Production can revert in 15 minutes. We maintain GitHub Actions as active fallback.

**Q: Do we need HashiCorp support?**
A: Yes, for Phases 0-1 primarily. This reduces architectural risk and ensures we follow best practices. By Phase 4, we operate independently.

**Q: What's the effort requirement?**
A: ~600 hours total over 6 months. Effort is front-loaded in Phases 0-1. Effort pays back within 3-4 months through operational efficiency gains.

**Q: Why migrate production last?**
A: We want to learn and refine on non-prod first. By the time we touch production, the process is proven and confidence is high.

---

## END OF SPEAKER NOTES

---

# RECOMMENDED SLIDE CONSOLIDATION

The presentation could be tightened from 26 slides to **18-20 slides** by consolidating:

**Consolidate These:**
1. **Slides 12-17 (Phases 0-5)** → Could be 1 visual slide with phases, details in speaker notes
2. **Slides 20-22 (Rollback/Contingency)** → Combine into 1-2 slides, full details in notes
3. **Slide 24 (Migration Principle)** → Move to notes, reference from workflow slide

**Recommended Streamlined Flow (18 slides):**
1. Title
2. High-Level Overview
3. GitHub Repositories & In-Scope
4. Current vs Ideal Workflows (combine 4-5 with diagram)
5. Workflow Comparison Table
6. Azure Subscription Mapping
7. What Changes / Doesn't Change
8. Terraform Cloud Architecture
9. Workspace Strategy & Challenge
10. TFC Migration Initiative Overview
11. Migration Approach
12. All Phases (1 visual slide, 5 with details in notes)
13. Timeline with Milestones
14. Effort & Cost-Benefit
15. Rollback Strategy (consolidated)
16. Contingency Scenarios
17. Risk Mitigation & Controls
18. Summary + Q&A

**Note:** This keeps the detailed explanations in speaker notes while making the presentation deck more visually digestible.

