# Install

Get Quality Checks running on a project in three steps.

## 1. Copy the bundle

```bash
# From the root of the project you want to gate
cp -r path/to/quality-checks ./
cp path/to/.claude/commands/quality-checks.md .claude/commands/
mkdir -p .github/workflows
cp path/to/.github/workflows/quality-checks.yml .github/workflows/
```

The `quality-checks/` directory is self-contained: references, scripts, tests, fixtures. The `.claude/commands/quality-checks.md` is the user-invokable slash command. The workflow is the CI gate.

## 2. Make the scripts executable and run a self-test

```bash
chmod +x quality-checks/scripts/check.sh quality-checks/tests/run-tests.sh
quality-checks/tests/run-tests.sh
```

Expected: `13 passed, 0 failed`. If anything fails, the install is broken — check that `ripgrep` is installed (`brew install ripgrep` on macOS, `apt-get install ripgrep` on Debian / Ubuntu).

## 3. Run the deterministic gate on the project

```bash
quality-checks/scripts/check.sh
```

Scans the source directories for the eleven regex-detectable rules (banned diction, em dashes, side-stripe borders, gradient text, layout-property animation, `outline: none`, `<div onClick>`, `dangerouslySetInnerHTML`, `v-html`, Svelte `{@html}`, pure black / white). Returns exit 0 on pass, 1 on any P0 or P1 finding.

The CI workflow runs the same script on every pull request and posts a comment summarising findings.

## Optional: invoke the full pipeline

The deterministic gate is the floor. The full nine-phase pipeline runs through Claude Code's slash command and adds the subjective phases:

```
/quality-checks
```

Loads the references under `quality-checks/`, runs each phase against the current branch diff, emits the verdict per `RUBRIC.md`.

## Optional: add a PRODUCT.md

The pipeline degrades gracefully if `PRODUCT.md` is missing. It runs every phase except the register-specific and industry-specific checks, and flags the omission in the report.

For the full benefit, add a root-level `PRODUCT.md` with:

- **Purpose** (one paragraph)
- **Users** (specific roles, not "everyone")
- **Register** (`brand` or `product`)
- **Anti-references** (the aesthetic families this product specifically rejects)

The `impeccable` skill's `/impeccable teach` command generates one through guided questions.

## Wiring into an existing pipeline

The script is exit-code-driven. Wire it anywhere:

```bash
# Pre-commit hook
quality-checks/scripts/check.sh || exit 1
```

```yaml
# Non-GitHub CI runner
- name: Quality Checks
  run: quality-checks/scripts/check.sh
```

```makefile
# Makefile
check:
	quality-checks/scripts/check.sh
```

## Configuration

Environment variables tune the runner:

| Var | Default | Effect |
|-----|---------|--------|
| `QC_ROOT` | `$(pwd)` | Root directory to scan |
| `QC_TARGETS` | auto-detected | Space-separated paths to scan |
| `QC_JSON` | (none) | Write findings JSON to this path |
| `QC_STRICT` | `0` | When `1`, P2 findings also fail the gate |
| `QC_QUIET` | `0` | When `1`, suppress per-finding console output |

## Uninstall

```bash
rm -rf quality-checks .github/workflows/quality-checks.yml .claude/commands/quality-checks.md
```

No persistent state. No external services. No global config.
