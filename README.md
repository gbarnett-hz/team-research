# team-research

A [skills.sh](https://skills.sh) skill for exhaustive, cited, adversarially-tested research into any topic. Output is a verbose markdown file named after the topic slug (`SLUG.md`).

## Install

```
npx skills add gbarnett-hz/team-research
```

## Use

In Claude Code, ask for it by name:

```
team-research: Raft consensus in practice
```

Flow: scouts map the landscape and you pick areas of focus, then a team of researchers (including an adversarial one) digs into every area, an arbiter rules on each claim on merit, and an anti-slop pass cleans the prose before `SLUG.md` is written. Raw ledgers and rulings are kept in `.team-research/SLUG/`.

See [`skills/team-research/SKILL.md`](skills/team-research/SKILL.md) for details.
