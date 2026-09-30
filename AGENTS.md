# AGENTS.md

## What this is

The default template for `leinardi/*` repositories: the shared scaffolding (Makefile, pre-commit hooks, CI, release, GitHub
settings files, docs, skills) with no language or stack yet. Repositories created from it copy these files once; a change here
reaches only the repositories created afterwards, so a fix that existing repositories need is ported to them separately.

## Common commands

```bash
make check               # pre-commit on all files (checkmake, actionlint, markdownlint, shellcheck, prettier, yamllint, …)
make check-stage         # pre-commit on the staged files only
make pre-commit-install  # installs the pre-commit and commit-msg hooks
make help                # lists every target
```

Before calling a change done, run `make check`.

The Makefile pulls shared snippets from `leinardi/make-common@v1` into `.mk/` on first run. To refresh: `make mk-common-update`.
Project targets live in the local `.mk/*.mk` files listed in `MK_LOCAL_FILES`, never as recipes in the Makefile.

## Layout

| Path | What lives there |
| --- | --- |
| `Makefile`, `.mk/`, `scripts/bootstrap-mk-common.sh` | make-common: shared snippets in `MK_COMMON_FILES`, local ones in `MK_LOCAL_FILES` |
| `.github/workflows/ci.yaml` | the reviewdog jobs and `conventional-commits`, on pull requests |
| `.github/workflows/release.yaml` | the **Release** workflow, on gh-reusable-workflows' `simple-tag-and-release` |
| `.github/workflows/pre-commit-warmup.yaml` | refreshes the pre-commit cache on `main` when the hook config changes |
| `.agents/skills/` | project skills; `.claude/skills` is a symlink to it |

## Conventions

- Pin every action and reusable workflow by full commit SHA, with the version in a trailing comment (`@<sha> # vX.Y.Z`).
- Give every workflow a top-level `permissions: contents: read`, and widen it only on the job that needs more.
- Pass `${{ }}` expressions into `run:` scripts through `env:`, never inline them in the script.
- Name workflow files `.yaml`, except a file whose name a trusted publisher (PyPI, npm) is bound to.
- `README.md` is the user-facing contract: keep it in step with what the repository does.

## Commit messages

All commits MUST be Conventional Commits 1.0.0 **with a scope**: `<type>(<scope>)[!]: <description>`, optional blank-line body
and footers. Enforced by the `conventional-pre-commit` `commit-msg` hook (`--force-scope`, installed by
`make pre-commit-install`) and by the `conventional-commits` CI job. Breaking changes use `!` before `:` or a `BREAKING CHANGE:`
footer. Examples: `fix(ci): pin actionlint by SHA`, `feat(release): attach the build to the release`.

A release with no explicit version is derived by `svu` from the commits since the last `vX.Y.Z` tag, so a wrong type ships a wrong
version:

| Release | Commit | Example |
| --- | --- | --- |
| major | any type with `!` before the colon, or a `BREAKING CHANGE:` footer | `feat(api)!: drop the v1 endpoints` |
| minor | `feat` | `feat(release): attach the build to the release` |
| patch | `fix` | `fix(ci): pin actionlint by SHA` |
| none | everything else: `perf`, `refactor`, `build`, `ci`, `chore`, `docs`, `style`, `test`, `revert` | `docs(readme): fix a link` |

The highest bump among the commits wins; with only "none" commits since the last tag, a release with no version fails with
"nothing to bump". **Pick the type by whether the change should ship, not by what kind of change it is.** Every commit of a pull
request lands on `main` (merge commits only), so every commit counts, not just the pull request title. See
[`CONTRIBUTING.md`](CONTRIBUTING.md#releasing).

## Project skills

Skills live in `.agents/skills/` (symlinked as `.claude/skills`). Load `adversarial-review` for any review request ("review my
diff", "is this ready to merge").
