# CSOC Platform decisions

Add new entries at the bottom. Don't rewrite old entries: supersede them with a new entry that links back.

## D1 — Work in a fork, keep the name CSOC (2026-11-03)
- **Where:** the work happens in `jawadqur/gen3-admin`, branch `feat/csoc-platform`, checked out at `~/projects/jawadqur/csoc`.
- **Why a fork:** the Gen3 workshop branch (`feat/csoc-bootstrap-wizard` in uc-cdis) must stay undisturbed.
- **Name:** stays "CSOC" until there's a product name. Renaming the image registry and Go module path is deferred.

## D2 — Product first, CNCF later (2026-11-03)
- **The order:** build the generic product and win outside adopters before applying to the CNCF Sandbox.
- **Why:** Sandbox reviewers look for neutrality and more than one organization, and a fork with one maintainer isn't ready.
- **What this means now:** build on CNCF projects (Argo, ESO, CAPI, cert-manager, Prometheus, OTel) rather than reimplementing them.

## D3 — Single Apache-2.0 codebase with a SaaS mode flag (2026-11-03)
- **How it works:** `CSOC_MODE=selfhosted|saas`. SaaS turns on multi-tenancy, hosted-LLM metering and billing hooks.
- **Why not open core:** a single codebase is simpler and friendlier to the CNCF. Revisit this only if a feature truly can't be open.

## D4 — AI can only propose changes (2026-11-03)
- **What AI can do:** read through the agent tunnel, open PRs (`propose_change`), notify, and set time-boxed silences.
- **What it can't do:** apply anything to clusters or clouds.
- **Controls:** every tool call is checked against Keycloak role allowlists and audit-logged.
- **Model choice:** the LLM provider is pluggable. It defaults to Claude (`claude-sonnet-5-5` for triage and chat, `claude-opus-5-5` for deep review) and stays off until configured.

## D5 — CAPI for new managed providers, Terraform for cloud foundations (2026-11-03)
- **Clusters:** GKE, AKS and k3s are provisioned through Cluster API providers instead of hand-written per-cloud Terraform. The existing EKS Terraform path is kept.
- **Foundations:** Terraform still owns what CAPI doesn't cover: state buckets, DNS, IAM, and any user module.
- **Exception:** k3s over SSH (k3sup-style) is allowed as the fast on-prem path.

## D6 — GitOps by default (2026-11-03)
- **Rule:** every change to Helm releases, Terraform workspaces or blueprints lands as a PR through the GitHub App (`api/internal/gitops/github.go`).
- **Escape hatch:** direct mode (`INSTALL_MODE=direct`) stays, but isn't the default.
