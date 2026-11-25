# Hotspot catalog

Common places a developer Mac fills up. The profile lists which sections apply to the user,
and every sweep reads those sections' rows from here. "Trash" means `scripts/sweep-mv`.
Paths are examples: confirm each one exists before you measure it.

## Everyone

| Hotspot | Measure | Action |
|---|---|---|
| App caches, `~/Library/Caches/*` | `du` per folder, top 15 | Trash once the app is quit. Browser caches grow straight back, so offer them only when they're huge. |
| Installers in `~/Downloads` | `.dmg`, `.pkg`, `.zip`, `.xip` for apps already in `/Applications` | Trash |
| Large old downloads | Files over 500 MB in `~/Downloads` untouched for 90 days | Your call |
| Old logs, `~/Library/Logs` | Files older than the profile's log cutoff | Trash |
| Leftovers from deleted apps, `~/Library/Application Support/*` | Folders whose app is gone from `/Applications` and `~/Applications` | Your call: some belong to CLI tools |
| Phone backups, `~/Library/Application Support/MobileSync/Backup` | `du` per backup, with dates | Your call. Never a safe item. |
| Time Machine local snapshots | `tmutil listlocalsnapshots /` | Report only. macOS purges them when space runs low. |
| Old agent CLI versions | e.g. `~/.local/share/claude/versions`, `~/.local/share/cursor-agent/versions` | Trash all but the version the launcher symlink points to and the version the harness hosting you runs from |
| Git leftovers | Stale worktrees (`git worktree list`: clean, merged, untouched for 48 hours, no process with it as cwd per `lsof -a -d cwd`). `tmp_pack_*` files in `.git/objects/pack` older than 60 minutes. | Trash the worktree. Don't run `git worktree prune`: it deletes the metadata a restored worktree needs, and git clears stale entries itself. If git has already cleared it when you restore one, recreate it with `git worktree add <path> <branch>`. Trash the tmp packs. |

## Xcode and iOS (`xcrun`)

| Hotspot | Measure | Action |
|---|---|---|
| DerivedData, `~/Library/Developer/Xcode/DerivedData/*` | Stale if the `WorkspacePath` in its `info.plist` no longer exists, or it's older than the dormant cutoff. Keep every `*.noindex` folder: they're shared caches, and every project pays to rebuild them. | Trash. It all regenerates, but active projects pay for a full rebuild. |
| Unavailable simulators | `xcrun simctl list devices unavailable` | `xcrun simctl delete unavailable`, or `xcrun simctl delete <udid>` per device if the user keeps any |
| Simulators outside the keep set | `xcrun simctl list -j devices` | `xcrun simctl delete <udid>` |
| Test clones, `~/Library/Developer/XCTestDevices` | Count the folders. `du` overcounts these clones. | `xcrun simctl --set testing delete all` while `pgrep -x xcodebuild` finds nothing |
| Old simulator runtimes | `xcrun simctl runtime list` | Your call: `xcrun simctl runtime delete <id>` |
| Simulator caches, `~/Library/Developer/CoreSimulator/Caches` | `du` | Trash while no simulator is booted |
| Simulator symbolication caches, `<dataPath>/Library/Caches/com.apple.coresymbolicationd`, where `dataPath` comes from `xcrun simctl list -j devices` | `du` per kept simulator. These can reach tens of GB. Leave `PRBPosterExtensionDataStore` alone: it's the wallpaper render cache and grows straight back. | Trash, only on simulators that are shut down |
| Device support, `~/Library/Developer/Xcode/*DeviceSupport/*` | List by OS version | Trash all but the newest per platform |
| Archives, `~/Library/Developer/Xcode/Archives` | `du` per month | Your call: shipped builds need their dSYMs for crash symbolication |
| Previews, `~/Library/Developer/Xcode/UserData/Previews` | `du` | Trash |
| CocoaPods, `~/Library/Caches/CocoaPods` | `du` | `pod cache clean --all` |

## JavaScript

| Hotspot | Measure | Action |
|---|---|---|
| npm, `~/.npm/_cacache` and `~/.npm/_npx/*` | `du` | `npm cache clean --force`. Trash the npx folders that aren't in use (see Gotchas). |
| pnpm store | `pnpm store path` | `pnpm store prune` |
| Yarn, `~/Library/Caches/Yarn` | `du` | `yarn cache clean` |
| Bun, `~/.bun/install/cache` | `du` | `bun pm cache rm` |
| Old Node versions, `~/.nvm/versions/node/*` (or fnm, volta) | List them | Trash all but the default and the versions projects pin |
| Dormant build output | `node_modules`, `.next`, `.turbo`, `dist` in projects untouched past the cutoff | Trash |

## Python

| Hotspot | Measure | Action |
|---|---|---|
| pip, `~/Library/Caches/pip` | `du` | `pip3 cache purge` |
| uv, `~/.cache/uv` | `du` | `uv cache prune` |
| Conda | `conda info` for the pkgs dir | `conda clean --all` |
| Dormant venvs | `.venv` and `venv` in projects untouched past the cutoff that have a `requirements*.txt` or `pyproject.toml` | Trash |
| Model downloads, `~/.cache/huggingface`, `~/.ollama/models` | `du` per model | Your call. Never a safe item. |

## Rust, Go, JVM and Android

| Hotspot | Measure | Action |
|---|---|---|
| Cargo, `~/.cargo/registry/{cache,src}` | `du` | Trash. It redownloads. |
| Dormant `target/` folders | In Rust projects untouched past the cutoff | Trash |
| Old Rust toolchains | `rustup toolchain list` | `rustup toolchain uninstall <name>` |
| Go | `go env GOMODCACHE GOCACHE` | `go clean -modcache` and `go clean -cache` |
| Gradle, `~/.gradle/caches` | `du` | Trash while no Gradle daemon runs (`pgrep -f '[G]radleDaemon'`) |
| Maven, `~/.m2/repository` | `du` | Trash |
| Android emulators, `~/.android/avd` | List them | Your call: delete in Android Studio's Device Manager |
| Android system images, `~/Library/Android/sdk/system-images` | List by API level | Your call |

## Docker

| Hotspot | Measure | Action |
|---|---|---|
| Images, build cache, stopped containers | `docker system df` | `docker builder prune -f` and `docker image prune -a -f`. Volumes hold data, so they're your call. |

## Homebrew

| Hotspot | Measure | Action |
|---|---|---|
| Old versions and downloads | `brew cleanup -n` and `brew autoremove -n` | `brew cleanup --prune=all` and `brew autoremove` |
