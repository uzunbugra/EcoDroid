# Contributing to EcoDroid

Thanks for your interest! Read this once before your first pull request.

## 1. Branching

| Branch | Purpose | Protection |
|---|---|---|
| `main` | Released code. Updated only by a release PR from `develop`. | PR + code-owner approval, CI green, no force-push, maintainer-only merge |
| `develop` | Integration branch for the next release (default branch). | PR required, CI green, no force-push |
| `feature/<short-desc>` | New work, e.g. `feature/eco-adaptive-fps` | Branch off `develop`, open the PR against `develop` |
| `fix/<short-desc>` | Bug fixes | Same as feature branches |
| `hotfix/<short-desc>` | Urgent fix for a release | Branch off `main`, merge back into both `main` and `develop` |

External contributors: fork the repository and open the PR from your fork
against `develop`.

## 2. Commits

Use [Conventional Commits](https://www.conventionalcommits.org/); the
changelog is built from them.

```text
<type>(<scope>): <short imperative summary>

feat(eco): lower FPS step by step when the window is hidden
fix(vmm): keep the vCPU budget when the guest resumes
docs(readme): add build instructions
ci(rust): run cargo deny on pull requests
```

Types: `feat fix docs test chore refactor perf ci build`.

## 3. Pull requests

- Fill in the PR template and link issues with `Closes #<n>`.
- Keep PRs small (under about 400 changed lines). Keep refactors separate
  from behaviour changes.
- CI must be green before merge.
- `develop` takes squash merges. `develop` → `main` happens only through a
  release PR that adds a `CHANGELOG.md` entry.
- Only the maintainer (`@uzunbugra`) can approve and merge into `main`.

## 4. Local checks

Run these before pushing (they are the same checks CI runs):

```bash
# Rust engine (from engine/hvf)
cargo clippy --all-targets
AVO_VMM_MUTE=1 cargo test
cargo deny --manifest-path engine/hvf/Cargo.toml check   # from the repo root

# EcoDroid's own crates (engine/eco) must also be rustfmt-clean.
# Upstream Avo code is left unformatted so upstream merges stay clean.

# Markdown
npx markdownlint-cli2 "*.md" ".github/**/*.md" "docs/security-audit.md"
```

## 5. Ground rules for game compatibility

EcoDroid does not hide itself from games. Pull requests that add emulator
detection evasion, anti-cheat or attestation bypasses, game file modification,
bots or macros will not be accepted.

## 6. Security

Never commit secrets. Report vulnerabilities privately as described in
[SECURITY.md](SECURITY.md), not in public issues.

## 7. License

By contributing you agree that your contribution is licensed under the
[Apache License 2.0](LICENSE).
