# Easy Execute Skill

Reusable Codex skill for taking a goal, plan, spec, or review/repair effort through a thread-long execution workflow: planning, foundation decisions, delegation, implementation, adversarial review, repair, verification, and closeout.

## Install

Install with Codex's skill installer:

```bash
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo timothyreavis/easy-execute-skill \
  --path skills/easy-execute
```

Restart Codex after installing so the skill is loaded into new sessions.

## Update From Local Skill

When the installed local skill is edited, refresh this repo copy:

```bash
cp ~/.codex/skills/easy-execute/SKILL.md skills/easy-execute/SKILL.md
cp ~/.codex/skills/easy-execute/agents/openai.yaml skills/easy-execute/agents/openai.yaml
git diff
```

Then commit and push the change.

## Skill Path

The skill lives at:

```text
skills/easy-execute/
```
