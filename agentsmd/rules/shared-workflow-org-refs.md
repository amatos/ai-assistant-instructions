---
name: shared-workflow-org-refs
description: In GitHub Actions uses: clauses, reference shared-CI repos by literal current owner — the don't-rewrite-on-sight redirect rule does NOT apply
paths:
  - ".github/workflows/*.yml"
  - ".github/workflows/*.yaml"
---

# Shared Workflow Org References

GitHub Actions reusable-workflow and action `uses:` clauses are the one place the
"don't rewrite `amatos/*` on sight — the redirect holds" rule does NOT apply.

Two hard GitHub constraints:

- `uses:` cannot contain expressions or variables — the owner is always a literal.
  `uses: ${{ vars.X }}/repo/...@ref` is rejected. No org variable can centralize it.
- `uses:` does NOT follow repository transfer/rename redirects (by design). API, git, and the browser DO follow them; Actions does not.

Consequence: when a shared-CI repo changes orgs, every consumer's `uses:` fails at parse time
(the run shows zero jobs and "workflow was not found"). There is no runtime variable that prevents this.

## Canonical home: amatos

**amatos is the canonical home for everything amatos uses.** Anything a amatos
repo consumes — reusable workflows, presets, policies — belongs under `amatos/*`.
`amatos-personal/*` may depend on `amatos/*`; `amatos/*` must **never** depend
on `amatos-personal/*`. When a shared workflow is used by amatos, its home is
`amatos/.github` (or the relevant `amatos/*` repo), not the personal account.

Because `uses:` does not follow redirects, reference each workflow by its literal
**current** owner from the table below until a pending relocation actually lands.

| Shared-CI workflow set | Current home | Status |
| --- | --- | --- |
| `ai-workflows` reusable workflows | `amatos/ai-workflows` | canonical |
| Nix reusable workflows (`_nix-validate.yml`, `_nix-build.yml`) | `amatos/.github` | canonical |
| Release-please (`_release-please.yml`) | `amatos/.github` | canonical — org-native (amatos release App, major-bump block, auto-merge) |
| `_markdown-lint`, `_file-size`, `_osv-scan`, `_ci-gate`, … | `amatos-personal/.github` | **pending relocation to `amatos/.github`** |

Nix and release-please were deliberately relocated to `amatos/.github` (the org owns
its own CI). The remaining non-Nix `.github` workflows are still in
`amatos-personal/.github` **only until they are moved the same way** — that is a
transitional home, not a permanent one. Repoint consumers via the sweep below as each
moves; do not move any back to the personal account.

## Rules

- In `uses:`, always reference the literal current owner above.
- Do NOT replace a reusable-workflow call with a `gh workflow run` / checkout `vars.*` dispatcher
  just to gain a variable: that loses required-check status, inputs/outputs, and `secrets: inherit`.
- gh-aw `{{#import ...}}` references and compiled `*.lock.yml` files resolve at compile time
  via the redirect — leave them to the don't-rewrite rule and gh-aw recompilation.

## If a shared-CI repo must move anyway (sweep)

1. `gh search code 'OLD_OWNER/REPO' --owner amatos` (and `--owner amatos-personal`), filtered to `.github/workflows/*.yml`.
2. Per consumer repo: swap only the `uses:` owner segment, preserving path and `@ref`; one PR per repo.
3. Skip `*.lock.yml`, `{{#import}}`, and docs.
4. Token tiers: amatos repos → amatos; amatos-personal repos → PRIVATE.
