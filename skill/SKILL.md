---
name: impeccable
description: >-
  Use for goal-first requests — "impeccable this", "make this faster",
  "verify this change", "harden this", "audit this", "fix this properly",
  "review this for real" — when the user gives a goal, not a skill name.
  Detects the task's language from file, manifest, and prompt evidence,
  installs the matching impeccable member skill when it is missing, and
  hands the work to it whole.
---

# Impeccable

A router, not a rulebook. This skill holds no language rules: it resolves
the task's language, makes sure the member skill that owns that language
is installed, and tells you to follow that skill end to end. All
verification and optimization knowledge lives in the member skills so
they evolve independently. The member list is `registry.yaml`, next to
this file.

## 1. Resolve the language

Gather evidence in this order; stop at the first tier that resolves:

1. **Changed files.** The files the task touches (`git status --porcelain`
   or `git diff --name-only HEAD` — staged, unstaged, and untracked —
   the diff under review, the paths the user points at). Match extensions
   against `signals.extensions` in `registry.yaml`. An extension that
   names a language the registry does not cover (`.go`, `.ts`, `.java`,
   …) is still language evidence: it resolves to "not covered" — do not
   fall through to manifests to find a covered language. Only a change
   with no language signal at all (docs, configs, data, an empty diff)
   continues to tier 2.
2. **Manifests.** Match repo-root files against `signals.manifests`. In a
   monorepo, check for a manifest beside the changed paths — a component's
   own manifest beats the repo root's.
3. **The user's words.** Whole-word matches against `signals.words`.

Rules for hard cases:

- Mixed-language work routes per component: run each member skill on the
  component it owns and report per component.
- If repo evidence and the user's words disagree ("harden this" inside a
  Go service), the repo wins: name the languages `registry.yaml` covers
  and stop. Never route to a member the registry has no row for, and
  never pretend a skill exists.
- If nothing resolves, say which languages are covered and ask the user
  to point at the files.

## 2. Make sure the member skill is installed

The member skill lives as a sibling of this one: if `impeccable` sits at
`<skills-dir>/impeccable`, the member must sit at `<skills-dir>/<skill>`
with `<skill>` from the registry row (e.g. `<skills-dir>/impeccable-rust`).

- If the member skill is already loaded in your session — through a
  plugin manager or another skills dir — skip the install and go to
  step 3.
- Otherwise check for `<skills-dir>/<skill>/SKILL.md`. If present, go to
  step 3.
- If missing, install it from the registry `repo`:

  ```sh
  git clone --depth 1 <repo> /tmp/<skill>
  cp -r /tmp/<skill>/skill <skills-dir>/<skill>
  ```

  Follow the member repo's own README install instructions when they
  differ. If `<skills-dir>/<skill>` exists without a `SKILL.md` (a
  partial or stale install), move it aside first —
  `mv <skills-dir>/<skill> <skills-dir>/<skill>.bak` — so `cp -r` does
  not nest `skill/` inside it. Verify `<skills-dir>/<skill>/SKILL.md`
  exists before handing off.
- If you cannot install (no network, no git, a read-only skills dir),
  give the user this one line, filling in the registry row's values, and
  stop:

  ```sh
  git clone <repo> && cp -r <repo-dir>/skill <skills-dir>/<skill>
  ```

## 3. Hand off

Invoke the member skill's `SKILL.md` and follow it end to end, verbatim —
its checklist, its tooling, its report format. This skill adds nothing
and removes nothing. If you cannot activate another skill by name, read
`<skills-dir>/<skill>/SKILL.md` and execute its protocol directly.

In your report, name the member skill that ran, the evidence that routed
the work to it, and — in the member skill's own format — what it checked.

## Adding a language

One row in `registry.yaml`: `language`, `skill`, `repo`, and the signals
that detect it. Nothing else in this skill changes.
