# Easy Execute Skill

Reusable Codex skill for taking a goal, plan, spec, or review/repair effort through a thread-long execution workflow: planning, foundation decisions, delegation, implementation, adversarial review, repair, verification, and closeout.

## Install

Install with Codex's skill installer:

```bash
python3 "${CODEX_HOME:-$HOME/.codex}/skills/.system/skill-installer/scripts/install-skill-from-github.py" \
  --repo OWNER/easy-execute-skill \
  --path skills/easy-execute
```

Restart Codex after installing so the skill is loaded into new sessions.

## Update From Local Skill

When the installed local skill is edited, refresh this repo copy:

```bash
cp "${CODEX_HOME:-$HOME/.codex}/skills/easy-execute/SKILL.md" skills/easy-execute/SKILL.md
cp "${CODEX_HOME:-$HOME/.codex}/skills/easy-execute/agents/openai.yaml" skills/easy-execute/agents/openai.yaml
git diff
```

Then commit and push the change.

Before pushing, audit the diff for private names, company details, machine-specific paths, credentials, URLs, customer data, and environment-specific operational rules. Keep the public skill generic unless a private fork is intended.

## Skill Path

The skill lives at:

```text
skills/easy-execute/
```
