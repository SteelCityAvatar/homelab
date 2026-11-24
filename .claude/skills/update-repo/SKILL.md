---
name: update-repo
description: Bring the whole homelab repo up to date after work on the server or a decision — session log, HANDOFF, ROADMAP, plans, infra records, ADRs, README, CLAUDE.md, and the local notes plus their phone copy — then review for privacy and commit. Use when asked to "update the repo", "log this", "record this", or at the end of a working session.
---

# Update the repo

Goal: every file that states a fact which changed is updated, nothing stale remains,
and nothing private reaches the public repo. Work through all steps; skip a file only
after checking it.

## 1. Collect what changed

From the conversation and the current `docs/sessions/<date>.md`, list:
- **Facts** (versions, addresses, IDs, states: done / not done / unconfirmed)
- **Decisions** (and whether each is significant enough for an ADR)
- **Commands run** (exact, read-only vs destructive, outcome, including failed or
  blocked ones)
- **Explanations** worth keeping (→ notes)
- **Open questions** — mark ⚠️; never fill them in with assumptions

## 2. Survey for stale statements

```bash
git status -sb
git ls-files
```
Then `grep -rn -i` the tracked Markdown (excluding `docs/sessions/`, which is
history) for each topic touched: old states ("next", "not set up", "pending"), the
component names, addresses, versions. Every hit is a candidate edit.

## 3. Update, file by file

| File | Update when |
|---|---|
| `docs/sessions/<date>.md` | Always. Summary/findings at top, then "Command log". Span dates in the title if work crossed midnight. Earlier logs get only a one-line "Resolved <date>: see …" pointer, never rewrites |
| `HANDOFF.md` | Always. "Last updated", server-at-a-glance, current work checklist (`← next` on exactly one item), open-items table. Fold closed items into durable files |
| `ROADMAP.md` | A checklist item moved, or a new cross-phase item appeared |
| `docs/plans/*.md` | The plan's own checklist items changed (they hold the most detail) |
| `infra/*/README.md` | Install records, guest table, driver/device facts, post-install tasks |
| `docs/decisions/NNNN-*.md` | A significant, settled choice. Next number, dated, Status/Context/Decision/Rationale/Consequences |
| `CLAUDE.md` | Current-status paragraph; one line per new ADR under "Decisions already made"; new conventions |
| `README.md` | Phase status table / roadmap row moved, notable incident resolved, or a new design principle (link the ADR) |
| `docs/architecture.md` | Topology, networking, or service layout changed |
| `docs/spec.md`, `bom.md` | Hardware facts or purchases changed |

## 4. Notes (local only, gitignored)

- Add or extend a note in `docs/notes/` for anything explained during the work.
- Every note starts with `## Acronyms`; add new acronyms there **and** to the full
  list in `docs/notes/README.md` (alphabetical).
- New note → a row in the README table and a slot in `GROUPS` in
  `build_phone_page.py` (otherwise it lands under "Other").
- Rebuild the phone copy: `python -I docs/notes/build_phone_page.py docs/notes <scratchpad>/notes-phone.html`,
  check the output has no `>Other<` group, then republish to the artifact URL given in
  `docs/notes/README.md` (read the artifact first if this session hasn't).
- The script writes a complete HTML document. A publish tool that wraps pages in its
  own skeleton (the `Artifact` tool's publish) needs the output built with
  `--fragment`; one that serves the file as-is (`publish_files`) doesn't. Afterwards,
  read the artifact's first lines: exactly one doctype, at the very top.

## 5. Privacy pass (the repo is public)

Review `git diff` in full before staging. Remove or redact:
- Public IPv6 addresses/prefixes, full public keys, tokens, passwords, MAC addresses
- Personal or household details (people, retailers, receipts, travel plans)
- References to AI chats or sessions; "the owner"
Voice: impersonal engineer's build log. Operator slips only as stated lessons.

## 6. Verify and commit

- Re-run the stale-statement greps from step 2; resolve or ⚠️-flag every remaining hit.
- Commit with the GitHub no-reply address:
  `git -c user.email=103851266+SteelCityAvatar@users.noreply.github.com commit`
  One commit per coherent change; message says what changed, not how.
- **Push only when asked.** Report the ahead-count instead.
