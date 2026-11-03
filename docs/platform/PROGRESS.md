# CSOC Platform progress

Read this first at the start of every session, and update it last before you stop.
The plan is in [PLAN.md](PLAN.md) and decisions are in [DECISIONS.md](DECISIONS.md).

Status key: `[ ]` todo · `[~]` in progress · `[x]` done · `[-]` dropped (say why)

## Current focus
**Phase 0** (de-Gen3) and the **Phase 1** shell and navigation, in parallel. Both are MVP steps 1–2.

## Phase 0 — De-Gen3
- [ ] Inventory every Gen3 reference: `grep -ril gen3` per directory. Record the baseline counts here.
- [ ] Design the plugin registration hook for the frontend (nav entries, pages) and the api (route groups)
- [ ] Move `frontend/components/StepperForm/*` and `serviceRegistry.js` → `plugins/gen3`
- [ ] Move `pages/sheepdog.jsx`, `pages/helm/gen3/*` and `pages/elasticsearch` → `plugins/gen3`
- [ ] Move `api/internal/keycloakadmin/commons.go` and `server/sql_*` → the gen3 plugin
- [ ] Move the defaults (`helm.gen3.org`, the `gen3-terraform//examples/csoc` module) into the gen3 blueprint config
- [ ] Add DCO, `GOVERNANCE.md` and `SECURITY.md`
- [ ] Verify: `grep -ril gen3` outside `plugins/gen3` returns 0, and the Gen3 deploy still works

## Phase 1 — UI redesign
- [ ] `docs/platform/DESIGN.md`: tokens, type, spacing, components, page templates
- [ ] New Mantine theme (dark-first, plus light)
- [ ] New app shell: pillar navigation, ⌘K palette, docked AI panel placeholder
- [ ] Detail-page template (status header, tabs, activity/PR timeline)
- [ ] Migrate pages one at a time (list them here as they're done, with screenshots)
- [ ] Turn `/bootstrap` into a general "Create environment" flow
- [ ] Get react-doctor clean

## Phase 2 — Any Helm, any Terraform, GitOps
- [ ] `Workspace` model (from `docker_runner.go`/`install_csoc.go`)
- [ ] Helm catalog (repos, OCI, Artifact Hub), with each release as an ArgoCD app via PR
- [ ] Values editor with `values.schema.json` validation
- [ ] Terraform outputs → Helm values
- [ ] Blueprints: `platform-core`, `aws-addons`, `gen3`

## Phase 3 — Any cloud, credentials
- [ ] `CloudAccount` and credential broker (AWS SSO/AssumeRole, GCP WIF, Azure federated)
- [ ] Permission pre-check
- [ ] `SecretBackend` → ESO `ClusterSecretStore` (AWS SM, GCP SM, Azure KV, Vault, k8s)
- [ ] `ClusterProvider`: Terraform/EKS, Import, k3s SSH, CAPI
- [ ] Pluggable ingress, DNS and TLS

## Phase 4 — AI first-line SRE
- [ ] Observability baseline (kube-prometheus-stack, Loki, default rules)
- [ ] MCP server (read tools, `propose_change`, `notify`, `silence`; RBAC and audit)
- [ ] Alertmanager webhook → triage → Slack/PagerDuty
- [ ] Plan and drift review
- [ ] Chat panel and change authoring
- [ ] Pluggable LLM provider

## Phase 5 — SaaS + self-hosted
- [ ] `CSOC_MODE` flag and tenant ID on every model
- [ ] Hosted LLM metering and billing hooks

## Phase 6 — CNCF Sandbox
- [ ] Adopters, a second organization's maintainer, security self-assessment, application

## Open decisions
- The final product name (CSOC for now)
- The image registry and Go module path rename (deferred)
- GitLab support, and when

## Session log
- **2026-11-03 — Session 1:**
  - Forked to `~/projects/jawadqur/csoc`, branch `feat/csoc-platform` from gen3-admin `6cff26b`.
  - Wrote the PLAN, PROGRESS and DECISIONS files and `CLAUDE.md`.
  - No code changes.
