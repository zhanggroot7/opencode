# self-improving-agent-3.0.18

This `.learnings/` tree is **created/updated** by the global Cursor hook `afterAgentResponse` → `~/.cursor/hooks/self-improvement-after-response.sh`.

| | |
|---|---|
| **Skill (Agent Skills)** | `self-improving-agent-3.0.18` |
| **SKILL.md** | `/Users/groot/.cursor/skills/self-improving-agent-3.0.18/SKILL.md` |
| **What the hook does** | First-use init (same as “First-Use Initialisation” in the skill), plus a line in `.after-agent-response-hook` per assistant reply. |
| **What the hook does *not* do** | It does not run the model or “execute” the skill text — the agent should follow `SKILL.md` for logging format and rules. |

See **/Users/groot/.cursor/skills/self-improving-agent-3.0.18/SKILL.md** for formats (`LEARNINGS.md`, `ERRORS.md`, `FEATURE_REQUESTS.md`), promotion, and triggers.

## Persistence

This hook **keeps** your logs: it does not remove `.learnings/` or rewrite existing files above. Only missing seed files are created; `.after-agent-response-hook` is append-only (run log).

---

