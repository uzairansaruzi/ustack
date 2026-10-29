---
name: t3-batch
description: Work a batch of GitHub issues overnight in parallel T3 Code threads, one Sol-reviewed PR per issue. Invoke with /t3-batch.
disable-model-invocation: true
---

# T3 Batch

Hand a batch of GitHub issues to agents before bed. By morning, each issue
has a reviewed, CI-green pull request or a named reason it is blocked. You
also get a list of every call the agents made for you. Nothing merges
without you.

```
/t3-batch #12 #15 #19            # issue numbers
/t3-batch label <name>           # also: milestone <name>, epic #<n>
/t3-batch status | report | resume [run]
/t3-batch pause | stop [lane <k>]
/t3-batch merge [run]
```

Requires T3 Code (thread, delegation and PR-watch tools), an authenticated
`gh` with push access, and the [tstack](https://github.com/t3dotgg/tstack)
skills `sol-loop`, `file-pr`, `babysit-pr` and `leaving-pr-comments`.

## Roles and files

- **Lead**: this thread. Plans the run with the owner, launches lanes, keeps
  the ledger, and writes the report. It never edits the repository.
- **Lane**: a T3 thread in its own worktree. Lane `k` takes chain `k` from
  code to ready PRs by following [lane.md](references/lane.md).
- **Reviewer**: GPT Sol, which each lane starts through `sol-loop`.

The run lives in scratch outside every checkout:
`~/.t3-batch/<owner>-<repo>/<run>/`, where `<run>` is
`<YYYYMMDD-HHMM>-<selection>`. Each file has exactly one writer:

| File | Writer | Holds |
|---|---|---|
| `plan.md` | lead, frozen at approval | everything a lane needs to start cold |
| `ledger.md` | lead | lane thread IDs and links, chain states, PR URLs |
| `lane-<k>.md` | lane `k` | stage log, heads, review and repair counts, decisions |
| `report.md` | lead | the morning report |

## Phase A: the owner is here

The owner is present only during Phase A, so ask every question now. Phase A
ends with one approval message.

1. **Preflight.** Settle each item, or report it, before you read issues:
   - `gh auth status`, `git fetch origin`, and
     `gh repo view --json nameWithOwner,defaultBranchRef,viewerPermission`
     (WRITE or higher).
   - Load these T3 tools: `t3_thread_launch`, `t3_thread_send`,
     `t3_thread_read`, `t3_thread_search`, `t3_thread_interrupt`,
     `delegate_task`, `watch_pull_request`, `link_pull_request`. Call
     `orchestrator_capabilities` once, confirm it lists a GPT Sol model, and
     save the response in scratch.
   - Lanes run unattended, so they launch with `runtimeMode: "full-access"`.
     A thread cannot give a lane more access than it has itself. If this
     thread is not full-access, tell the owner. Never change modes silently.
   - The tstack skills above are installed in this harness's skills
     directory.
   - The lead thread ID is `$T3CODE_THREAD_ID`. If that is unset, find this
     thread with `t3_thread_search`.
   - Record `git status` of the main checkout. The run never changes it.
2. **Learn the project.** Read `AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING*`, the
   CI workflows, and the package manifest or Makefile. Write down three
   things so lanes never have to work them out again:
   - the **setup** command for a fresh worktree;
   - the **check** commands that CI runs (build, typecheck, lint, test);
   - any **exclusive resource** that two lanes would fight over: a fixed
     port, a simulator, a local database, a GPU. Assign one to each lane.
3. **Read and screen the issues.** Resolve the selection with `gh`. For an
   epic, use `gh api repos/<o>/<r>/issues/<n>/sub_issues`. For each issue,
   read the body and every comment:
   `gh issue view <n> --json title,body,comments,assignees,labels,state`.
   For more than about 15 issues, split the reading across read-only
   subagents. Each issue ends up included or skipped with a reason. Skip an
   issue when any of these is true:
   - It is closed, assigned to someone else, or already done. Look for an
     open or merged PR (`gh pr list --state all --search "<n>"`) and for
     commits (`git log origin/<default> --grep "#<n>"`).
   - It needs something only the owner can do: credentials, production
     data, another repo, a paid service, or a physical device.
   - Nobody could verify it is done. If one answer would fix that, ask the
     question in step 6 instead of skipping the issue.

   Note the dependencies: GitHub's blocked-by list
   (`gh api repos/<o>/<r>/issues/<n>/dependencies/blocked_by`), "depends on
   #n" in the text, and code that forces one issue to land first.
4. **Plan the chains.** A chain is a set of issues that must land in order.
   Each later issue builds on the branch of the issue before it, and its PR
   stacks on that branch. Unrelated issues are separate chains of one, even
   when they touch the same files: a rebase handles that. Order the chains
   longest first. The default lane cap is 2 on a machine with 16 GB of RAM
   or less (`sysctl hw.memsize`, or `/proc/meminfo`) and 3 above that. The
   cap is never more than the number of chains, and the owner can change it.
5. **Warm up.** Launch the first wave of lanes with the prep brief (see
   [Launching lanes](#launching-lanes)), **one at a time**. Wait for each
   lane's report before you launch the next: parallel cold installs and
   builds can push the machine into swap. If the default branch is red, or
   setup fails, fix it or report it now. At 2 AM nobody can.
6. **Ask once.** Send one message that the owner can read on a phone, in
   under about 300 words. Then end your turn and wait for the answer.
   - **Plan:** the chains in order (`#n title` for each), the lane cap, the
     lane model and effort, the Sol model, and the warm-up times.
   - **Questions** that only the owner can answer, about what users see,
     scope, or risk. Number them D1, D2, and so on. Give each a
     recommendation (`➡️ …`) and a reason. The owner can answer "accept all
     except D2: …". An unanswered question takes the recommendation.
   - **Skipped:** `#n <reason>`, one per line.
   - **Grants:** lanes may push `t3-batch/*` branches, open ready PRs, reply
     to and resolve review threads, rerun CI for evidenced flakes, and
     force-push their own branches with `--force-with-lease`. Lanes never
     merge, push the default branch, touch production, close issues or
     other people's PRs, or act in another repo. The owner can add grants,
     for example "lanes may file follow-up issues".

   An engineering call with one sensible answer is not a question. Make it
   and list it under Defaults in the plan.
7. **Start.** Write `plan.md`, with the owner's answers copied in word for
   word. Send each warm lane its go brief, then launch any more lanes up to
   the cap. Tell the owner how many issues and lanes are running, link each
   lane thread, and say that the report will arrive in this thread.

`plan.md` holds: the repo, the default branch, the main checkout path, the
scratch path, the lead thread ID, the setup and check commands with warm-up
times, the models, the grants, the lane cap, each lane's exclusive resource,
the skips, and each chain's issues in order. For each issue it has the
title, the approved decisions and defaults, a one-line acceptance test, and
notes.

## Launching lanes

Use `t3_thread_launch` with these settings:

- `title`: `t3-batch <run> · lane <k>`. This title is the lookup key.
- `clientRequestId`: `<run>-lane-<k>`. After a lost response, search for the
  title with `t3_thread_search` before you retry with the same ID.
- `workspaceStrategy`: `{type: "worktree", baseRef: "<default>",
  startFromOrigin: true, branch: "t3-batch/<run>-lane-<k>"}`. The lane makes
  its issue branches from there.
- `runtimeMode`: `"full-access"`. For `modelSelection`, pass the approved
  lane model, or omit it to inherit this thread's model.
- `message`: the prep brief during warm-up, or the go brief for lanes that
  launch later.

**Prep brief:** "t3-batch prep for lane k. Report your worktree path and
`git rev-parse HEAD`. Run the setup and check commands below, and time each
one. Send the results to lead thread `<id>` with `t3_thread_send` (mode
`queue`, request ID `<run>-lane-<k>-prep`), then stop. Make no branches,
commits or pushes." Include the commands in the brief, because `plan.md`
does not exist yet.

**Go brief:** "Read `<absolute path>/references/lane.md` and follow it.
Run `<run>`, lane `<k>`, plan `<scratch>/plan.md`, lead thread `<id>`." Send
it to a warm lane with `t3_thread_send`, mode `queue`, request ID
`<run>-lane-<k>-go`.

Record each lane's thread ID and link in `ledger.md` as soon as you have it.

## Phase B: overnight

**The owner is asleep.** From approval until the report, nothing waits on
the owner. Do not use a question tool, and never end a turn on a question.
When a call would go to the owner, take the safest choice that still meets
the issue's acceptance: the narrowest visible change, or the option that
destroys nothing. Record it as a decision with its alternative, and keep
going. The plan's grants are the outer limit, and no decision goes past
them.

The lead wakes on lane messages. It ends its turn between messages. On each
message:

- Update `ledger.md`. Verify new PRs and heads against GitHub instead of
  trusting the report, and `link_pull_request` each new PR to this thread.
- `built` means the lane has filed all its PRs and is only babysitting
  them, so launch the next unstarted chain. If lanes hold exclusive
  resources, launch the next chain on `done` instead.
- `blocked` means record the reason. Relaunch a chain only when the cause is
  gone, for example a T3 failure. Use `existing_worktree` with the lane's
  path, and send a brief that tells the lane to resume.
- `red-base` means the default branch fails its checks. Launch no new
  chains, and mark every issue in the unstarted chains `blocked: red base`.
  The lane that reported it is `done`, and its unfinished issues are
  blocked. Send each other running lane "Red base: file a PR for your
  current issue, mark your unstarted issues `blocked: red base`, and
  babysit what you filed until you can report `done`."

Write the report when every chain is `done`, meaning each issue in it is
ready or blocked.

Never poll: no sleep loops, no scheduled checks, no watcher agents. Each
lane watches its own PRs.

When a lane hits a provider usage limit, it stops, and T3 may resume it
after the reset. Never move a lane to another model or account unless the
plan names that fallback. Record the limit in the ledger and carry on.

## Report

Write `report.md` and post it in this thread.

1. Open with **Decisions made without you**: every decision in every lane
   file, in plain words, each with its alternative and a reversal code
   (`#42 alt1`).
2. A table per chain, bottom first: issue, PR link, state (ready, or blocked
   with the reason), CI on the latest head, sol-loop rounds, and repair
   passes used.
3. Follow-ups: findings that lanes declined or left at a limit, with PR
   links.
4. Manual checks to do before merging, taken from the PR bodies.
5. Everything that went wrong: flaky reruns with their evidence, usage
   limits, relaunches, and the stages that took longest.
6. The lane worktrees and their sizes. They stay until the merge walk.

Unwatch any PR this thread still watches. End with: "Run `/t3-batch merge`
when you have looked."

## Controls

None of these need a second confirmation.

- **status:** summarize the ledger and lane files. Check them against
  `t3_thread_read` for each lane and against GitHub.
- **pause [lane k]:** send "Pause: finish the current step and commit it,
  then set your lane file's state to `paused`" with `t3_thread_send`, mode
  `queue`. The lead launches nothing new until resume.
- **stop [lane k]:** `t3_thread_interrupt` the lanes, then send "Stopped:
  set your lane file's state to `stopped` and do nothing else". Record the
  stop in the ledger. Worktrees and commits stay as they are.
- **resume [run]:** use the named run, or else the newest one in scratch.
  Read the ledger, every lane file, each lane thread, and GitHub. Leave lanes
  that are still working alone. Send each paused or stopped lane "Resume:
  reconcile with your lane file and continue." Relaunch a lost thread with
  `existing_worktree` at its recorded path. Never redo a finished stage.

## Merge walk

`/t3-batch merge` needs the owner present, and the owner approves every
merge.

1. Go chain by chain, bottom PR first. For each PR, re-read its head. The
   required checks must be green on that head, the branch must be mergeable
   and up to date with its base, and every bot finding must be handled.
   Tell the owner in one or two sentences what the PR does, list its manual
   test steps, and ask. If a PR has manual steps, "merge" is not a pass:
   ask whether the steps passed.
2. Merge with the repo's usual method:
   `gh pr merge <n> --<method> --match-head-commit <sha>`. Never use
   `--admin` or `--delete-branch`, because a lane worktree may still have
   the branch checked out.
3. Confirm GitHub shows the PR as merged, then `git fetch origin`. The next
   PR in the stack still targets the merged branch, whatever the merge
   method, so run `gh pr edit <next> --base <default>`.
4. After a squash or rebase merge, restack the rest of the chain before you
   ask about the next PR. First record the current head of every remaining
   branch. Then, bottom first, rebase each branch onto its new parent,
   cutting at that parent's recorded old head:
   `git rebase --onto <new-parent> <old-parent-head> <branch>`. The first
   new parent is `origin/<default>`. Push each with `--force-with-lease`,
   and wait for green.
5. When the walk is over, offer cleanup: remove the run's worktrees through
   T3, delete the merged `t3-batch/*` branches, and archive the lane
   threads. Do it only when the owner says so.
