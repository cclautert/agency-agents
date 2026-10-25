# Project profile — agency-agents

Read by the ai-engineering harness (context pack, `dev-task`) and by agents before any work here.
Last refreshed: 2026-10-24 on main@ad9264e3 by Claude (ai-engineering onboarding; commands taken from `.github/workflows/*.yml`).
`[V]` verified · `[I]` inferred, confirm before relying on it.

## 1. What it is
A collection of AI agent definitions (Markdown with YAML frontmatter: `name`, `description`, `color`), plus bash/python
scripts that lint them, check manifest consistency, convert them per tool (Claude Code, Cursor, Codex, Gemini CLI,
OpenCode, Hermes, ...) and install them. No application code, no build artefact. No CLAUDE.md, no `docs/`. [V]

| Part | Path | Notes |
|---|---|---|
| Agents | `academic/ design/ engineering/ finance/ game-development/ gis/ healthcare/ marketing/ paid-media/ product/ project-management/ research/ sales/ security/ spatial-computing/ specialized/ support/ testing/` | 18 divisions, 279 agent files on main@ad9264e3 [V] |
| Manifests | `divisions.json`, `tools.json`, `strategy/runbooks.json` | single sources of truth, cross-checked by CI scripts [V] |
| Scripts | `scripts/*.sh`, `scripts/*.py` | lint, checks, `convert.sh`, `install.sh`, tests [V] |
| Generated | `integrations/<tool>/...` | outputs of `convert.sh`; mostly git-ignored, only READMEs committed [V] |
| Playbooks | `strategy/`, `examples/` | prose, no agent frontmatter [V] |

## 2. Branches and remotes
- Fork: `origin` = GitHub `cclautert/agency-agents`, `upstream` = `msitarzewski/agency-agents` [V]
- **Integration branch (PRs target): `main`** [V] — default branch on both remotes; merges are `Merge pull request #N from ...` (upstream PRs).
- **origin has no commits of the user's own**: `upstream/main..origin/main` is empty; origin/main is a plain mirror of an
  older upstream commit (ad9264e, 2026-09-12) and is **421 commits behind `upstream/main`** (4e3ac3c, 2026-10-19). origin has only `main`. [V]
- Local checkout: `main` == `origin/main`, clean, on 2026-10-24. [V]
- Work items: GitHub Issues / PRs on upstream (`#NNN`) [V]
- CI: GitHub Actions on `pull_request` and `push` to `main` (ubuntu; installer also on macOS) — never trigger workflows unasked. [V]

## 3. Commands (Windows `cmd.exe`, from the repo root — this is how the harness runs them)
The scripts are bash and call `python3`. On this machine `python3` is the Microsoft Store stub (it fails), so every
command defines `python3` as an exported bash function pointing at `C:\Python314\python` (3.14, has PyYAML/tomllib).
Git Bash by full path: under `cmd`, `bash` can resolve to WSL.

| Check | Command | Notes |
|---|---|---|
| build | `"C:\Program Files\Git\bin\bash.exe" -c "python3(){ /c/Python314/python \"$@\"; }; export -f python3; set -e; for f in scripts/*.sh; do bash -n $f; done; ./scripts/lint-agents.sh; ./scripts/check-divisions.sh; ./scripts/check-tools.sh; ./scripts/check-runbooks.sh"` | Nothing to compile: the repo's own validation is the "build". CI lints only changed files; this lints all 279. ~13 min on Windows. [V] |
| unit | `"C:\Program Files\Git\bin\bash.exe" -c "python3(){ /c/Python314/python \"$@\"; }; export -f python3; set -e; ./scripts/test-install.sh; ./scripts/test-agent-selection.sh; ./scripts/test-convert-frontmatter.sh; ./scripts/check-hermes-config-rewrite.sh; python3 scripts/check-hermes-plugin.py"` | `test-install.sh` alone took ~46 min on Windows Git Bash (CI: Linux/macOS) and fails one Windows-only case (see 4). The later scripts did not run. [V] |
| convert-outputs | `... ./scripts/test-convert-outputs.sh` (same prefix) | CI step in `check-tools.yml` (drift advisory on PRs). Not measured: stopped after the time-box. [I] |
| originality | `./scripts/check-agent-originality.sh <changed agent files>` | CI runs it only on the agent files a PR changes. [V from CI] |

Tests sandbox `HOME` for the installer [V]; they also pass `CLAUDE_CONFIG_DIR`/`COPILOT_AGENT_DIR` explicitly per case. [V]
Never run: `scripts/install.sh` without a sandboxed `HOME`/`--path` (it writes into the real `~/.claude/agents` and other
tool config dirs — the user's live agents may come from here), `scripts/convert.sh` in the user's checkout (writes
`integrations/`), `scripts/i18n/localize-agents-zh.ps1`. No deploy/publish/release scripts exist in this repo. [V]

## 4. Test baseline
`main@ad9264e3, 2026-10-24`, measured by the harness in a `C:\wt` worktree (RUN-20261025T001617-d9f54d, state Failed):
build PASS (0 errors, 58 warnings for missing recommended sections, 279 files; 18 divisions, 16 tools, 4 runbooks
consistent) [V] · `agencyagents.Tests` **35/37 passed** (`scripts/test-install.sh`; 1 failed + 1 known-broken xfail) [V].
Failing set: `--link creates symlinks, not copies` (expected 64, got 0) — environment: Git Bash on Windows does not
create real symlinks without Developer Mode / `MSYS=winsymlinks:nativestrict`; passes on CI's Linux/macOS [I].
xfail (declared by the script): `--parallel installs exactly the one selected agent` — "fixed by PR #755" upstream [V].
Not measured: `test-agent-selection.sh`, `test-convert-frontmatter.sh`, `check-hermes-config-rewrite.sh`,
`check-hermes-plugin.py` (did not run: `set -e` stopped after `test-install.sh`), `test-convert-outputs.sh` (stopped
after the time-box). ⚠️ The unit check overran its 1800 s timeout on a loaded machine; expect ~45+ min per run on Windows.

## 5. Skills
No project skills: the repo has no `.claude/skills/` (no `.claude/` at all before this profile). Nothing bound. [V]

## 6. Risk
The ai-engineering harness sets each change's tier from the files it touches (`harness/lib/risk.js`: these rows
on top of its defaults — CI/deploy workflows, Dockerfiles, compose files, `appsettings*`, `.env*` are high; docs are
low; anything else is medium). The tier raises the review depth (low quick · medium standard · high deep) and sets
the human approvals `harness merge check` asks for, bound to the exact commit. Solo repo: high asks for two people
and gets one — the harness reports the cap.
Human approvers: Cristiano

| Tier | Path | Why |
|---|---|---|
| high | `scripts/install.sh` | writes into users' tool config dirs (`~/.claude/agents`, Hermes config rewrite) |
| high | `scripts/lib.sh` | shared by install, convert and lint |
| high | `scripts/convert.sh` | generates every tool's output; drift checked against `scripts/convert-outputs.sha256` |
| high | `scripts/build-hermes-plugin.py` | generated plugin code installed into Hermes |
| high | `.github/**` | CI gates and their path filters |
| high | `divisions.json` | source of truth consumed by the Agency Agents app and the scripts |
| high | `tools.json` | source of truth for supported tools |
| high | `strategy/runbooks.json` | the app's one-click team deploy depends on its slugs |
| low | `examples/**` | prose examples |
| low | `strategy/**/*.md` | playbooks, no frontmatter |
| low | `integrations/*/README.md` | docs |
| low | `README.md` | top-level docs |
| low | `CONTRIBUTING*.md` | contributor docs |

## Contradictions found
- `README.md`/`CONTRIBUTING.md` present the scripts as runnable as-is; on this Windows machine they need `python3` remapped
  (Store stub) and `--link` cannot work in Git Bash. [V]
- The fork's `origin/main` is 421 commits behind `upstream/main`; anything read on origin (counts, agents, scripts) is the
  2026-09-12 state, not the current upstream. [V]
