# ustack

Agent skills I share with other [T3 Code](https://github.com/pingdotgg/t3code)
users. They work in Claude Code and Codex. They build on
[tstack](https://github.com/t3dotgg/tstack).

## Skills

| Skill | Say | What it does |
|---|---|---|
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
