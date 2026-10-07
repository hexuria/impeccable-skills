# impeccable

One word instead of a skill name. Say "impeccable this" or "make this
faster" — the skill detects the language and runs the right impeccable
member skill. You never name the language skill.

## Install

Copy `skill/` into your agent's skills folder under the name `impeccable`:

```sh
git clone https://github.com/hexuria/impeccable-skills
mkdir -p ~/.claude/skills
cp -r impeccable-skills/skill ~/.claude/skills/impeccable
```

To get updates with `git pull`, link the folder instead of copying it:

```sh
ln -s "$PWD/impeccable-skills/skill" ~/.claude/skills/impeccable
```

The folder name must be `impeccable`, matching the skill's `name`. For
other agents, put the folder wherever they load skills from, for example
`.claude/skills/` or `.cursor/skills/` inside a project.

Member skills install themselves: when routing lands on a language whose
skill is missing, the agent clones the member repo and copies its
`skill/` folder in next to this one. If the agent cannot reach the
network, it gives you the one-line install to run yourself.

## Use it

```text
impeccable this
make this parser faster
verify this change properly
harden this api
audit this crate
```

## Languages

Rust and Python today — the list is [`skill/registry.yaml`](skill/registry.yaml).
Work in another language gets an honest "not covered," never a fake route.

## Add a language

Add one row to `skill/registry.yaml`: `language`, `skill`, `repo`, and
the signals that detect it. That is the whole change — the orchestrator
holds no language rules of its own.

## License

MIT
