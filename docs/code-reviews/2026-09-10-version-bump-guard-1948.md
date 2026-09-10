# Code review: manifest.json version-bump CI guard (ut-docs#1948)

**Date:** 2026-09-10
**Author:** Farshid Mirza (pipeline, Sonnet dev, `complexity:medium`)
**Independent reviewer:** Opus, fresh-context subagent (`isolation: "worktree"`)
**PR:** universaltill/ut-plugin-theme-screen-top#4 (branch: `fix/1948-version-bump-guard-theme-screen-top`)

## What shipped

Third deployment of the ut-docs#1940 guard: `ut-plugin-language-de`
(original) → `ut-plugin-theme-midnight` (this cycle, earlier today) →
`ut-plugin-theme-screen-top` (this PR). Byte-identical shape to
`theme-midnight` (`runtime: "none"`, no `src/`,
`entries=(manifest.json assets README.md)`), so `check-version-bump.sh`
carries over verbatim; only the test fixtures (manifest id/name/theme key)
needed repo-specific adaptation. Ships with the malformed-shellcheck-
directive fix (found in `theme-midnight`'s review earlier today) already
folded in from the first commit, plus the `CLAUDE.md` exemption-list
wording fix (`scripts/` named alongside `docs/`/`.github/`).

## Independent review findings

Verdict: **SAFE TO MERGE**, no blockers. The reviewer deliberately did not
treat "matches the already-reviewed pattern" as license to skim — it ran
the full verification suite itself rather than diffing text and assuming:

- 11/11 self-tests reproduced locally; `validate.sh`/`package.sh` clean.
- `shellcheck 0.11.0`: exit 0 on both new scripts, verified three ways —
  a clean run, then **deliberately removing the directive** (confirmed
  SC2053 actually fires, so the suppression isn't dead weight), then
  **deliberately re-injecting the old malformed directive** (confirmed it
  reproduces SC1072/SC1073 exactly as the theme-midnight review found) —
  so the fix is proven load-bearing, not just present.
- `check-version-bump.sh` vs `theme-midnight`'s: **byte-identical**
  (confirmed by `diff` and matching SHA-256).
- `check-version-bump.test.sh` vs `theme-midnight`'s: differs in exactly
  5 lines, all fixture identifiers (id, name, theme key, two README
  placeholder strings) — no logic drift, no missing cases, no
  reintroduced bugs. Grepped all four changed files for leftover foreign
  identifiers (`midnight`, `language-de`, `locales/`, etc.): zero hits.
- `version-bump` CI job block: identical to `theme-midnight`'s, and the
  PR's own live CI run has `version-bump` green.
- Independent scratch-commit simulations (own clone, not trusting the PR
  body): asset edit without bump → FAIL naming the file; bump too → PASS;
  bump to an existing real tag (`v1.0.2`) → FAIL, ALREADY TAGGED; a
  **new nested asset path** (`assets/sub/x.css`) → FAIL, confirming the
  `assets/*` glob correctly crosses into subdirectories (separately
  confirmed it does *not* false-positive-match `assetsfoo.css`); the
  entries-mirror self-test mutation-tested by injecting a real
  `package.sh` drift (`+CHANGELOG.md`) → correctly caught. All scratch
  commits reverted, working tree confirmed clean afterward.
- Fixture self-consistency: extracted the fixture manifest from the test
  file and ran it through this repo's **real** `validate.sh`/`package.sh`
  — matches (id, name, theme key, permissions, locales, runtime,
  device_arch, config.css all consistent with the real `manifest.json`).
- Secrets/PII scan: clean (only the RFC 2606 `test@example.com` fixture
  email, same as both prior deployments).

**Process finding (addressed by this record's own existence):** the
reviewer flagged that this review record wasn't yet on the branch at
review time — same timing as `theme-midnight`, where the record also
lands as part of this step rather than being present when the reviewer
first read the diff. Added now, before merge.

**Nits, all inherited byte-for-byte from the reference (not introduced by
this adaptation), not fixed here:**
- `ci.yml`'s self-test step `if: always()` is a no-op-at-best guard
  (would still attempt to run past a failed checkout). Identical in all
  three repos.
- The already-tagged branch's suggested-next-version can itself already
  be tagged (cosmetic — the author still has to pick a real version).
- The entries-mirror pattern extraction scans the whole script's
  single-quoted literals, not just the array (F3 from `theme-midnight`'s
  review, still un-fixed everywhere).

**New information from this review, useful for the card:** the reviewer
confirmed **`ut-plugin-language-de`'s own copy of the guard still has the
un-fixed shellcheck directive today** (ran shellcheck against it directly:
still errors SC1072/SC1073) — so `-de` is now the one out of step with
`-midnight` and `-screen-top`, not the other way around. Reinforces the
ut-docs#1948 follow-up note to port F1 back to `-de` (and `-es`, not yet
checked directly but presumed to share it, ported from the same source).

## What was verified beyond automated tests

Everything in "Independent review findings" above was run for real by the
reviewer in an isolated worktree, not inferred from a text diff — see that
section for the specific commands, mutation tests, and scratch-commit
simulations.

## Safe-to-merge verdict

**Yes.** No blockers. Nits are pre-existing and shared across all three
repos with this guard — tracked as an ut-docs#1948 follow-up rather than
fixed piecemeal per repo.
