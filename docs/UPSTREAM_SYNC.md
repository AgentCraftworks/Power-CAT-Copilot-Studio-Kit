# Upstream Sync Guide

> **Upstream:** [microsoft/Powercat-Copilotstudio-Accelerator](https://github.com/microsoft/Powercat-Copilotstudio-Accelerator) (main branch)  
> **Schedule:** Monthly, 1st of each month at 09:00 UTC  
> **Workflow:** `.github/workflows/upstream-sync.yml`  
> **Tracking file:** `.upstream/base-commit.txt`

## Overview

`AgentCraftworks/Power-CAT-Copilot-Studio-Kit` is a fork of the Microsoft Power CAT Copilot Studio Accelerator. We track upstream changes monthly and incorporate them via pull requests so AgentCraftworks developers have access to the latest Kit features (batch testing, Compliance Hub, Agent Inventory, Rubrics Refinement, etc.).

## How the Sync Works

1. **Monthly schedule** — On the 1st of each month, the workflow fetches the upstream `main` branch
2. **Change detection** — Compares `.upstream/base-commit.txt` (last synced SHA) with upstream HEAD
3. **Merge** — If there are new commits, creates a `upstream-sync/YYYY-MM` branch with a merge commit
4. **PR** — Opens a PR against `main` labeled `upstream-sync,automated`
5. **Base commit update** — On PR merge, `.upstream/base-commit.txt` is updated to the new upstream HEAD SHA

## Owned Paths (Do Not Blindly Accept Upstream)

These files/directories may have intentional AgentCraftworks divergence. **Review carefully** before merging the sync PR:

| Path | Why We Own It |
|------|---------------|
| `.github/` | Our workflows (upstream-sync.yml), issue templates, CODEOWNERS |
| `.upstream/` | Our sync tracking infrastructure |
| `AGENT_INVENTORY.md` | AgentCraftworks-specific agent registry entries |
| `AGENTREVIEWTOOL_REFERENCE_GUIDE.md` | AgentCraftworks-specific review guidance |
| `docs/UPSTREAM_SYNC.md` | This file |

For all **other paths**, accept upstream changes unless they conflict with known customizations.

## Reviewing a Sync PR

When the automated sync PR opens:

1. **Check the conflict count** — If there are many conflicts, the PR will flag them
2. **Review owned paths** — Look at diffs for the paths listed above
3. **Check the changelog** — Review the upstream repo for release notes since the last sync
4. **Test new features** — If major features were added, deploy to a dev environment first
5. **Merge** — Use "Merge commit" (not squash/rebase) to preserve the upstream history

## Manual Sync

To trigger a sync outside the schedule:

```bash
# Via GitHub UI
Actions → Upstream Sync → Run workflow

# Or locally:
git remote add upstream https://github.com/microsoft/Powercat-Copilotstudio-Accelerator.git
git fetch upstream main
git merge upstream/main --no-edit
# Resolve any conflicts in owned paths
git push origin main
```

## Version Tracking

The current upstream base commit is stored in `.upstream/base-commit.txt`. This allows the workflow to:
- Only sync new commits (not re-apply already-synced changes)
- Report the exact upstream SHA in the PR description

## Upstream Release Cadence

The Microsoft Power CAT team releases updates to `microsoft/Powercat-Copilotstudio-Accelerator` approximately:
- Feature releases: Monthly or with major Power Platform releases
- Bug fixes: As needed

Check the upstream [releases page](https://github.com/microsoft/Powercat-Copilotstudio-Accelerator/releases) and [changelog](https://github.com/microsoft/Powercat-Copilotstudio-Accelerator/blob/main/CHANGELOG.md) when reviewing sync PRs.

## AgentCraftworks-Specific Features

The following features in this repo are **AgentCraftworks additions** (not in upstream):
- `.upstream/` directory and tracking infrastructure
- `.github/workflows/upstream-sync.yml` — this sync workflow
- Integration with AgentCraftworks eval gate (test results feed into `copilot_studio_get_eval_results` MCP tool)

## Troubleshooting

### Merge conflicts on sync PR

If the workflow cannot auto-merge:
1. Pull the `upstream-sync/YYYY-MM` branch locally
2. Resolve conflicts (prefer upstream for non-owned paths, prefer ours for owned paths)
3. Push the resolved branch
4. The PR will update automatically

### Long path errors (Windows)

The upstream repo contains paths that exceed the Windows 260-character limit. The workflow runs on `ubuntu-latest` and sets `core.longpaths true` to avoid this. If cloning locally on Windows:

```bash
git config --global core.longpaths true
```

### Workflow failed — upstream fetch error

If the upstream repo URL changes (e.g., the Microsoft repo is renamed):
1. Update the `upstream` remote URL in `.github/workflows/upstream-sync.yml`
2. Update the URL in `.upstream/config.json`
3. Trigger the workflow manually
