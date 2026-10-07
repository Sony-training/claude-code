# Claude Code Reference

Quick lookup for the training. Every entry was checked against the official documentation at `code.claude.com/docs` on 2026-10-07 (pages: setup, cli-reference, commands, interactive-mode, keybindings, terminal-config, vs-code, claude-directory, memory, settings, settings-reference, permissions, permission-modes, managed-settings, hooks, env-vars, model-config, fast-mode, sandboxing, glossary). Claude Code ships often: confirm with `claude --version`, `/help` and the Docs Map at the end before class.

Conventions

- **T** terminal only · **V** VS Code panel only · **both** works in both
- `~` is your home directory. On Windows it is `%USERPROFILE%` (for example `C:\Users\<you>`).
- "macOS / Linux / WSL" and "Windows (native)" are separate columns. Claude Code inside WSL uses the Linux paths of that WSL distribution.
- "v2.1.xxx+" means the feature needs at least that Claude Code version.

---

## 1. Install and Verify

| Task | macOS / Linux / WSL | Windows (native) |
|---|---|---|
| Native installer (recommended, auto-updates) | `curl -fsSL https://claude.ai/install.sh \| bash` | PowerShell: `irm https://claude.ai/install.ps1 \| iex` |
| Native installer, alternative shell | n/a | CMD: `curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd` |
| Specific version or channel | `curl -fsSL https://claude.ai/install.sh \| bash -s 2.1.89` (or `stable`, `latest`) | PowerShell: `& ([scriptblock]::Create((irm https://claude.ai/install.ps1))) 2.1.89` (or `stable`) |
| Package manager | Homebrew: `brew install --cask claude-code` (stable) or `claude-code@latest` · apt, dnf, apk: signed repositories (docs: "Install with Linux package managers") | WinGet: `winget install Anthropic.ClaudeCode` |
| npm (Node.js 22+, never `sudo`) | `npm install -g @anthropic-ai/claude-code` | same command |
| Verify | `claude --version` | `claude --version` |
| Diagnose | `claude doctor` | `claude doctor` |
| Update (native) | auto in background, or `claude update` | same |
| Update (package manager) | `brew upgrade claude-code` · `sudo apt update && sudo apt upgrade claude-code` · `sudo dnf upgrade claude-code` · `apk update && apk upgrade claude-code` | `winget upgrade Anthropic.ClaudeCode` |
| Log in | run `claude`, follow the browser prompt | same |
| Auth status | `claude auth status` (`--text` for readable output; exit 0 if logged in) | same |
| VS Code extension | Extensions view → "Claude Code" (id `anthropic.claude-code`); also `code --install-extension anthropic.claude-code` (standard VS Code CLI) | same |

Requirements: macOS 13+, Ubuntu 20.04+, Debian 10+, Alpine 3.19+, Windows 10 1809+ or Windows Server 2019+; 4 GB+ RAM; x64 or ARM64; internet; shell: Bash, Zsh, PowerShell or CMD. Account: Pro, Max, Team, Enterprise or Console (the free claude.ai plan has no Claude Code access). VS Code 1.94.0 or later for the extension.

Windows notes

| Topic | Detail |
|---|---|
| Git for Windows | Optional. With it, Claude Code uses Git Bash for the Bash tool. Without it, it uses the PowerShell tool |
| Git Bash not found | `{"env": {"CLAUDE_CODE_GIT_BASH_PATH": "C:\\Program Files\\Git\\bin\\bash.exe"}}` in settings |
| Admin rights | Not needed to install |
| WSL 2 vs native | WSL 2 supports sandboxing; native Windows and WSL 1 do not. Use WSL 2 for Linux toolchains |
| Alpine | `apk add bash curl libgcc libstdc++ ripgrep` and `USE_BUILTIN_RIPGREP=0` |

Install locations (native installer)

| Item | macOS / Linux | Windows (native) |
|---|---|---|
| Launcher | `~/.local/bin/claude` (symlink) | `%USERPROFILE%\.local\bin\claude.exe` |
| Versions | `~/.local/share/claude/versions/` | `%USERPROFILE%\.local\share\claude` |

Uninstall (native)

| macOS / Linux / WSL | Windows PowerShell |
|---|---|
| `rm -f ~/.local/bin/claude` and `rm -rf ~/.local/share/claude` | `Remove-Item "$env:USERPROFILE\.local\bin\claude.exe" -Force` and `Remove-Item "$env:USERPROFILE\.local\share\claude" -Recurse -Force` |
| Settings and state: `rm -rf ~/.claude` and `rm ~/.claude.json` | `Remove-Item "$env:USERPROFILE\.claude" -Recurse -Force` and `Remove-Item "$env:USERPROFILE\.claude.json" -Force` |

Homebrew: `brew uninstall --cask claude-code` · WinGet: `winget uninstall Anthropic.ClaudeCode` · npm: `npm uninstall -g @anthropic-ai/claude-code`.

---

## 2. CLI Commands and Flags

`claude --help` does not list every flag; absence from `--help` does not mean unavailable.

### Commands

| Command | Meaning |
|---|---|
| `claude` | Interactive session |
| `claude "query"` | Interactive session with a first prompt |
| `claude -p "query"` | Non-interactive: print the answer and exit |
| `cat f \| claude -p "query"` | Pipe content in |
| `claude -c` / `--continue` | Continue the most recent conversation in this directory |
| `claude -r "<session>" "query"` | Resume by ID or name (`--resume` with no value opens a picker) |
| `claude update` | Update to the latest version |
| `claude install [version]` | Install or reinstall the native binary (`stable`, `latest` or a version) |
| `claude auth login` / `logout` / `status` | Account (`login` accepts `--email`, `--sso`, `--console`) |
| `claude setup-token` | Generate a long-lived OAuth token for CI (printed, not saved; needs a subscription) |
| `claude doctor` | Read-only install and settings diagnostics, no session |
| `claude mcp ...` | Configure MCP servers (`add`, `login <name>`, `logout <name>`) |
| `claude plugin ...` | Manage plugins (alias `claude plugins`) |
| `claude agents` | Open agent view for background sessions (`--json`, `--cwd <path>`) |
| `claude attach <id\|name>` · `logs` · `stop` · `respawn` · `rm` | Manage background sessions |
| `claude purge [path]` | Delete local state for a project (`--dry-run`, `-y`, `--all`) |
| `claude remote-control` | Remote Control server for claude.ai or the Claude app |
| `claude ultrareview [target]` | Non-interactive deep review (prints findings; `--json`) |
| `claude auto-mode defaults` / `reset` | Inspect or reset auto-mode classifier rules |
| `claude import [codex\|gemini\|cursor]` | Import config from other coding agents |

### Flags

| Flag | Meaning |
|---|---|
| `-p`, `--print` | Non-interactive |
| `-c`, `--continue` · `-r`, `--resume` | Continue or resume |
| `-n`, `--name <name>` | Name the session (shown in `/resume`) |
| `--fork-session` | New session ID when resuming |
| `--session-id <uuid>` | Use a specific session ID |
| `--model <alias\|id>` | Model for this session (`sonnet`, `opus`, `haiku`, `fable`, full ID) |
| `--fallback-model <list>` | Fallback chain if the primary is overloaded |
| `--effort <level>` | `low` `medium` `high` `xhigh` `max` or `ultracode` (levels depend on model) |
| `--permission-mode <mode>` | `default` `acceptEdits` `plan` `auto` `dontAsk` `bypassPermissions` (`manual` = alias of `default`, v2.1.200+) |
| `--dangerously-skip-permissions` | = `bypassPermissions` |
| `--allow-dangerously-skip-permissions` | Add `bypassPermissions` to the `Shift+Tab` cycle without starting in it |
| `--add-dir <path>` | Extra working directories (file access; not UNC `\\server\share`, map a drive letter instead) |
| `-w`, `--worktree [name]` | Isolated git worktree at `<repo>/.claude/worktrees/<name>` (`#<n>` or a PR URL branches from a PR) |
| `--tmux` | tmux session for the worktree (needs `--worktree`) |
| `--allowedTools` / `--disallowedTools` | Allow without prompt / deny rules (a bare tool name in `--disallowedTools` removes the tool) |
| `--tools` | Restrict which built-in tools exist (`""` none, `"default"`, or a list) |
| `--agent <name>` · `--agents '<json>'` | Run as a subagent definition · define subagents inline |
| `--output-format` | `text` `json` `stream-json` (with `-p`) |
| `--input-format` | `text` `stream-json` (with `-p`) |
| `--max-turns <n>` | Limit agentic turns (`-p`); errors when reached |
| `--max-budget-usd <n>` | Spend cap (`-p`); client-side estimate |
| `--json-schema '<schema>'` | Validated JSON output (`-p`) |
| `--append-system-prompt` / `-file` | Add to the default system prompt |
| `--system-prompt` / `-file` | Replace the system prompt |
| `--settings <file\|json>` | Settings overlay for this session (file ≤ 2 MiB) |
| `--setting-sources user,project,local` | Load only these sources |
| `--mcp-config` / `--strict-mcp-config` | Load MCP servers from file / use only those |
| `--plugin-dir <path>` | Load a plugin for this session only (repeat the flag) |
| `--bare` | Skip hooks, skills, plugins, MCP, auto memory, CLAUDE.md (fast scripted runs; = `CLAUDE_CODE_SIMPLE=1`) |
| `--safe-mode` | Disable all customizations to troubleshoot (auth, model, built-in tools, permissions still work) |
| `--restricted` | Locked-down mode for shared machines (v2.1.248+) |
| `--bg`, `--background` | Start as a background agent |
| `--cloud` | Start a cloud session (`--remote` is the deprecated alias) |
| `--teleport` | Resume a cloud session in the terminal |
| `--desktop` | Open the current directory in the Desktop app |
| `--ide` · `--chrome` / `--no-chrome` | Auto-connect IDE · Chrome integration |
| `--from-pr <n\|url>` | Sessions linked to a pull request |
| `--debug[=filter]` / `--debug-file <path>` | Debug logs (filter binds only in the `=` form) |
| `--verbose` | Full turn-by-turn output |
| `--no-session-persistence` | Do not save the session (`-p`) |
| `--init-only` | Run Setup and SessionStart hooks, then exit |
| `-v`, `--version` | Version |

---

## 3. Slash Commands

Type `/` to list. A command is recognized only at the start of the message. The CLI has all; the VS Code panel has a subset (type `/` there). Items marked *skill* are bundled skills that give Claude a prompt rather than fixed logic. Run `/help` for your version.

### Setup, memory, config

| Command | Does |
|---|---|
| `/init` | Create or improve CLAUDE.md (`CLAUDE_CODE_NEW_INIT=1` for the interactive multi-phase flow) |
| `/memory` | Edit CLAUDE.md files, toggle auto memory, view its entries |
| `/permissions` | Allow, ask, deny rules by scope; working directories; recent auto-mode denials |
| `/config [key=value]` | Settings UI (theme, model, editor mode, output style, project instructions, update channel) |
| `/status` | Settings → Status tab: version, model, account, connectivity, setting sources |
| `/doctor [prompt-audit [path]]` | *skill.* Setup checkup that can fix problems; `prompt-audit` audits instruction files |
| `/login` · `/logout` | Sign in / out |
| `/theme` · `/tui [default\|fullscreen]` | Color theme · terminal renderer |
| `/keybindings` | Open `~/.claude/keybindings.json` |
| `/terminal-setup` | Shift+Enter binding for VS Code, Cursor, Alacritty < 0.16, Zed; Option+Enter on Apple Terminal |
| `/ide` | IDE integrations and status |
| `/update-config [request]` | *skill.* Describe a settings change; Claude edits `settings.json` |
| `/fewer-permission-prompts` | *skill.* Build an allowlist from your transcripts |

### During a task

| Command | Does |
|---|---|
| `/plan [task]` | Enter plan mode (optionally start on a task) |
| `/model [model]` | Switch model (saved as default; `s` in the picker = this session only) |
| `/effort [level\|auto\|status\|ultracode [on\|off]]` | Effort level; no argument opens a slider |
| `/fast [on\|off]` | Toggle fast mode |
| `/context [all]` | Context usage grid, memory files loaded, optimization hints |
| `/compact [focus]` | Summarize the conversation to free context |
| `/autocompact [auto\|<tokens>]` | Auto-compact window (`500k`) |
| `/btw [question]` | Side question that does not enter the conversation |
| `/diff` | Review working-tree changes |
| `/code-review` (`/review`) | *skill.* Review a diff, PR, branch or path |
| `/security-review` | Security review of the branch diff (needs an `origin` remote) |
| `/simplify [target]` | *skill.* Review changed code and apply cleanups |
| `/verify` · `/run` | *skill.* Build and run the app to confirm a change works |
| `/goal [condition\|clear]` | Keep working across turns until a condition is met |
| `/loop [interval] [prompt]` | *skill.* Repeat a prompt while the session is open |
| `/batch <instruction>` | *skill.* Split a change into 5 to 30 worktree agents |
| `/add-dir <path>` · `/cd <path>` | Add a working directory · move the session to another directory |
| `/sandbox` | Toggle sandbox mode (macOS, Linux, WSL 2 only) |
| `/debug [description]` | *skill.* Turn on debug logging and read the session log |
| `/focus` | Toggle focus view (last prompt, tool summary, final answer) |

### Sessions

| Command | Does |
|---|---|
| `/clear [name]` | New conversation with empty context (previous stays in `/resume`) |
| `/resume [session]` | Pick or resume an earlier conversation |
| `/branch [name]` | Branch the conversation at this point |
| `/rewind` (`/checkpoint`, `/undo`) | Restore code and/or conversation, or summarize from a message |
| `/rename [name]` | Name the session |
| `/export [file]` | Export the conversation as plain text |
| `/copy [N]` | Copy the last (Nth-latest) response |
| `/recap` | One-line summary of the session |
| `/tasks` (`/bashes`) | Background shells and subagents |
| `/background [prompt]` · `/fork [prompt]` · `/subtask <task>` | Detach to a background agent · copy into a background session · forked background subagent |
| `/usage` (`/cost`, `/stats`) | Cost, plan limits, activity |
| `/exit` (`/quit`) | Leave |

### Extensions and integrations

| Command | Does |
|---|---|
| `/mcp [reconnect\|enable\|disable ...]` | MCP servers, status, OAuth |
| `/hooks` | View configured hooks |
| `/skills` | List skills (`t` sorts by token cost) |
| `/agents` | Prints a reminder to ask Claude to create subagents or edit `.claude/agents/`; opened an interface in v2.1.197 and earlier |
| `/list-agents` | Subagents, teammates and sessions Claude can message |
| `/plugin` · `/reload-plugins` · `/reload-skills` | Plugins menu · reload without restart |
| `/statusline` · `/output-style [style]` | Status line · response style |
| `/schedule` · `/remote-control` · `/teleport` (`/tp`) · `/desktop` (`/app`) | Routines · Remote Control · pull a cloud session here · continue in Desktop |
| `/import [codex\|gemini\|cursor]` | Bring config from other agents |
| `/ultrareview [PR\|branch]` | Cloud multi-agent review |
| `/powerup` · `/insights` · `/release-notes` · `/help` · `/bug` · `/feedback` | Learn · usage report · changelog · help · report |

Removed or changed: `/vim` (use `/config` → Editor mode), `/pr-comments` (removed v2.1.91), `/ultraplan` (use plan mode). Commands and skills are one mechanism: `.claude/skills/<name>/SKILL.md` and `.claude/commands/<name>.md` both create `/<name>`.

Prompt keyword: `ultrathink` in a prompt asks for deeper reasoning for that turn only. `ultracode` is a mode (via `/effort ultracode` or `--effort ultracode`) that lets Claude use dynamic workflows, not a prompt word.

---

## 4. Keyboard Shortcuts

Shortcuts vary by terminal. On macOS, Option shortcuts (`Alt+B/F/D/Y`, `Option+Enter`) need "Use Option as Meta" in the terminal (`/terminal-setup` handles Apple Terminal; iTerm2: Profiles → Keys → Left/Right Option = "Esc+"; VS Code: `"terminal.integrated.macOptionIsMeta": true`). `Option+T` works without it; `Option+P` also needs it. Remap anything except the reserved keys with `/keybindings`.

### Terminal (T)

| Action | macOS | Windows / Linux |
|---|---|---|
| Stop current turn (keeps work done) | `Esc` | `Esc` |
| Empty prompt: rewind menu · text in prompt: clear draft | `Esc Esc` | `Esc Esc` |
| Cycle permission modes | `Shift+Tab` | `Shift+Tab` (Windows without VT input mode: `Alt+M`) |
| Interrupt; clear input; second press exits | `Ctrl+C` | `Ctrl+C` |
| Exit (second press within 800 ms) | `Ctrl+D` | `Ctrl+D` |
| Edit prompt in external editor | `Ctrl+G` or `Ctrl+X Ctrl+E` | same |
| Transcript viewer | `Ctrl+O` | `Ctrl+O` |
| Search prompt history | `Ctrl+R` | `Ctrl+R` |
| Background running Bash or agent | `Ctrl+B` (tmux: twice) or `Ctrl+X Ctrl+B` | same |
| Show or hide Claude's task checklist | `Ctrl+T` | `Ctrl+T` |
| Stash or restore prompt | `Ctrl+S` | `Ctrl+S` |
| Redraw screen | `Ctrl+L` | `Ctrl+L` |
| Suspend (then `fg`) | `Ctrl+Z` | Unix only (not native Windows) |
| Paste image | `Ctrl+V` (`Cmd+V` in iTerm2) | `Alt+V` on Windows and WSL (WSL binds both `Ctrl+V` and `Alt+V`); `Ctrl+V` on Linux |
| Switch model | `Option+P` | `Alt+P` |
| Toggle extended thinking | `Option+T` | `Alt+T` (no effect on Opus 5.5, Sonnet 5.5, Fable: always on) |
| Toggle fast mode | `Option+O` | `Alt+O` |
| Send queued messages now | `Ctrl+Enter` or `Ctrl+X Ctrl+S` | same (v2.1.275+) |
| Stop all background subagents (press twice within 3 s) | `Ctrl+X Ctrl+K` | same |
| Undo last input edit | `Ctrl+_` or `Ctrl+Shift+-` | same |
| Accept autocomplete / comment on a permission answer | `Tab` | `Tab` |
| History (when on first or last row) | `Up` `Down` or `Ctrl+P` `Ctrl+N` | same |

Line editing: `Ctrl+A` start, `Ctrl+E` end, `Ctrl+K` delete to end, `Ctrl+U` delete to start, `Ctrl+W` delete back to whitespace (whole path), `Ctrl+Y` paste deleted text; `Alt+B` / `Alt+F` / `Alt+D` word move or delete; delete one word with `Option+Delete` (macOS) or `Ctrl+Backspace` (Windows). Windows Backspace deleting a whole word: set `CLAUDE_CODE_BS_AS_CTRL_BACKSPACE=0`.

### Newline in the prompt

| Method | Where |
|---|---|
| `\` then `Enter` | every terminal |
| `Ctrl+J` | every terminal, no setup |
| `Shift+Enter` | works without setup in iTerm2, WezTerm, Ghostty, Kitty, Warp, Apple Terminal, Windows Terminal (and Alacritty 0.16+, foot, v2.1.269+); VS Code, Cursor, Zed, Alacritty < 0.16: run `/terminal-setup`; not available in gnome-terminal and JetBrains terminals |
| `Option+Enter` | macOS with Option as Meta |

### Input prefixes

| Prefix | Meaning | Where |
|---|---|---|
| `/` at start | Command or skill | both |
| `@` | File or folder mention with autocomplete (use forward slashes on Windows too) | both |
| `!` at start | Shell mode: run a command, add its output to the session | T |
| `?` on empty input | Shortcut help panel | T |
| `:name:` | Emoji shortcode (v2.1.217+) | T |

### VS Code panel (V)

| Action | macOS | Windows / Linux |
|---|---|---|
| Toggle focus between editor and Claude | `Cmd+Esc` | `Ctrl+Esc` |
| New conversation as an editor tab | `Cmd+Shift+Esc` | `Ctrl+Shift+Esc` |
| New conversation (needs `enableNewConversationShortcut`) | `Cmd+N` | `Ctrl+N` |
| Reopen closed Claude session tab | `Cmd+Shift+T` | `Ctrl+Shift+T` |
| Insert `@file#5-10` for the selection (editor focused) | `Option+K` | `Alt+K` |
| Toggle focus view (hide tool activity) | `Ctrl+Option+F` | `Ctrl+Alt+F` |
| Command Palette, type "Claude Code" | `Cmd+Shift+P` | `Ctrl+Shift+P` |
| Extensions view | `Cmd+Shift+X` | `Ctrl+Shift+X` |
| Settings | `Cmd+,` | `Ctrl+,` |
| Integrated terminal (run `claude` there) | ``Cmd+` `` | ``Ctrl+` `` |
| Newline in the panel | `Shift+Enter` | `Shift+Enter` |
| Expand or collapse all thinking blocks | `Ctrl+O` | `Ctrl+O` |

Also: mode indicator under the prompt switches permission mode; click the model name for the picker; hover a message for the rewind button (fork conversation, rewind code, or both); status bar item "✻ Claude Code" opens the panel. On macOS Tahoe and later, `Cmd+Esc` can be intercepted by the system Game Overlay; free it in System Settings or rebind "Claude Code: Focus input".

VS Code extension commands (Command Palette): Focus Input, Open in Side Bar, Open in Terminal, Open in New Tab, Open in New Window, New Conversation, Reopen Closed Session, Insert @-Mention Reference, Toggle Focus view, Rename Session Tab, Show Logs, Logout. Useful extension settings: `claudeCode.useTerminal` (CLI-style panel), `claudeCode.initialPermissionMode`, `useCtrlEnterToSend`, `enableNewConversationShortcut`, `claudeCode.claudeProcessWrapper`.

---

## 5. Permission Modes

| Mode | Runs without asking | Use for |
|---|---|---|
| `default` (shown as **Manual**) | Reads only | Review every action, sensitive work |
| `acceptEdits` | Reads, file edits, common filesystem commands (`mkdir`, `touch`, `mv`, `cp`) | Iterating on code you review |
| `plan` | Reads (and classifier-approved commands where auto is available); no edits until you approve a plan | Explore before changing |
| `auto` | Everything, with a classifier reviewing actions and background safety checks; your explicit ask rules still prompt | Long tasks, fewer prompts |
| `dontAsk` | Reads and pre-approved tools; anything that would prompt is denied | Locked-down CI and scripts |
| `bypassPermissions` | Everything | Isolated containers and VMs only |

- **Starting mode:** with Claude Code v2.1.283+, `auto` is the built-in starting mode for interactive terminal and VS Code sessions (earlier versions: only Pro, Max, Team). Organizations can turn it off (for example the HIPAA configuration starts in Manual). `claude -p` starts in `default` in most sessions.
- **Switch:** `Shift+Tab` (T) · mode indicator (V) · `/plan` · `--permission-mode <mode>` · `permissions.defaultMode` in settings.
- **Cycle order:** `default` → `acceptEdits` → `plan` → (`bypassPermissions` when available) → `auto`; from `auto` the first press returns to `default`.
- **Settings restriction:** `auto` and `bypassPermissions` set as `defaultMode` in `.claude/settings.json` or `.claude/settings.local.json` do not take effect; put `auto` in `~/.claude/settings.json`.
- **Sandbox is separate from modes:** Manual mode plus `/sandbox` auto-allow gives fewer prompts without a classifier.

---

## 6. Important File Locations

`~/.claude` is `%USERPROFILE%\.claude` on Windows. If you set `CLAUDE_CONFIG_DIR` (environment only, not in settings), every `~/.claude` path lives under that directory instead. Project paths are relative to the repository root (use `\` or `/` on Windows).

### Project (in the repository)

| Path | Purpose | Commit? |
|---|---|---|
| `CLAUDE.md` or `.claude/CLAUDE.md` | Team instructions loaded every session | yes |
| `CLAUDE.local.md` | Personal project notes (create by hand; add to `.gitignore`) | no |
| `AGENTS.md` (root, `.claude/`, any directory) | Read when no CLAUDE.md exists (v2.1.277+); or `@AGENTS.md` import from CLAUDE.md. On Windows prefer the import: symlinks need Developer Mode or admin | yes |
| `.claude/settings.json` | Team permissions, hooks, plugins, env | yes |
| `.claude/settings.local.json` | Personal overrides; "don't ask again" approvals are saved here | no |
| `.claude/rules/*.md` | Topic rules; `paths:` frontmatter scopes them | yes |
| `.claude/skills/<name>/SKILL.md` | Skills (+ supporting files) | yes |
| `.claude/commands/<name>.md` | Single-file commands, same mechanism as skills | yes |
| `.claude/agents/<name>.md` | Subagents | yes |
| `.claude/agent-memory/<name>/` | Persistent subagent memory | yes |
| `.claude/output-styles/*.md` | Output styles | yes |
| `.claude/workflows/*.js` | Saved dynamic workflow scripts (each becomes a `/<name>` command) | yes |
| `.claude/worktrees/` | Worktrees created by `-w` | no |
| `.mcp.json` | Project-scoped MCP servers | yes |
| `.worktreeinclude` | Gitignored files to copy into new worktrees | yes |
| `<subdir>/CLAUDE.md` | Loaded on demand when Claude reads files there | yes |
| any path, e.g. `.claude/hooks/*.sh` | Hook scripts, referenced as `${CLAUDE_PROJECT_DIR}/...` | yes |

Project `.claude/settings.json` and hooks load from the directory you start in only, with no parent-directory fallback. CLAUDE.md files in parent directories are loaded.

### User, application data and install

| Item | macOS / Linux / WSL | Windows (native) |
|---|---|---|
| Personal instructions | `~/.claude/CLAUDE.md` | `%USERPROFILE%\.claude\CLAUDE.md` |
| Personal settings | `~/.claude/settings.json` | `%USERPROFILE%\.claude\settings.json` |
| Personal rules, skills, commands, agents, output styles, workflows | `~/.claude/rules/` `skills/` `commands/` `agents/` `output-styles/` `workflows/` | `%USERPROFILE%\.claude\rules\` `skills\` `commands\` `agents\` `output-styles\` `workflows\` |
| Keybindings | `~/.claude/keybindings.json` (`/keybindings`) | `%USERPROFILE%\.claude\keybindings.json` |
| Custom themes | `~/.claude/themes/*.json` | `%USERPROFILE%\.claude\themes\*.json` |
| Installed plugins | `~/.claude/plugins/` | `%USERPROFILE%\.claude\plugins\` |
| App state (sign-in, user-scope MCP, trust, UI toggles) | `~/.claude.json` (backups in `~/.claude/backups/`) | `%USERPROFILE%\.claude.json` |
| Session transcripts | `~/.claude/projects/<project>/<session>.jsonl` | `%USERPROFILE%\.claude\projects\<project>\<session>.jsonl` |
| Auto memory | `~/.claude/projects/<project>/memory/MEMORY.md` + topic files | `%USERPROFILE%\.claude\projects\<project>\memory\MEMORY.md` |
| Checkpoint snapshots | `~/.claude/file-history/<session>/` | `%USERPROFILE%\.claude\file-history\<session>\` |
| Plan-mode plans | `~/.claude/plans/` | `%USERPROFILE%\.claude\plans\` |
| Debug logs | `~/.claude/debug/` (on with `--debug` or `/debug`) | `%USERPROFILE%\.claude\debug\` |
| Large pastes cache | `~/.claude/paste-cache/` | `%USERPROFILE%\.claude\paste-cache\` |
| Session scratchpad | macOS `/private/tmp/claude-<uid>/<project>/<session-id>/scratchpad/` · Linux `/tmp/claude-<uid>/...` (or under `$TMPDIR`) | `%TEMP%\claude\<project>\<session-id>\scratchpad\` |
| Temp tree override | `CLAUDE_CODE_TMPDIR` | `CLAUDE_CODE_TMPDIR` |
| Launcher and versions | `~/.local/bin/claude`, `~/.local/share/claude/versions/` | `%USERPROFILE%\.local\bin\claude.exe`, `%USERPROFILE%\.local\share\claude` |

Transcripts, file history and caches are plaintext and cleaned up after `cleanupPeriodDays` (default 30, minimum 1); auto memory is not swept. `claude purge` removes a project's local state. `<project>` is the working-directory path with every character other than letters and digits replaced by `-`.

### Managed (organization)

| Item | macOS | Linux / WSL | Windows (native) |
|---|---|---|---|
| Directory | `/Library/Application Support/ClaudeCode/` | `/etc/claude-code/` | `C:\Program Files\ClaudeCode\` |
| Settings file | `managed-settings.json` | `managed-settings.json` | `managed-settings.json` |
| Drop-ins | `managed-settings.d/*.json` | `managed-settings.d/*.json` | `managed-settings.d\*.json` |
| Managed MCP | `managed-mcp.json` | `managed-mcp.json` | `managed-mcp.json` |
| Managed CLAUDE.md | `CLAUDE.md` in that directory | `CLAUDE.md` in that directory | `CLAUDE.md` in that directory |
| MDM / OS policy | configuration profile, domain `com.anthropic.claudecode` | n/a | registry value `Settings` (REG_SZ) under `HKLM\SOFTWARE\Policies\ClaudeCode` |
| User-writable fallback | n/a | n/a | same `Settings` value under `HKCU\SOFTWARE\Policies\ClaudeCode` (used only when no admin source is present) |
| Server-managed | claude.ai admin console (all platforms) | | |

None of these files or directories exist after installing Claude Code: an administrator creates and deploys them (MDM, Group Policy, Ansible). A missing `/etc/claude-code/CLAUDE.md` or `managed-settings.json` is normal and simply skipped. To try one on a lab machine: `sudo mkdir -p /etc/claude-code`, then write the file with `sudo tee`.

The legacy Windows path `C:\ProgramData\ClaudeCode\managed-settings.json` is not read. Precedence among managed sources: server-managed, then MDM/OS policy, then managed files (`managed-settings.d` merged with `managed-settings.json`), then HKCU. On WSL, `wslInheritsWindowsSettings` lets the Windows policy chain apply.

### CLAUDE.md load order (broad to specific, all concatenated)

1. Managed `CLAUDE.md`
2. `~/.claude/CLAUDE.md`
3. Ancestor directories, from the filesystem root down to the working directory (at launch)
4. Working directory `CLAUDE.md`, then `CLAUDE.local.md`
5. Subdirectory `CLAUDE.md` (on demand, when Claude reads a file there)

Target under 200 lines per file; files over 4 MiB are skipped. `@path` imports resolve relative to the importing file, up to four hops deep. Block-level HTML comments are stripped before injection. `/context` lists loaded files under **Memory files**.

---

## 7. Settings

### Scopes (highest wins; arrays merge, scalars from the higher layer win)

| # | Layer | macOS / Linux / WSL | Windows (native) |
|---|---|---|---|
| 1 | Managed | see section 6 (file, MDM, registry, server-managed) | see section 6 |
| 2 | Command line | `claude --settings <file\|json>` | same |
| 3 | Project local | `.claude/settings.local.json` | `.claude\settings.local.json` |
| 4 | Project shared | `.claude/settings.json` | `.claude\settings.json` |
| 5 | User | `~/.claude/settings.json` | `%USERPROFILE%\.claude\settings.json` |

Strict JSON: no comments, no trailing commas. `/status` shows setting sources; `claude doctor` shows validation errors. Edits apply to a running session without restart (a few keys are read once at startup). Project `allow` rules and marketplaces wait for folder trust; `deny` and `ask` apply immediately. A managed value can be overridden only by a few security keys where the stricter value wins.

### Common keys

| Key | Purpose |
|---|---|
| `permissions.allow` / `ask` / `deny` | Rules, evaluated deny, then ask, then allow; first match wins |
| `permissions.defaultMode` | Starting permission mode |
| `permissions.additionalDirectories` | Extra accessible directories |
| `permissions.disableBypassPermissionsMode` | `"disable"` removes bypass mode (managed) |
| `hooks` | Event → matcher → handler |
| `disableAllHooks` | Turn off hooks, custom status line and `@` suggestion command |
| `env` | Environment variables for every session and its subprocesses |
| `model` · `effortLevel` · `modelSettings` | Default model · default effort (`low` `medium` `high` `xhigh`) · per-model effort or `autoCompactWindow` |
| `fastMode` · `fastModePerSessionOptIn` | Fast mode default · reset each session |
| `sandbox` | OS isolation of Bash: `enabled`, `filesystem`, `network` (macOS, Linux, WSL 2) |
| `autoMode` | Allow and deny rules for the auto-mode classifier |
| `enabledPlugins` / `extraKnownMarketplaces` | Plugins and marketplaces for the team |
| `claudeMdExcludes` | Skip CLAUDE.md files by absolute-path glob (monorepos; managed CLAUDE.md cannot be excluded) |
| `claudeMd` | Managed-only inline CLAUDE.md content |
| `worktree.sparsePaths` | Sparse checkout for worktrees |
| `autoUpdatesChannel` · `minimumVersion` | `latest` or `stable` · update floor |
| `cleanupPeriodDays` | Transcript retention (default 30) |
| `allowManagedPermissionRulesOnly` · `allowManagedHooksOnly` | Only managed rules or hooks apply (managed) |
| `statusLine` · `outputStyle` · `editorMode` | UI customizations (`editorMode: "vim"`) |
| `preferredNotifChannel` | `"terminal_bell"` for a bell in terminals without desktop notifications |

### Permission rule syntax

| Rule | Matches |
|---|---|
| `Bash` / `Read` / `WebFetch` | The whole tool (as a deny rule, a bare tool name removes the tool) |
| `Bash(npm run build)` | That exact command |
| `Bash(npm run *)` | `npm run build`, `npm run test --watch`, `npm run` (not `npm install`) |
| `Bash(git push *)` | `git push origin main` (not `git -C . push`) |
| `Read(./.env)` | That file |
| `Read(./secrets/**)` | Everything under the folder |
| `Read(.env)` / `Read(**/.env)` | Any `.env` at or under the current directory |
| `Edit(/src/**)` | `/` anchors at the settings source: the project root in project settings, `~/.claude` in user settings |
| `Read(//Users/alice/secrets/**)` | `//` = absolute path from the filesystem root |
| `Read(~/.zshrc)` | Home-relative path |
| `Read(//c/**/.env)` | Windows: paths normalize to POSIX form (`C:\Users\alice` becomes `/c/Users/alice`); `//**/.env` matches across all drives |
| `WebFetch(domain:example.com)` | That domain |
| `Agent(model:opus)` · `Bash(run_in_background:true)` | Subagent or Bash calls with that parameter |

Read and Edit rules use gitignore pattern syntax; a Read deny also blocks Edit and Write on that path. Bash rules match the command as written and are not a security boundary (`/usr/bin/curl` and `sh -c '...'` bypass `Bash(curl *)`): combine deny rules with the sandbox and hooks. On Windows, commands touching UNC paths (`\\server\share`) always prompt.

Where "don't ask again" saves: Bash and WebFetch approvals go to `.claude/settings.local.json` at the git repository root (resolved through worktrees); file-edit approvals last until the session ends.

---

## 8. Hooks

| Item | Values |
|---|---|
| Handler types | `command` (shell), `http`, `mcp_tool`, `prompt` (single-turn LLM check), `agent` (subagent, experimental) |
| Matcher | Tool name or list (`Bash`, `Edit\|Write`); other characters are a JavaScript regex (`mcp__.*`); `*`, empty or omitted matches all |
| `if` (tool events) | Permission-rule syntax filter, e.g. `"Bash(git *)"` |
| Input | JSON on stdin (`session_id`, `cwd`, `permission_mode`, `tool_input`, `hook_event_name`, ...) |
| Exit `0` | Proceed (stdout JSON is honored) |
| Exit `2` | Block; stderr is fed back (on events that can block, such as PreToolUse, UserPromptSubmit, Stop, PermissionRequest) |
| Other exit codes | Non-blocking error |
| JSON output | `decision`, `reason`, `additionalContext`, `hookSpecificOutput.permissionDecision` (`allow` `deny` `ask` `defer`), `updatedInput` |
| Default timeout | 600 s command, http and mcp_tool; 30 s prompt; 60 s agent |
| Path variables | `${CLAUDE_PROJECT_DIR}`, `${CLAUDE_PLUGIN_ROOT}`, `${CLAUDE_PLUGIN_DATA}` |
| Browse | `/hooks` (view) |
| Disable | `"disableAllHooks": true` (only managed settings can disable managed hooks) |

Events (32 at the time of checking; the `/hooks` menu is authoritative):

| Group | Events |
|---|---|
| Session | `SessionStart` `Setup` `SessionEnd` `InstructionsLoaded` `ConfigChange` `CwdChanged` `DirectoryAdded` `FileChanged` |
| Prompt | `UserPromptSubmit` `UserPromptExpansion` |
| Tools | `PreToolUse` `PermissionRequest` `PermissionDenied` `PostToolUse` `PostToolUseFailure` `PostToolBatch` |
| Turn | `Stop` `StopFailure` `Notification` `MessageDisplay` |
| Agents and tasks | `SubagentStart` `SubagentStop` `TaskCreated` `TaskCompleted` `TeammateIdle` |
| Context and model | `PreCompact` `PostCompact` `PreModelSwitch` `PostModelSwitch` |
| Worktrees | `WorktreeCreate` `WorktreeRemove` |
| MCP | `Elicitation` `ElicitationResult` |

Windows: a command hook with `"shell": "powershell"` runs in PowerShell; otherwise the shell form uses Git Bash when installed. For a `.ps1` use exec form: `"command": "powershell.exe", "args": ["-NoProfile", "-ExecutionPolicy", "Bypass", "-File", "${CLAUDE_PROJECT_DIR}/.claude/hooks/script.ps1"]`. `.cmd` and `.bat` shims (npm, eslint) are not supported in exec form: spawn `node` on the underlying script.

---

## 9. Environment Variables

Set persistently in the `env` block of `settings.json`, or per shell: macOS/Linux `VAR=1 claude` · PowerShell `$env:VAR = "1"; claude` · CMD `set VAR=1`.

| Variable | Effect |
|---|---|
| `ANTHROPIC_API_KEY` | API key auth (interactive sessions prompt once to approve it) |
| `ANTHROPIC_MODEL` | Session model (overrides the `model` setting) |
| `ANTHROPIC_BASE_URL` | Route requests through a proxy or gateway |
| `CLAUDE_CODE_EFFORT_LEVEL` | Effort level (highest precedence over `--effort` and `/effort`) |
| `CLAUDE_CONFIG_DIR` | Relocate the `~/.claude` directory (environment only) |
| `CLAUDE_CODE_NEW_INIT=1` | Interactive multi-phase `/init` |
| `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1` | Load CLAUDE.md from `--add-dir` directories |
| `DISABLE_AUTOUPDATER=1` | Stop background update checks (`claude update` still works) |
| `DISABLE_UPDATES=1` | Block all update paths |
| `CLAUDE_CODE_SIMPLE=1` | Same as `--bare` |
| `CLAUDE_CODE_SAFE_MODE=1` | Same as `--safe-mode` |
| `MCP_TIMEOUT` | MCP startup timeout (default 30 s) |
| `MAX_THINKING_TOKENS` | Thinking budget on fixed-budget models; `0` turns thinking off except on Opus 5.5, Sonnet 5.5 and Fable (always on) |
| `CLAUDE_CODE_AUTO_COMPACT_WINDOW` | Auto-compact threshold in tokens |
| `BASH_DEFAULT_TIMEOUT_MS` · `API_TIMEOUT_MS` | Bash command timeout (default 120000) · API request timeout (default 600000) |
| `DISABLE_TELEMETRY` · `DO_NOT_TRACK` | Opt out of telemetry |
| `CLAUDE_CODE_DISABLE_FAST_MODE=1` | Disable fast mode entirely |
| `CLAUDE_CODE_NO_FLICKER=1` | Fullscreen renderer (or `/tui fullscreen`) |
| `CLAUDE_CODE_DEBUG_LOGS_DIR` | Debug log file path (despite the name, a file) |
| `CLAUDE_CODE_TMPDIR` | Move Claude Code's temp tree (scratchpad, images) |
| `CLAUDE_CODE_SHELL` | Shell for the Bash tool (`bash` or `zsh` path) |
| `CLAUDE_CODE_GIT_BASH_PATH` | Git Bash path on Windows |
| `CLAUDE_CODE_USE_POWERSHELL_TOOL` | `1` or `0`: PowerShell tool on third-party providers or off |
| `CLAUDE_CODE_BS_AS_CTRL_BACKSPACE` | Windows: `0` makes Backspace erase one character |
| `USE_BUILTIN_RIPGREP=0` | Use system ripgrep (Alpine) |
| `CLAUDE_CODE_PACKAGE_MANAGER_AUTO_UPDATE=1` | Let Claude Code run the Homebrew or WinGet upgrade |

---

## 10. Terminology

| Term | Meaning |
|---|---|
| Agentic loop | Gather context → act → verify → repeat until done |
| Agentic harness | The tools, context management, permissions and loop around the model; Claude Code is the harness, Claude is the model |
| Tool | An action Claude can take (read, edit, run a command, search the web, spawn a subagent) |
| Turn | One complete response, from your message until Claude finishes (hooks `Stop` fire here) |
| Session | A conversation with its own context window, stored under `~/.claude/projects/` |
| Context window | Working memory: conversation, files read, outputs, CLAUDE.md, auto memory, skills |
| Compaction | Automatic or `/compact` summary when context fills; project-root CLAUDE.md and auto memory reload from disk afterward |
| Checkpoint | Restore point per prompt; covers file-tool edits only, not Bash changes, not git |
| CLAUDE.md | Instructions you write; loaded every session as context after the system prompt; guidance, not enforcement |
| AGENTS.md | Instruction file for coding agents; read directly or imported by CLAUDE.md |
| Auto memory | Notes Claude writes itself per repository; first 200 lines or 25 KB of `MEMORY.md` load each session |
| Rules | `.claude/rules/*.md`; `paths:` frontmatter makes them path-scoped |
| Import | `@path/to/file` inside CLAUDE.md; max four hops |
| Settings layers | managed > command line > local > project > user |
| Managed settings | Org-enforced via file, MDM, registry or admin console; users and projects cannot override |
| Project trust | Folder acceptance before project config loads; home directory trust lasts one session |
| Permission mode | Baseline approval behavior: Manual (`default`), `acceptEdits`, `plan`, `auto`, `dontAsk`, `bypassPermissions` |
| Permission rule | `allow` / `ask` / `deny` entry for a tool and pattern, evaluated deny → ask → allow |
| Plan mode | Claude researches and proposes; no edits until you approve |
| Auto mode | A separate classifier reviews actions instead of you; default start mode from v2.1.283 for interactive sessions |
| Sandboxing | OS-level file and network isolation of shell commands; separate from permission rules |
| Hook | Deterministic handler at a lifecycle event; three levels: event, matcher, handler |
| Skill | `SKILL.md` playbook; loads when relevant or via `/name` |
| Bundled skills | Prompt playbooks shipped with Claude Code (`/batch`, `/code-review`, `/debug`, `/loop`, ...) |
| Command | `/name` instruction; now unified with skills |
| Frontmatter | YAML block between `---` lines at the top of skill, subagent, output-style and rule files |
| Subagent | Specialist with its own context window, tools and prompt; returns a summary (built in: Explore, Plan, general-purpose) |
| Agent teams | Several coordinated sessions with a shared task list (experimental, off by default) |
| Agent view / background session | `claude agents`, `--bg`: parallel sessions you monitor and attach to |
| MCP, MCP server | Model Context Protocol; a program that gives Claude tools, prompts or resources |
| Connector | An MCP server added to your claude.ai account; appears in `/mcp` |
| MCP Tool Search | Defers MCP tool schemas until needed |
| Channel | MCP server that pushes events into a running session (research preview) |
| Plugin / marketplace | Installable bundle of skills, hooks, subagents and MCP servers / catalog of plugins |
| Worktree isolation | Separate git worktree per session or subagent under `.claude/worktrees/` |
| Cloud session | Session on Anthropic-managed or self-hosted cloud infrastructure; `--cloud`, claude.ai/code |
| Remote Control | Continue a local session from phone or browser; execution stays on your machine |
| Teleport | `/teleport` (`/tp`): pull a cloud session into the local terminal |
| Dispatch | Phone-initiated task that starts a session in the Desktop app |
| Artifact | Live web page Claude publishes from a session to a private claude.ai URL |
| Non-interactive mode | `claude -p`; formerly "headless" |
| Bare mode | `--bare`; skips auto-discovery of hooks, skills, plugins, MCP, memory, CLAUDE.md |
| Output style | Changes Claude's instructions for tone and format; unlike CLAUDE.md it can replace the default coding instructions |
| System prompt | Sent before the conversation; CLAUDE.md and output styles arrive as system reminders, not in it |
| System reminder | Harness-inserted context (CLAUDE.md, hook output, skill list, attribution lines) |
| Surface | CLI, VS Code, JetBrains, Desktop, claude.ai: same engine |
| Prompt injection | Hostile instructions hidden in files, web pages or tool results |
| Verification loop | A check Claude can run (tests, build, screenshot) so "done" is provable; prerequisite for `/goal` |
| Effort level | Adaptive reasoning depth: `low` `medium` `high` `xhigh` `max` (model-dependent) |
| Ultracode | An effort setting that turns on dynamic workflows; not a level |
| Extended thinking | Visible reasoning before the answer; `ultrathink` in a prompt boosts one turn |
| Fast mode | Same Opus model on a faster, pricier API configuration (Opus 5.5, 5, 4.8; usage credits on subscriptions); `/fast` |

Renamed: slash commands → commands · custom commands → skills · headless → non-interactive · web session → cloud session.

### Course terms (not official product terms)

| Term | Meaning |
|---|---|
| Blast radius | Callers, tests and assumptions a change can reach |
| Verification target | Exact command plus expected output Claude must reproduce |
| Precise bug report | Reproduction, expected result, consistency, "suggest, then wait" |
| Dial | A named control for slide depth (Explanation depth, Jargon, Examples) |

---

## 11. Docs Map

Base URL `https://code.claude.com/docs/en/`. Index of every page: `https://code.claude.com/docs/llms.txt`.

| Topic | Page |
|---|---|
| Install, update, uninstall, Windows setup | `setup` |
| CLI commands and flags | `cli-reference` |
| Slash commands, bundled skills | `commands` |
| Shortcuts, input modes, vim mode | `interactive-mode` |
| Custom keybindings | `keybindings` |
| Terminal setup (newlines, Option key, tmux, Windows) | `terminal-config` |
| VS Code | `vs-code` |
| CLAUDE.md, rules, auto memory, AGENTS.md | `memory` |
| Monorepos | `large-codebases` |
| `.claude` directory and application data | `claude-directory` |
| Settings files and precedence | `settings`, `settings-reference` |
| Example settings | `settings-example` |
| Permissions and rule syntax | `permissions` |
| Permission modes, auto mode | `permission-modes` |
| Managed settings (file, MDM, registry) | `managed-settings`, `server-managed-settings` |
| Hooks | `hooks-guide`, `hooks` |
| Sandboxing | `sandboxing` |
| Models, effort, extended thinking | `model-config` |
| Fast mode | `fast-mode` |
| Environment variables | `env-vars` |
| Non-interactive and bare mode | `headless` |
| Agent view, background sessions | `agent-view` |
| Best practices | `best-practices` |
| Glossary | `glossary` |

---

## 12. Validation Notes

Checked line by line against the pages above. Differences from the first version of this file:

| Item | Before | Now |
|---|---|---|
| Hook events | 8 listed | Full list of 32 (`/hooks` is authoritative) |
| Hook handlers | "shell command, HTTP, MCP tool, LLM prompt, subagent" | Exact types `command`, `http`, `mcp_tool`, `prompt`, `agent`; matcher syntax, `if`, timeouts, exit codes |
| Auto mode | Listed as one of six modes | Also the built-in starting mode in interactive sessions from v2.1.283; cycle order documented |
| `/agents` | "behavior varies by version" | Prints a reminder since v2.1.198; interface in v2.1.197 and earlier |
| `/vim`, `/pr-comments`, `/ultraplan` | not mentioned | Marked removed |
| `ultracode` | shown as an `--effort` value only | Clarified: a mode that turns on workflows; `ultrathink` is the per-prompt keyword |
| Fast mode | "faster output setting" | Same Opus model, faster API configuration, higher price, Opus only |
| Windows paths | only the managed directory | Windows column for install, user, application data, scratchpad, managed files, registry and MDM |
| Windows shortcuts | not covered | `Alt+V`, `Alt+M`, `Alt+P/T/O`, Windows Terminal Shift+Enter, Backspace behavior |
| Windows install | PowerShell only | Also CMD and WinGet; Git for Windows is optional |
| Glossary | "Harness" | "Agentic harness"; added AGENTS.md, bundled skills, channel, cloud session, Remote Control, Dispatch, artifact, agent view, ultracode |
| CLI | subset | Added `install`, `agents`, `attach`, `logs`, `stop`, `respawn`, `rm`, `--bg`, `--cloud`, `--desktop`, `--restricted`, `--agent`, `--from-pr` and others |

Not stated verbatim in the docs (kept because they are standard behavior, but check before class): `code --install-extension anthropic.claude-code` (the docs show the Extensions view and a `vscode:extension/anthropic.claude-code` link), and WSL using the Linux paths inside the distribution (implied by "install and launch inside the WSL terminal"). The docs do not give a `$schema` URL for `settings.json`.
