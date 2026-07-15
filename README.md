# Easy Execute Skill

Reusable Codex skill for taking a goal, plan, spec, or review/repair effort through a thread-long execution workflow: planning, foundation decisions, delegation, implementation, adversarial review, repair, verification, and closeout.

## Install

Install with Codex's skill installer:

```bash
python3 "${CODEX_HOME:-$HOME/.codex}/skills/.system/skill-installer/scripts/install-skill-from-github.py" \
  --repo timothyreavis/easy-execute-skill \
  --path skills/easy-execute
```

Restart Codex after installing so the skill is loaded into new sessions.

## Develop And Update

Treat this repository as the canonical public source. Make reusable changes here first, review them, then reinstall or copy the reviewed files into the local skill directory. Do not copy an installed skill back into this repository blindly: another bundle, project overlay, or private fork may have replaced it.

```bash
git diff
cp skills/easy-execute/SKILL.md "${CODEX_HOME:-$HOME/.codex}/skills/easy-execute/SKILL.md"
cp skills/easy-execute/agents/openai.yaml "${CODEX_HOME:-$HOME/.codex}/skills/easy-execute/agents/openai.yaml"
```

Commit and push the public change before treating the installed copy as current.

Before pushing, audit the diff for private names, company details, machine-specific paths, credentials, URLs, customer data, and environment-specific operational rules. Keep the public skill generic unless a private fork is intended.

## Skill Path

The skill lives at:

```text
skills/easy-execute/
```
