---
name: disk-sweep
description: Find what is filling your Mac's disk and reclaim it one decision at a time. Approved files go to a single Trash batch you can undo. The first run learns your setup, and later runs reuse what it learned.
disable-model-invocation: true
---

# disk-sweep

Reclaim disk space on the user's Mac. The sweep stays **read-only** until the user decides
each item. Everything moved goes into one **Trash batch** with a **manifest**, so every move
can be undone until the user empties the Trash. A tool's own cleanup command (`brew cleanup`,
`simctl delete`, `docker prune`) deletes directly and can't be undone. Never empty the Trash
and never use `sudo`.

The skill learns each machine. The user's hotspots, keep list and quirks live in a **profile**
at `~/.config/disk-sweep/profile.md`. It sits outside the skill folder, so updating the skill
keeps it, and every agent shares the same profile.

## 0. Load the profile

If `~/.config/disk-sweep/profile.md` exists, read it and go to **Sweep**. If it doesn't, run
**Intake** first and then continue straight into the sweep in the same session.

## Intake (first run only)

Keep it short: detect everything you can, then ask one round of questions.

1. **Detect.** Don't ask about anything you can check:
   - Toolchains: `command -v` for `xcrun`, `docker`, `brew`, `npm`, `pnpm`, `yarn`, `bun`,
     `pip3`, `uv`, `conda`, `cargo`, `go`, `gradle`, `pod`, plus `~/Library/Android/sdk`.
   - Admin rights: `id -Gn | grep -qw admin`.
   - The biggest folders: `du -k -d 1` on `~`, `~/Library`, `~/Library/Caches`,
     `~/Library/Application Support`, `~/Library/Developer` and `/Applications`, top 10 of
     each. Bound each call (see Gotchas).
   - Project roots: parents of `.git` folders up to 3 levels under `~`.
2. **Ask one message** with numbered questions and a default for each, so "defaults" is a
   valid answer:
   1. What you found: the toolchains and the 10 biggest folders. "Anything here you already
      suspect, or anything that grows fast?"
   2. "What should I never touch?" Suggest likely candidates from what you found: sync
      caches (Google Drive, Dropbox), agent session history, model downloads, signing keys,
      phone backups.
   3. The detected project roots. "Right list?"
   4. If Xcode is installed, list the simulators. "Which do you keep?" Default: keep all of
      the available ones and delete the unavailable ones.
   5. The staleness cutoffs. Default: dependencies in projects untouched for 90 days, and
      logs older than 30 days.
3. **Write the profile** from `references/profile-template.md`. List the sections of
   `references/hotspots.md` whose toolchain exists, then add the user's own hotspots and keep
   list. Don't copy catalog rows into the profile: each sweep reads them from the catalog, so
   fixes to the skill reach every user. Done when the file exists and you have told the user
   its path in one line.

## Sweep

1. **Baseline.** Record `df -h /System/Volumes/Data`. Done when you have the used and free
   figures.
2. **Measure every hotspot**: the rows of the catalog sections the profile lists, plus the
   profile's own rows. Done when each row has a size or says "nothing to reclaim". If free
   space is still low after that, size `~`, `~/Library` and `/Applications` from the top down
   to look for new growth.
3. **Ask one round of questions.** Leave out anything on the keep list. Number every item and
   give its size, what it is, the risk in a sentence, and your recommendation. Mark items a
   tool command deletes directly as "can't be undone". Group items into **Safe** (regenerates
   or is obsolete) and **Your call**. A standing approval makes your recommendation "yes", but
   it still needs the user's answer this run. End with this line: *Reply like "all safe, 7 no,
   9 yes". Add "always" or "never" to an item to make it a standing decision.* Done when every
   item has an answer.
4. **Act on the approved items.** Before each group, check that nothing is using it (see
   Gotchas). Move paths with `<this skill's folder>/scripts/sweep-mv <path>...`, calling the
   script by its absolute path and passing absolute paths. Run a tool's own cleanup command
   only where the hotspot row names one. Done when every approved source path is gone and has
   a line in the manifest. Report every line the script prints that isn't a plain `moved`,
   whatever its exit code.
5. **Report.** Give free space before and after, the total waiting in the Trash, anything
   skipped and why, and a paste-ready list of anything that needs admin rights. Done when the
   user knows the space comes back once they empty the Trash.
6. **Refine the profile.** This step is what makes the next sweep faster. Update the profile:
   - Each "never" goes on the keep list and each "always" goes under standing approvals.
   - Add new growth you found in step 2 as a row under "My hotspots", with its measure and
     action.
   - Add catalog sections for toolchains installed since the last run, and drop sections
     for toolchains that are gone.
   - Add a gotcha for anything that surprised you: a hang, a permission error, a misleading
     `du`.
   - Add a history line.

   Done when you have told the user what changed in the profile in one or two lines.

## Trash batch

`scripts/sweep-mv` moves each path to `~/.Trash/disk-sweep-<YYYY-MM-DD>/` and adds a line to
`~/.config/disk-sweep/manifests/<YYYY-MM-DD>.tsv` with the size in KB, the original path, and
where it went. Each item gets its own folder in the batch, named after its flattened path
(`Library/Caches/Foo` goes to `Library__Caches__Foo/Foo`), and nothing is ever overwritten.
It refuses `/`, `~`, and paths already in the Trash. When a tool command deletes something directly, add its manifest line yourself and use
`-` as the batch path.

To undo, move each batch path back to its original path from the manifest. Finder's Put Back
doesn't work for these moves.

## Gotchas

- **`~/.Trash` is write-only for the shell.** `mv` and `mkdir` work, but listing it doesn't.
  Confirm a move by checking that the source is gone.
- **Bound slow commands.** macOS has no `timeout`, so use
  `perl -e 'alarm 60; exec @ARGV' <command>`. Bound every `du` on a big tree and every
  `xcrun simctl` call. A `simctl` hang means CoreSimulator is wedged: report it, don't fight
  it.
- **`du` lies about clones.** APFS clones (simulator test clones in `XCTestDevices`, copied
  files) share their blocks, so `du` overcounts. Report the count and say the real saving is
  smaller.
- **Cloud folders.** If iCloud Drive, Dropbox or Google Drive syncs a folder, trashing a file
  there deletes it everywhere. Files kept only in the cloud still show their full size. Put
  anything inside a synced folder under "Your call".
- **Quit the app first.** Only clear an app's cache when `pgrep -x <process>` finds nothing.
  Use `-x`, because `pgrep -f` matches your own shell command. When you must match a full
  command line, bracket one letter (`pgrep -f '[G]radleDaemon'`) so the pattern can't match
  itself.
- **npx folders in use.** Some tools (like `chrome-devtools-mcp`) rename their process, so
  `ps` won't show which `~/.npm/_npx` folder they run from. Treat an npx folder as in use if
  its `package.json` names a package that a running `npm exec <pkg>` is serving.
- **Sandboxed and system folders.** Moves inside other apps' `~/Library/Containers` folders
  and anything root-owned fail with "Operation not permitted". Without admin rights, put them
  on the admin list instead of retrying.
- **Freed space arrives late.** Deleted simulators and Docker's disk image shrink over
  minutes. Finder's "available" figure counts purgeable space and `df` doesn't, so the two
  disagree.
- **zsh stops on unmatched globs.** Run `setopt null_glob` before looping over globs.
