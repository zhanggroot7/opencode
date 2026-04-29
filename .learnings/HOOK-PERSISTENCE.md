# self-improving-agent-3.0.18 — what the Cursor hook does to this folder

The global hook `afterAgentResponse` → `~/.cursor/hooks/self-improvement-after-response.sh` is **non-destructive**:

- **Keeps** `.learnings/` and your entries for the long term (until you change or delete them).
- **Does not** delete this directory or the skill log files; **does not** overwrite a file that already exists (seed files and `SKILL-REFERENCE.md` are create-if-missing only).
- **Does** `mkdir -p .learnings`, then create any **missing** files from the skill’s first-use list: `LEARNINGS.md`, `ERRORS.md`, `FEATURE_REQUESTS.md`.
- **Does** append one line to `.after-agent-response-hook` per run (optional audit trail; you may delete that file if you do not want it).

For git: see “Gitignore Options” in `/Users/groot/.cursor/skills/self-improving-agent-3.0.18/SKILL.md` if you want `.learnings/` local-only or shared.

---

