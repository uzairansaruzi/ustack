# ustack

Agent skills I share with other [T3 Code](https://github.com/pingdotgg/t3code)
users. They work in Claude Code, Codex, Hermes, and more. They build on
[tstack](https://github.com/t3dotgg/tstack).

## Skills

| Skill | Say | What it does |
|---|---|---|
| `disk-sweep` | `/disk-sweep` | Finds what is filling your Mac's disk and walks you through reclaiming it, one decision at a time. Approved files go to a single Trash batch you can undo. The first run learns your setup, and later runs reuse it. |
| `duo` | `duo with <model>` | Works your task independently alongside a partner model, checks both results, and gives you one merged answer. |
| `t3-batch` | `/t3-batch label <name>` | Plans a batch of GitHub issues with you, then works them overnight in parallel T3 threads. By morning there is one Sol-reviewed PR per issue and a list of every call made for you. Nothing merges without you. |

## Install

`t3-batch` uses these tstack skills: `sol-loop`, `file-pr`, `babysit-pr`, and
`leaving-pr-comments`. Install them too.

```bash
git clone https://github.com/uzairansaruzi/ustack
git clone https://github.com/t3dotgg/tstack
mkdir -p ~/.claude/skills && cp -R ustack/skills/* tstack/skills/* ~/.claude/skills/   # Claude Code
mkdir -p ~/.codex/skills && cp -R ustack/skills/* tstack/skills/* ~/.codex/skills/     # Codex
```

## Requirements

`disk-sweep` is macOS only. It saves what it learns about your machine in
`~/.config/disk-sweep/profile.md`, so updating the skill keeps your setup.

`duo` needs access to the model you name, through T3 delegation or a native
subagent outside T3.

`t3-batch` requires:

- [T3 Code](https://github.com/pingdotgg/t3code). `t3-batch` launches its
  lanes as T3 threads in their own worktrees and watches PRs through T3.
- A GPT Sol model in T3's provider list, for `sol-loop` reviews.
- An authenticated [GitHub CLI](https://cli.github.com) (`gh`) with push
  access to the repo.

## Make them yours

These are starting points, so edit them. Put personal defaults, like the
lane model or how many lanes to run, in your `CLAUDE.md` or `AGENTS.md`. Then
you can update the skills without losing your settings.

## License

[MIT](LICENSE)
