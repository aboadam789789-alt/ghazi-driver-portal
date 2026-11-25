# EL-GHAZY — Master Project Register
**Version:** 0.1  
**Snapshot date:** 24 November 2026  
**Purpose:** A single factual register for the company's digital systems, deployments, database coverage, and next actions.

> This register distinguishes database structure, interface availability, deployment, and verified operation. An HTTP 200 response or a READY deployment is not proof that a workflow works end to end.

## 1. Executive status

- The repository is `aboadam789789-alt/ghazi-driver-portal`; its default branch is `main`.
- Current production deployment: `dpl_G8xXUwC9syMx3h19NvfjmcNnawCA`, READY, from `main` commit `c4bb7a6f72a1dcf0caafc3a99816f93e23485a4e`.
- The central workspace exists on `feature/central-workspace` and is deployed as a READY Preview. It is not in production. The register is stored in `docs/EL-GHAZY-MASTER-REGISTER.md` and linked from the portal.
- The project control center exists on `feature/project-control-center` and is deployed as a READY Preview. It is not in production.
- The two feature branches are independent workstreams; neither contains the other's changes. Keep both intact until an integration plan is reviewed.
- No production deployment, database schema change, or branch merge was performed during this audit.

## 2. Repository and deployment register

| Item | Source / commit | Environment | Current status | Verification |
|---|---|---|---|---|
| Driver portal, admin, HR, finance | `main` / `c4bb7a6f72a1dcf0caafc3a99816f93e23485a4e` | Production | Deployed | Root routes previously returned HTTP 200; workflows not end-to-end tested in this audit |
| EL-GHAZY CENTRAL | `feature/central-workspace` (portal + master register) | Preview | READY | Page source inspected; official logo path present; not production |
| Project Control Center | `feature/project-control-center` / `2155489b0e8a75b38f09fcdcbc3d938464a11da2` | Preview | READY | Page source inspected; prior calculation/trigger tests recorded; full UI workflow still needs user acceptance testing |

### Verified links

- Central workspace Preview: https://ghazi-driver-portal-git-feature-centr-2bd41b-aboadam789789-7874.vercel.app/central.html
- Project Control Center Preview: https://ghazi-driver-portal-clwnfl7oo-aboadam789789-7874.vercel.app/projects.html
- Production base: https://ghazi-driver-portal-kfohukzne-aboadam789789-7874.vercel.app/
- Vercel project: `ghazi-driver-portal` (`prj_Ex2nPNAw05D24T5anbaMI8Agxu8O`)

**Access note:** Vercel SSO protection is enabled for deployments except custom domains. The `vercel.app` URLs may therefore require an authorized Vercel session. Do not treat these as public employee-facing links until access policy and a company domain are reviewed.

## 3. Application modules

| Module | UI / source | Data foundation | Status | Next action |
|---|---|---|---|---|
| Driver portal | Root `index.html` on `main` | drivers, equipment, projects, operation_logs, Storage, Edge Function | Production route exists | Test login, manual entry, automatic start/finish, attachment upload, and history with an authorized test account |
| Operations/admin | Root `admin.html` on `main` | operation_logs, drivers, equipment, projects, profiles | Production route exists | Test review/approve/reject and role restrictions |
| HR | Root `hr.html` on `main` | HR tables | UI exists; HR data tables currently empty | Test employee setup, attendance, leave, advances, payroll draft/approval and cancellation using controlled test records |
| Project finance | Root `finance.html` on `main` | project contracts, receipts, budgets, cost views, audit_logs | UI exists; source tables currently empty | Test contract, receipt, budget and cost calculations with approved test data |
| Project Control Center | `projects.html` on `feature/project-control-center` | project_phases, project_tasks, project_task_progress_updates, project_action_items, progress view | Preview only | Test phase/task creation, progress submission, pending review, approval/rejection, and dashboard recalculation |
| EL-GHAZY CENTRAL | `central.html` on `feature/central-workspace` | Static navigation | Preview only | Keep as entry point; do not merge to production before branch integration and acceptance testing |

## 4. Supabase data inventory

**Project:** `csogzlzqinczlxdotfmb`  
**Schema reviewed:** `public`  
**Tables found:** 32. RLS is enabled on all 32 tables. The three public analytical views reviewed use `security_invoker=true`.

Exact row counts from the audit snapshot:

| Table / group | Rows |
|---|---:|
| equipment | 8 |
| equipment_types | 4 |
| drivers | 1 |
| projects | 2 |
| profiles | 5 |
| material_types | 3 |
| operation_logs | 0 |
| hr_employees | 0 |
| hr_attendance | 0 |
| project_phases | 0 |
| project_tasks | 0 |
| project_task_progress_updates | 0 |
| project_action_items | 0 |
| project_contracts | 0 |
| project_receipts | 0 |
| project_cost_budgets | 0 |
| equipment_expenses | 0 |
| suppliers | 0 |
| supply_deliveries | 0 |
| audit_logs | 2 |

The two active project names currently recorded are **مشروع قطعة ٧٥** and **٧٣**. The company's stated headcount is 15, but `hr_employees` currently contains zero records; this is a data-entry gap, not evidence that the company has no employees.

### Existing data domains (do not duplicate)

- Equipment and equipment types.
- Drivers and driver PIN attempt controls.
- Operating logs and operation attachments.
- Equipment expenses.
- HR: employees, departments, job titles, work sites, assignments, attendance, leave, advances, installments, salary records, payroll runs/items/adjustments.
- Projects, contracts, receipts, cost budgets, financial cost/summary views.
- Suppliers and supply deliveries/material types.
- Project phases, tasks, progress submissions, action items.
- Profiles and audit logs.

## 5. Security and audit observations

### Positive controls observed
- RLS is enabled on all 32 public tables.
- HR and project-finance table policies restrict access to `owner` and `system_admin`.
- Equipment and supply management policies include `equipment_manager` alongside owner/admin where configured.
- The project progress and financial views use `security_invoker=true`.
- Driver PIN helper functions are not executable by `anon` or `authenticated`; the authenticated driver-operation functions checked require a valid signed-in driver profile.
- The frontend contains a Supabase publishable key, not a service-role key. No service-role key was found in the reviewed pages.

### Gaps / hardening backlog
1. `audit_logs` contains only 2 rows at this snapshot. Triggers currently cover operation logs and project financial records (contracts, receipts, budgets), but not all important HR, equipment, supply, and project-control changes. Define and implement a complete audit coverage matrix.
2. `driver_pin_attempts` has RLS enabled and no direct table policies; confirm that all intended access occurs only through the restricted server-side functions.
3. Trigger-only functions in the exposed `public` schema retain default EXECUTE grants to PUBLIC/anon in at least two cases. They are not SECURITY DEFINER, and one returns trigger, but their grants should be narrowed as defense-in-depth after confirming no client workflow depends on direct RPC access.
4. The repository contains duplicate root and `src/` copies of `admin.html` and `hr.html`; they are not identical. The root files are the deployed paths, but the source-of-truth/build process is unclear. Consolidate the canonical source before major UI changes.
5. Vercel SSO protection may block ordinary employees from the current `vercel.app` URLs. Decide the intended company domain and access policy before rollout.

## 6. Digital assets and missing work

| Asset / capability | Current evidence | Status |
|---|---|---|
| Official company logo | `assets/el-ghazy-logo.webp` exists in repository; central page references it | Available |
| Official website | No confirmed domain or deployment found in this audit | Awaiting verified URL / scope |
| Company Profile / Previous Works | No approved final file linked in the central portal | Awaiting approved source material |
| Document management / archive | No verified module or repository route found | Proposed |
| CRM / clients / tenders | No verified module found | Proposed |
| Procurement / inventory | Suppliers, material types, and deliveries tables exist; no confirmed full procurement UI | Gap assessment required |
| HSE / quality | No verified module found | Proposed |
| Maintenance / asset lifecycle | Equipment data exists; no verified maintenance workflow found | Proposed |
| Backup / disaster recovery / cybersecurity | Not yet audited end-to-end | Required governance work |
| Technical submittals / drawings / approvals | No verified document-control workflow found | Proposed |
| Shared approvals / notifications / task inbox | Partial approval patterns exist in modules; no unified layer verified | Gap assessment required |

## 7. Priority execution plan

1. **Freeze the factual baseline:** keep this register updated; do not add fake operational data.
2. **Resolve repository source-of-truth:** compare root and `src/` copies, document the build/deploy process, and remove duplication only through a reviewed change.
3. **Create an integration Preview:** combine the central portal and project-control branch in a new integration branch; preserve both existing feature branches and do not merge to `main` without explicit approval.
4. **Run functional acceptance tests:** driver workflow, admin review, HR, finance, and project progress approval. Use controlled test records and remove them only through an approved cleanup plan.
5. **Close audit/security gaps:** audit trigger coverage, function grants, Storage policies, Edge Function authentication, and role matrix.
6. **Populate HR and project-control records from verified company documents:** no invented employees, salaries, phases, tasks, progress percentages, or financial figures.
7. **Connect official website and approved Company Profile:** only after their actual URLs/files and approved content are confirmed.
8. **Assess procurement, HSE, maintenance, DMS, CRM, backup, and technical document control:** reuse existing tables and workflows where suitable before creating new structures.

## 8. Change-control rules

- Preview first; production only after tests and explicit approval.
- No schema changes, destructive operations, production merges, or role changes without explicit approval.
- Never record planned progress as actual progress.
- Never treat HTTP availability as proof of business workflow correctness.
- Preserve the official logo exactly as supplied.


## 9. Integration Preview and source audit — 25 November 2026

- Created isolated branch `integration/central-project-control` from `feature/central-workspace`; original feature branches remain unchanged.
- Copied `projects.html` from `feature/project-control-center` (source blob `27f92b09714fb4369ac57f57fe97340243a00adf`) into the integration branch.
- Updated `central.html` so the Project Control card opens local `/projects.html` within the integration preview instead of the separate project-control preview.
- Vercel integration deployment: `dpl_3mW3BVoFCY8ZTdFb9xsMhtzpPzMj`, state **READY**, preview URL https://ghazi-driver-portal-4mqih77w5-aboadam789789-7874.vercel.app, branch alias https://ghazi-driver-portal-git-integration-c-e20ec4-aboadam789789-7874.vercel.app. Deployed commit: `843efdde54d7933a0230f398fc39cd5b05bc4660`.
- Source comparison: root `admin.html` and `hr.html` match between `main` and `feature/central-workspace`; their `src/` counterparts have different blob SHAs. Root and `src/` `index.html` are identical. No root `package.json` or `vercel.json` was found. This does not yet establish the intended long-term canonical source; no duplicate files were deleted.
- Integration branch is a Preview only. No PR merge, production deployment, Supabase schema change, or permission change was performed.
- Next: verify preview route behavior and SSO access, inspect Vercel project build/root settings, then run acceptance tests for project/task/progress/approval before requesting production approval.
