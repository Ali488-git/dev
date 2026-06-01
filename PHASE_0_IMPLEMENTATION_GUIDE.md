# PHASE 0 IMPLEMENTATION GUIDE: DISCOVERY & VALIDATION WITH COPILOT
## Step-by-Step Implementation for Terraform Cloud Migration

---

## PHASE 0 OVERVIEW (Weeks 1-3)

**Duration:** 3 weeks  
**Effort with Copilot:** 56 hours (~2 weeks of 1 person, or 1 week of 2 people)  
**Goal:** Map current infrastructure and validate migration strategy with zero guesswork

**Deliverables:**
1. ✅ Complete state inventory (all repos, backends, environments)
2. ✅ Workspace consolidation strategy (60+ dirs → 24-30 workspaces)
3. ✅ HashiCorp validation & recommendations
4. ✅ Migration roadmap with risk assessment
5. ✅ Team readiness assessment

---

## STEP 1: INVENTORY CURRENT STATE (6 hours, mostly Copilot-assisted)

### 1.1 Identify All Repositories & Backends (1 hour manual + 0.5h Copilot)

**Your Task:**
- Open Azure-IaC repository → Document structure
- Open AZ Policy Repo → Document structure
- List all backend configurations currently in use

**Copilot Prompt:**
```
"I have two GitHub repositories for Terraform infrastructure:
1. Azure-IaC (contains 60+ workload directories across Dev/SIT/UAT/Prod)
2. AZ Policy Repo (contains Azure policy definitions)

Both currently use Azure Storage backends with templated configurations.
Generate a bash script that:
- Lists all Terraform state files in the specified Azure Storage accounts
- Exports metadata (state file name, size, last modified, backend config)
- Creates a CSV report with columns: Repository, StateFile, Environment, Size, LastModified
- Groups results by environment and backend"
```

**Expected Output:** CSV inventory file  
**Your Role:** Review CSV, manually verify a few state files to validate accuracy

---

### 1.2 Map Workspace Directories (2 hours manual + 1h Copilot)

**Your Task:**
- Analyze Azure-IaC directory structure
- Identify 60+ workload directories
- Categorize by function/team (networking, database, compute, storage, security)

**Copilot Prompt:**
```
"I have a Terraform monorepo with 60+ directories. Show me a script that:
1. Scans the repository for all directories containing main.tf files
2. Groups directories by functional category (infer from names: networking, database, 
   compute, storage, security, management, etc.)
3. Creates a report showing:
   - Directory path
   - Inferred function/team
   - Count of resources (by parsing main.tf)
   - Current backend configuration
   - Environment (dev/sit/uat/prod)

Output: CSV file with columns: Directory, Function, Environment, Backend, ResourceCount"
```

**Expected Output:** Categorized directory mapping  
**Your Role:**
- Review groupings (do they make sense?)
- Adjust categories if needed (e.g., merge related directories)
- Validate environment assignments

---

### 1.3 Document Current Backend Configurations (1.5 hours manual + 0.5h Copilot)

**Your Task:**
- List all Azure Storage accounts used for state
- Document backend configurations (container names, key patterns)
- List all environment variables and secrets used

**Copilot Prompt:**
```
"Generate a Python script that scans Terraform code for backend configurations:
1. Find all backend blocks (backend 'azurerm' or backend 'local')
2. Extract backend parameters: storage_account_name, container_name, key, resource_group
3. For each unique backend, identify which repos/directories use it
4. Check for environment variables passed to backends
5. Generate report showing:
   - Backend type and location
   - Directories using this backend
   - Environment count
   - State file count estimate

Output: JSON structured report"
```

**Expected Output:** Backend configuration report  
**Your Role:**
- Validate Azure Storage accounts and containers exist
- Confirm environment variables match your actual setup

---

### 1.4 Create State Files Inventory (1 hour manual + 0.5h Copilot)

**Your Task:**
- For each backend, list all state files
- Document state file size and metadata

**Copilot Prompt:**
```
"Using Azure CLI commands, generate a bash script that:
1. Connects to Azure Storage accounts (list provided)
2. For each container, lists all state files (*.tfstate)
3. Extracts metadata: name, size, last modified date
4. Counts resources in each state file
5. Identifies state dependencies (refs to other states)
6. Generates summary: Total states, Total environments, Total resources managed
7. Creates CSV: StateFile, Size(MB), LastModified, Environment, ResourceCount

Include error handling and logging."
```

**Example Script Output:**
```
State,Size(MB),LastModified,Environment,ResourceCount
azure-dev-networking.tfstate,2.3,2026-05-29,dev,15
azure-dev-compute.tfstate,1.8,2026-05-25,dev,42
azure-sit-networking.tfstate,2.4,2026-05-28,sit,16
...
```

**Your Role:** Run script, validate numbers make sense

---

## STEP 2: ANALYZE WORKSPACE STRATEGY (4 hours, mostly you + some Copilot)

### 2.1 Understand Current Workspace Patterns (1 hour manual)

**Your Task:**
- Review Azure Products IaC (already on TFC) for workspace patterns
- Check Terraform Cloud: app.terraform.io/app/metrolinx/workspaces
- Document how it organizes projects/workspaces

**What to Look For:**
- How many workspaces per environment?
- How are they named?
- What's the state isolation pattern?
- How are variables managed?

**Copilot Prompt:**
```
"I'm planning to consolidate 60+ Terraform directories into Terraform Cloud workspaces.
My existing TFC setup has workspaces organized as:
- Organization: metrolinx
- Projects: per-application (database-app, networking-core, compute-services)
- Workspaces: per-resource-type per-environment (db-server-prod, db-backup-prod)

Based on this pattern, recommend how to organize Azure-IaC (60+ dirs):
1. Suggest project groupings (cluster related directories)
2. Propose workspace naming convention
3. Calculate ideal number of workspaces (target: 24-30)
4. Show mapping: Directory → Project → Workspace

Consider: Team ownership, blast radius, infrastructure dependencies"
```

---

### 2.2 Design Workspace Consolidation Strategy (2 hours manual + 1h Copilot)

**Your Task:**
- Consolidate 60+ directories into 24-30 workspaces
- Group by function (not 1:1 mapping)

**Copilot Prompt:**
```
"Given these directory categories and counts:
- Networking: 8 directories (VNets, subnets, NSGs, firewalls)
- Database: 12 directories (SQL, PostgreSQL, backups, replication)
- Compute: 15 directories (VMs, scale sets, load balancers)
- Storage: 8 directories (Storage accounts, blobs, file shares)
- Security: 7 directories (Key Vault, managed identities, policies)
- Management: 10 directories (monitoring, logging, automation)

Create a workspace consolidation plan:
1. Group directories by resource type (not 1:1)
2. Create per-environment instances (Dev, SIT, UAT, Prod)
3. Propose workspace names following convention: team-function-environment
4. Map each directory to a workspace
5. Estimate state file size per workspace
6. Identify potential dependencies between workspaces

Output: Markdown table showing mapping"
```

**Example Output:**
```
| Workspace Name | Function | Directories Included | Est. Resources | Dev/SIT/UAT/Prod |
|---|---|---|---|---|
| core-networking-* | Core network infra | VNets, subnets, NSGs | 20-25 | 4 envs |
| app-database-* | Application DB | SQL, backups, replicas | 30-40 | 4 envs |
| app-compute-* | Compute resources | VMs, scale sets, LBs | 40-50 | 4 envs |
| shared-storage-* | Shared storage | Storage accounts, blobs | 15-20 | 4 envs |
| security-keyvault-* | Secrets & security | Key Vault, identities | 10-15 | 4 envs |
| mgmt-monitoring-* | Ops & monitoring | Monitor, logs, alerts | 20-25 | 4 envs |
```

**Your Role:**
- Review groupings with team leads
- Adjust if teams have ownership concerns
- Validate dependencies aren't overlooked

---

### 2.3 Create Detailed Workspace Mapping Document (1 hour manual + Copilot)

**Copilot Prompt:**
```
"Create a comprehensive workspace mapping document that includes:

1. For each proposed workspace:
   - Name and purpose
   - List of directories consolidated
   - Expected resource count
   - Team ownership
   - Scheduling/deployment order (dependencies)

2. For each environment (Dev, SIT, UAT, Prod):
   - List all workspaces
   - State file names
   - Variable inheritance hierarchy
   - Rollback sequence (if needed)

3. Dependency diagram (text-based):
   - Show which workspaces depend on others
   - Mark critical dependencies (e.g., networking)
   - Suggest deployment order

Format: Markdown with tables and ASCII diagrams"
```

---

## STEP 3: HASHICORP PARTNER VALIDATION (8 hours, mostly meetings + Copilot docs)

### 3.1 Prepare Documentation Package (1 hour manual + 1h Copilot)

**Your Task:**
- Compile all Phase 0 findings
- Create clear presentation of workspace strategy

**Copilot Prompt:**
```
"Create a presentation outline for HashiCorp consultants reviewing our 
Terraform Cloud migration plan:

1. Current State:
   - 2 repositories, 60+ directories, 4 environments
   - Current backend: Azure Storage
   - Current deployment: GitHub Actions

2. Proposed State:
   - Workspace consolidation strategy (24-30 workspaces)
   - Environment structure
   - Team ownership model

3. Key Questions for HashiCorp:
   - Does our workspace structure align with TFC best practices?
   - Are there potential issues with our consolidation approach?
   - What sizing/limits should we be aware of?
   - Any security/compliance considerations?
   - Recommended state migration sequence?

Generate: Agenda with timing, talking points, success criteria"
```

**Documents to Prepare:**
1. Current state inventory (CSV from Step 1.4)
2. Directory mapping (CSV from Step 1.2)
3. Proposed workspace structure (table from Step 2.3)
4. Infrastructure diagram (current → target)

---

### 3.2 HashiCorp Kickoff Meeting (2 hours, you lead)

**Agenda:**
- Week 1, Tuesday: 1-hour kickoff call with HashiCorp
- Present current state & proposed strategy
- Gather initial feedback

**Your Preparation:**
- 30 min: Current state deep dive
- 15 min: Workspace strategy walkthrough
- 15 min: Q&A and feedback

---

### 3.3 HashiCorp Architecture Review (Async, 1 week)

**What HashiCorp Does:**
- Reviews your workspace structure
- Validates against best practices
- Identifies risks or anti-patterns
- Suggests improvements

**Your Role:**
- Prepare answers to HashiCorp's questions
- Provide access to repositories if needed
- Document their recommendations

**Copilot Prompt (while waiting):**
```
"Based on Terraform Cloud best practices, create a pre-flight checklist 
for our migration:

1. Organization Setup:
   - [ ] Correct team structure in place
   - [ ] RBAC roles defined (admin, developer, operator)
   - [ ] SSO planned (Azure AD integration)

2. Project & Workspace Design:
   - [ ] Projects match business teams/applications
   - [ ] Workspace naming convention documented
   - [ ] Variable inheritance strategy planned
   - [ ] State file isolation strategy validated

3. Migration Safety:
   - [ ] Backup plan for all current states
   - [ ] Rollback procedures documented
   - [ ] Dry-run testing planned
   - [ ] Communication plan for stakeholders

4. Team Readiness:
   - [ ] Training needs identified
   - [ ] Documentation planned
   - [ ] Support model (on-call) designed

Format: Markdown checklist with brief descriptions"
```

---

### 3.4 Incorporate HashiCorp Feedback (2 hours, you + Copilot)

**Your Task:**
- Review HashiCorp recommendations
- Adjust workspace strategy if needed
- Document decisions

**Copilot Prompt:**
```
"I received feedback from HashiCorp on our workspace consolidation strategy.
[Paste their recommendations here]

Please help me:
1. Identify which recommendations are critical vs. nice-to-have
2. Propose how to implement each recommendation
3. Document any trade-offs or risks from accepting/rejecting recommendations
4. Update the workspace mapping document with approved changes
5. Create an action items list for the team

Output: Decision log with rationale"
```

---

## STEP 4: MIGRATION ROADMAP CREATION (5 hours, mostly Copilot + you verify)

### 4.1 Design Phased Migration Plan (1 hour manual + 1h Copilot)

**Copilot Prompt:**
```
"Based on our workspace strategy, create a detailed Phase 1-4 migration roadmap:

Context:
- 2 repositories (Azure-IaC, AZ Policy)
- 4 environments (Dev, SIT, UAT, Prod)
- 24-30 target workspaces
- Zero-downtime requirement

For each phase, document:
1. Scope: Which workspaces/environments
2. Duration: Weeks required
3. Effort: Estimated hours
4. Risks: What could go wrong
5. Success criteria: How do we know it's working
6. Rollback plan: How to revert if needed
7. Communication: Who needs to be notified

Output: Markdown roadmap with timelines"
```

---

### 4.2 Create Risk Assessment Matrix (1 hour manual + 1h Copilot)

**Copilot Prompt:**
```
"Create a risk matrix for the Terraform Cloud migration:

Format for each risk:
- Risk: What could go wrong
- Likelihood: High/Medium/Low
- Impact: High/Medium/Low
- Mitigation: How to prevent/reduce
- Contingency: What to do if it happens

Include risks like:
- State corruption during migration
- Access/authentication failures
- Team not trained properly
- Production downtime
- Rollback failures
- Secrets exposure
- Resource naming conflicts

Output: Prioritized risk list with mitigation strategies"
```

---

### 4.3 Draft Migration Roadmap Document (2 hours manual + Copilot assist)

**Copilot Prompt:**
```
"Create a comprehensive migration roadmap document for stakeholders:

Sections:
1. Executive Summary
   - What: Terraform Cloud migration
   - Why: Benefits (security, governance, automation)
   - When: 6-month timeline with milestones
   - Who: Teams involved, roles, responsibilities
   - Cost: Effort and resources required

2. Detailed Phases (0-5)
   - Phase 0: Discovery (this phase)
   - Phase 1-4: Implementation phases
   - Phase 5: Optimization

3. Timeline
   - Week-by-week schedule
   - Critical milestones with go/no-go gates
   - Communication checkpoints

4. Risk Management
   - Top 10 risks and mitigations
   - Contingency plans
   - Escalation procedures

5. Success Criteria
   - How we measure success
   - Monitoring during migration
   - Post-migration validation

Format: Professional document suitable for executive review"
```

---

## STEP 5: TEAM READINESS ASSESSMENT (3 hours, mostly you)

### 5.1 Assess Current Knowledge (1 hour manual)

**Your Task:**
- Survey team on Terraform Cloud experience
- Identify training gaps

**Copilot Prompt:**
```
"Create a training needs assessment survey for our team:

Questions to gauge:
1. Terraform basics knowledge (5-point scale)
2. Terraform Cloud experience (none/limited/moderate/advanced)
3. GitHub/Git experience
4. Azure infrastructure knowledge
5. CI/CD pipeline experience
6. Team role and responsibilities

Output: Simple survey (5-10 questions) + scoring rubric"
```

---

### 5.2 Create Training Plan (1 hour manual + 1h Copilot)

**Copilot Prompt:**
```
"Based on our team assessment, create a training roadmap:

Topics to cover:
1. Terraform Cloud fundamentals (workspace, state, runs)
2. Migration process and our specific approach
3. New workflow (ServiceNow → Ansible → TFC)
4. Troubleshooting common issues
5. On-call procedures and escalation

For each topic:
- Duration (minutes/hours)
- Format (video, hands-on lab, documentation, workshop)
- Audience (all engineers, ops only, leads only)
- When to deliver (which phase)
- Success metrics (test/certification)

Output: Training calendar and content outline"
```

---

## STEP 6: VALIDATION & SIGN-OFF (2 hours, you + stakeholders)

### 6.1 Validate Phase 0 Deliverables

**Checklist:**
- ✅ State inventory complete (all repos, all backends documented)
- ✅ Workspace strategy validated with team
- ✅ HashiCorp recommendations incorporated
- ✅ Migration roadmap documented
- ✅ Risk matrix reviewed
- ✅ Team training plan created
- ✅ Effort estimates realistic (56h with Copilot)

### 6.2 Get Stakeholder Sign-Off

**Meetings:**
- Meet with Infrastructure Lead: Validate technical approach
- Meet with Operations: Confirm timeline works
- Meet with Management: Approval for Phase 1 start

**Approval Document:**
```
"Phase 0 is APPROVED to proceed. Key decisions:
- Workspace consolidation: 24 workspaces (from 60+ dirs)
- Migration sequence: Dev → SIT → UAT → Prod
- HashiCorp support: Phases 0-2
- Phase 1 start date: [Date]
- Go/no-go criteria: All Phase 0 deliverables complete"
```

---

## PHASE 0 DELIVERABLES CHECKLIST

**By End of Week 3:**

✅ **Inventory Documents:**
- [ ] State files inventory (CSV)
- [ ] Directory mapping (CSV)
- [ ] Backend configurations documented
- [ ] Subscription mapping validated

✅ **Architecture Documents:**
- [ ] Proposed workspace structure (detailed)
- [ ] Workspace mapping (directories → workspaces)
- [ ] Dependency diagram
- [ ] Team ownership matrix

✅ **Validation Documents:**
- [ ] HashiCorp recommendations + responses
- [ ] Risk assessment matrix
- [ ] Migration sequence & gates

✅ **Planning Documents:**
- [ ] Detailed migration roadmap (Phases 1-5)
- [ ] Phase 1 execution plan (ready to start Week 4)
- [ ] Team training calendar
- [ ] Communication plan

✅ **Sign-Off:**
- [ ] Infrastructure Lead approval
- [ ] Operations approval
- [ ] Management/Budget approval
- [ ] Phase 1 kickoff scheduled

---

## EFFORT SUMMARY (Phase 0 with Copilot)

| Task | Manual | Copilot | Total |
|------|--------|---------|-------|
| Step 1: Inventory | 5h | 2.5h | 7.5h |
| Step 2: Workspace Strategy | 4h | 1.5h | 5.5h |
| Step 3: HashiCorp Validation | 8h | 2h | 10h |
| Step 4: Roadmap Creation | 4h | 3h | 7h |
| Step 5: Team Assessment | 3h | 1.5h | 4.5h |
| Step 6: Validation & Sign-off | 2h | 0.5h | 2.5h |
| **TOTAL** | **26h** | **11h** | **37h** |

**Efficiency Gain: 37h with Copilot vs. 56h manual = -34% effort reduction**

---

## SUCCESS CRITERIA FOR PHASE 0

✅ **Complete Understanding:**
- You can draw the current state from memory
- You understand dependencies between workspaces
- You know why each workspace grouping makes sense

✅ **Stakeholder Alignment:**
- Team leads agree with workspace assignments
- Infrastructure lead validates technical approach
- HashiCorp has signed off on strategy

✅ **Ready for Phase 1:**
- Phase 1 team knows exactly what to build
- TFC organization structure is designed
- No surprises in next phase

---

## KEY COPILOT TIPS FOR PHASE 0

1. **Be Specific:** Provide actual repo names, directory counts, environment names
2. **Ask for Formats:** Request CSV, JSON, Markdown, or ASCII diagrams
3. **Iterate:** Ask Copilot to refine/adjust based on feedback
4. **Verify:** Always check Copilot's output matches reality
5. **Document:** Save all Copilot outputs—they become your knowledge base

