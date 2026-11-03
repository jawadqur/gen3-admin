# CLAUDE.md — CSOC Platform fork

This repo is a **fork** of gen3-admin that is being generalized into a platform. The goal: deploy any Helm chart and Terraform module into any cloud, through GitOps, with credentials handled and an AI first-line SRE. The work spans many sessions.

## Start and end of every session
1. **Read first:** `docs/platform/PROGRESS.md` (current focus and status), then `docs/platform/PLAN.md` and `docs/platform/DECISIONS.md`.
2. **Work on** the "Current focus" in PROGRESS.md unless the user says otherwise.
3. **Update before you stop:**
   - Tick the checkboxes.
   - Add a session-log line (date, what changed, what's next).
   - Record any new decision in DECISIONS.md.

## Repo facts
- **Branch:** `feat/csoc-platform`. Remote `origin` is `jawadqur/gen3-admin` (the fork). Remote `gen3-local` is the local uc-cdis checkout, which only ever needs fetching.
- **Never push to `uc-cdis`.** Don't push to `gen3-local` either.
- **Stack:**
  - Go API (`api/`, Gin, gRPC agent tunnel with mTLS)
  - Next.js Pages Router + Mantine (`frontend/`)
  - Helm charts (`helm/`)
  - a Terraform runner image (`Dockerfile.terraform`)
  - runner scripts (`scripts/runner/`)
- **Gen3-specific code is moving to `plugins/gen3/`** (Phase 0). Don't add new Gen3 assumptions to core code.

## Rules
- **Cloud safety:**
  - **Never touch `csoc.planx-pla.net`.**
  - AWS end-to-end tests use the workshop account `354484406138` / zone `workshop.planx-pla.net`.
  - Tear test environments down with Destroy → **Force**, not graceful. Never edit runner scripts while a run is in progress.
- **Testing:** test the bootstrap or "Create environment" flow through the browser wizard, starting from a blank account. Don't create resources out of band.
- **Frontend:**
  - Import Monaco from `@/components/MonacoEditor`, never from `@monaco-editor/react` directly.
  - react-doctor config is `doctor.config.json` (key `ignore.files`, which excludes `.next`).
- **AI features can only propose changes:** read, open PRs, notify. They never apply to clusters or clouds (DECISIONS D4).
- **GitOps by default:** a change is a PR (DECISIONS D6).
