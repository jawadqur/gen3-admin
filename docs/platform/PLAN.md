# CSOC Platform plan

> Multi-session roadmap for turning CSOC (Gen3 admin) into a general platform.
> Status lives in [PROGRESS.md](PROGRESS.md); decisions live in [DECISIONS.md](DECISIONS.md).

## Pitch

Deploy **any Helm chart and any Terraform module into any cloud** (or k3s, or bare metal):

- **GitOps:** every change is a PR.
- **Credentials:** the platform handles the hard part.
- **AI:** built in, for alerting and as a **first-line SRE**.

It is open source (Apache-2.0) and self-hostable, and will also be offered as a hosted SaaS. The order is product first, then a CNCF Sandbox application once there are adopters.

The name stays **CSOC** for now. The UI gets a redesign (Phase 1).


## What the audit found (gen3-admin @ 6cff26b)
- **About 70% of the code is generic:**
  - the agent/gRPC mTLS tunnel (`api/gen3-agent`, `api/internal/tunnel`)
  - the argocd, helm, k8s and auth packages
  - the GitOps PR engine (`api/internal/gitops/github.go`)
  - the Terraform-in-Docker runner (`api/internal/terraform/docker_runner.go`)
  - frontend: the argocd, clusters, shell, observability and cloud pages
- **Gen3 coupling is limited to a few places:**
  - `components/StepperForm/*` and `serviceRegistry.js`
  - `pages/sheepdog.jsx`, `pages/helm/gen3/*`, `pages/elasticsearch`
  - `keycloakadmin/commons.go` and `server/sql_*`
  - the defaults `helm.gen3.org`, `gen3-terraform//examples/csoc` and `quay.io/cdis/csoc-*`
- **Only AWS works.** GCP and Azure exist only as UI radio buttons and runner allowlist entries.
- **Seams that can be extended:**
  - `moduleSources()` (`docker_runner.go:188`)
  - `ChartSource` (`gitops/csoc.go:49`)
  - `gitops/templates/csoc/`
  - `environments.Kind` (`registry.go:38`)
  - the allowlists in `runner/runner.go:33`
- **License:** Apache-2.0 is fine. The cloud icons in `third_party/tf-arch` are under the vendors' own terms and need a review.

## Positioning
- **The neighbours:**
  - Rancher does clusters only.
  - Spacelift, env0 and Atlantis do Terraform only.
  - Argo and Flux do Helm only.
  - Kubefirst is a bootstrap without an SRE layer.
- **What CSOC covers end to end:** cloud credentials → Terraform → cluster → Helm → alerts → AI triage, with every change a PR.
- **Build on CNCF projects rather than rewrite them:** Argo, ESO, CAPI, cert-manager, Prometheus and OpenTelemetry.

## Phase 0 — De-Gen3 (keep the CSOC name)
- **Move Gen3 code into `plugins/gen3/`** behind a frontend and api registration hook:
  - StepperForm and serviceRegistry
  - the sheepdog, elasticsearch and helm/gen3 pages
  - `keycloakadmin/commons.go` and `sql_*`
- **Move hard-coded defaults** (gen3-terraform module, `helm.gen3.org`) into the gen3 blueprint.
- **Defer the image registry and Go module path.** Leave them for now; renaming them comes later.
- **Add** DCO, `GOVERNANCE.md` and `SECURITY.md`.

## Phase 1 — UI redesign (alongside Phase 0, since every later pillar needs the new surfaces)
- **Direction:** a modern infra console. Dense, calm and dark-first (light theme too), in the spirit of Linear, Vercel and Grafana. Keep Mantine, with a new theme and tokens.
- **Navigation by pillar:**
  - **Overview:** fleet health, active incidents, open PRs, drift.
  - **Deploy:** Helm releases, Terraform workspaces, blueprints.
  - **Clouds & Credentials:** accounts, permission checks, secret backends.
  - **Clusters:** import, k3s, managed, CAPI.
  - **GitOps:** PRs and ArgoCD apps.
  - **Alerts & SRE:** incidents, AI investigations, rules.
  - **Settings**
- **Shared layout:**
  - A ⌘K command palette.
  - An AI panel docked on the right on every page, which knows the context of what's on screen.
  - A consistent detail-page pattern: header with status, tabs, and an activity/PR timeline.
- **Rework the `/bootstrap` wizard** into a general "Create environment" flow: pick provider → credentials → blueprint → review the plan and PR.
- **Write a design system doc** (`docs/platform/DESIGN.md`) covering tokens, components and page templates.
- **Use the dataviz rules** for charts.
- **Run react-doctor clean.**
- **Approach:** add the new shell and navigation first, then migrate pages one at a time, with the Gen3 pages living under the plugin's nav entry.

## Phase 2 — Pillar 1: any Helm, any Terraform, GitOps
- **A `Workspace` model** generalizing `docker_runner.go` and `install_csoc.go`:
  - It holds the module source (git, registry or OCI), version, vars, backend (S3, GCS or Azure Blob) and a credential binding.
  - It keeps the existing plan → diagram → approval → apply flow.
- **A Helm catalog** generalizing `gitops/csoc.go` and `helm_chart_values.go`:
  - Any Helm repo or OCI registry, plus Artifact Hub search.
  - Each release becomes an ArgoCD Application that lands as a PR.
  - Values are edited in the Monaco wrapper and validated against the chart's `values.schema.json`.
- **Outputs → inputs:** Terraform outputs are passed into Helm values through ESO-synced secrets or ConfigMaps.
- **Blueprints:** `gitops/templates/csoc/` becomes versioned bundles: `platform-core`, `aws-addons`, and `gen3` as the reference blueprint.
- **GitOps by default:** every change is a PR (GitHub App now; GitLab later). Direct mode stays as an escape hatch.

## Phase 3 — Pillar 2: any cloud, credentials handled
- **A `CloudAccount` model and credential broker**, keyless first:
  - AWS: SSO and AssumeRole; `internal/aws` already detects SSO expiry.
  - GCP: workload identity federation.
  - Azure: federated credentials.
  - Static keys only as a fallback, kept in ESO-backed stores and never in the platform database.
- **A permission pre-check** before each run: what the module needs versus what the role has.
- **A `SecretBackend` interface that generates the ESO `ClusterSecretStore`:** AWS Secrets Manager (exists), GCP Secret Manager, Azure Key Vault, Vault, and Kubernetes secrets.
- **A `ClusterProvider` interface (`api/internal/provider`):**
  - **Terraform/EKS:** wraps the existing path.
  - **Import:** a join token signed by `api/internal/ca`.
  - **k3s over SSH:** one server or three for HA, plus agents.
  - **CAPI:** CAPG/CAPZ/CAPA and the k3s provider, for GKE, AKS and more.
- **Pluggable ingress, DNS and TLS:** ALB + ACM, or ingress-nginx/Traefik + cert-manager + external-dns.
- **Replace `environments.Kind` and the runner allowlists** with provider, blueprint and account IDs.

## Phase 4 — Pillar 3: AI first-line SRE (AI can only propose: it reads, opens PRs and pages people; it never applies)
1. **Observability baseline in `platform-core`.**
   - Ship kube-prometheus-stack and Loki.
   - Ship default rules for nodes, pods, ArgoCD, cert expiry, ESO sync and Terraform drift.
2. **An MCP server (`api/internal/mcp`).**
   - **Read tools through the tunnel:** pods, events, logs, metrics, ArgoCD diffs, plan and state outputs, and recent PRs and deploys.
   - **Action tools:** `propose_change` (opens a PR), `notify`, and a time-boxed `silence`.
   - Keycloak roles map to the tool allowlists, and every call is audited.
3. **Alert triage.**
   - An Alertmanager webhook starts an investigation that correlates the alert with deploys, PRs and drift.
   - It then posts the evidence, a likely cause, a runbook step and an optional fix PR to Slack, PagerDuty or email.
   - It de-duplicates repeats into incidents and escalates to a person when confidence is low.
4. **Plan/drift review.** Summarize `terraform show -json` and ArgoCD diffs, and flag destructive or IAM/network changes. A scheduled drift check opens an alert or a PR.
5. **A chat and troubleshooting panel** (the docked AI panel from Phase 1), plus change authoring: a natural-language request becomes a values or tfvars PR.
6. **Pluggable LLM.**
   - Default to Claude: `claude-sonnet-5-5` for triage and chat, `claude-opus-5-5` for deep investigation and plan review.
   - Also support OpenAI-compatible and local endpoints.
   - Off until configured, with per-tenant budgets.

## Phase 5 — SaaS + self-hosted
- **One Apache-2.0 codebase with a `CSOC_MODE=selfhosted|saas` flag.**
- **SaaS mode turns on:**
  - multi-tenancy (an org/tenant ID on every model, with Keycloak realm or organization per tenant)
  - a hosted LLM with metering
  - billing hooks
  - agent-only connectivity, so customer clusters dial out to the SaaS and no inbound access is needed (the existing tunnel already works this way)
- **Self-hosted:** the Helm chart plus docker-compose, as today. It brings its own LLM key.
- **If an open-core split is ever wanted,** it can be revisited in `DECISIONS.md`.

## Phase 6 — Adoption → CNCF Sandbox
Get two or more outside adopters and a second organization's maintainer. Then publish a public roadmap and security self-assessment, and apply.

## MVP order (the smallest thing that demos the whole pitch)
1. Phase 0
2. The Phase 1 shell and navigation
3. Import cluster
4. Any Helm → PR → ArgoCD
5. Any Terraform with an AWS + GCP credential broker
6. Prometheus + Alertmanager → AI triage to Slack

After that: k3s SSH, CAPI, Azure, change-authoring chat, SaaS tenancy.

## Critical files
- `api/internal/terraform/docker_runner.go`, `install_csoc.go`: become the `Workspace` model and the first `ClusterProvider`
- `api/internal/gitops/csoc.go`, `github.go`, `templates/csoc/`: the Helm catalog, PR engine and blueprints
- `api/internal/aws/*`: the AWS half of the credential broker
- `api/internal/environments/registry.go`, `api/internal/runner/runner.go`: the registry
- `api/internal/ca`, `api/gen3-agent`: join token for import
- `scripts/runner/install-csoc.sh`, `plan-diagram.mjs`: blueprint steps; plan JSON feeds the AI review
- `frontend/pages/_app.*`, the layout and nav components, `pages/bootstrap.jsx`, `components/bootstrap/*`: the new shell and "Create environment" flow
- `frontend/components/StepperForm/*`, `pages/sheepdog.jsx`, `pages/helm/gen3/*`: move to `plugins/gen3`

## Verification (per phase)
- **De-Gen3:** `grep -ril gen3` outside `plugins/gen3` returns 0, and the Gen3 deploy flow still works through the plugin.
- **UI:** every migrated page renders in both themes, ⌘K reaches every page, react-doctor is clean, and screenshots of each page go into `PROGRESS.md`.
- **Any Helm:** bitnami/redis from OCI deploys to an imported k3d cluster. A PR opens, ArgoCD syncs, and schema errors show in the editor.
- **Any Terraform:** run `terraform-aws-modules/s3-bucket` on the workshop account (never `csoc.planx-pla.net`) and feed an output into a Helm release. Tear down with Destroy → Force.
- **Credentials:** a GCP workload identity federation run with no stored key succeeds, and the pre-check catches a missing permission.
- **AI SRE:**
  - A bad image tag fires an alert. Triage names the PR that caused it and proposes the revert PR.
  - A read-only role can't call `propose_change`.
  - A destroyed-VPC plan gets flagged.
- **SaaS:** two tenants can't see each other's clusters, accounts or incidents.
