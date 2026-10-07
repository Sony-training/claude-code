# Claude Code Day 1 Lab Guide

Modules 1 to 5. Part A to D are the foundations; Part E holds the module labs.

| Part | Content |
|---|---|
| A | Install CLI and VS Code extension, lab sources |
| B | CLAUDE.md: content, standards, compliance, monorepo |
| C | settings.json: location, guardrails, examples |
| D | Prompt structure and keywords |
| E | Labs, Modules 1 to 5 |

Companion file: `Claude_Code_Reference.md` (shortcuts, commands, file locations, terminology).

---

## Part A: Setup

### A1. Prerequisites

| Item | Requirement |
|---|---|
| OS | macOS 13+, Ubuntu 20.04+, Debian 10+, Windows 10 1809+ (WSL 2 for Linux toolchains and sandboxing) |
| RAM | 4 GB+ |
| Account | Pro, Max, Team, Enterprise or Console |
| Network | HTTPS to claude.ai, git hosts, package mirrors |
| VS Code | 1.94 or later (extension only) |

### A2. Install the CLI

```bash
# macOS, Linux, WSL (recommended)
curl -fsSL https://claude.ai/install.sh | bash

# alternative: npm (Node.js 22+), never with sudo
npm install -g @anthropic-ai/claude-code
```

Windows PowerShell: `irm https://claude.ai/install.ps1 | iex`

Open a new terminal, then:

```bash
claude --version
claude doctor
```

Expected: a version number; `claude doctor` reports install health with no settings errors.

If `claude: command not found`: the install directory is not on PATH (native installer uses `~/.local/bin`).

### A3. First run and login

```bash
cd ~/work/busybox        # any git repository
claude
```

1. Follow the browser prompt to sign in
2. Accept the folder trust dialog
3. Run `/status` (account, model, settings sources)
4. Run `/help`

CI or scripts: `claude setup-token`, then set the token as a secret.

### A4. Install the VS Code extension

| Step | Action |
|---|---|
| 1 | `code --install-extension anthropic.claude-code`, or Extensions view (`Ctrl+Shift+X`) → search "Claude Code" → Install |
| 2 | If it does not appear: Command Palette → `Developer: Reload Window` |
| 3 | Open the project folder in VS Code |
| 4 | Click the Spark icon in the editor toolbar (or the status bar item "Claude Code") |
| 5 | Sign in when prompted |
| 6 | Run `/context` in the panel to see what loaded |

Verify:

- The panel answers a prompt that uses `@coreutils/cat.c`
- Mode indicator under the prompt shows the permission mode
- `claude --resume` in a terminal lists the panel conversation

Use the CLI inside VS Code: open the integrated terminal (``Ctrl+` ``), run `claude`. The extension bundles its own CLI for the panel but does not put `claude` on PATH, so the standalone CLI install (A2) is still required for this.

Shared by panel and terminal: CLAUDE.md, `~/.claude/settings.json`, `.claude/settings.json`, hooks, MCP servers, plugins, conversation history.

### A5. Lab sources and tools

```bash
sudo apt install -y build-essential git jq meson ninja-build qemu-user \
  gcc-arm-linux-gnueabihf gstreamer1.0-tools \
  libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev python3

# BusyBox (full history, needed for Day 2 patch work)
git clone https://git.busybox.net/busybox ~/work/busybox
cd ~/work/busybox
git tag | grep -x 1_35_0          # tag name check; if empty, use the tarball fallback below
git checkout 1_35_0 && git checkout -b lab
make defconfig

# GStreamer monorepo (read-only reference and element generator tools)
git clone --depth 1 --branch 1.24.9 https://gitlab.freedesktop.org/gstreamer/gstreamer.git ~/work/gstreamer

# Cross toolchain: distro package above, or your custom build (crosstool-ng or vendor SDK)
export CROSS=arm-linux-gnueabihf-
export SYSROOT=/usr/arm-linux-gnueabihf        # custom toolchain: <prefix>/<tuple>/sysroot
export PATH="$TOOLCHAIN_DIR/bin:$PATH"          # only for a custom toolchain; set TOOLCHAIN_DIR first
${CROSS}gcc --version && gcc --version
```

Fallback if the tag is missing: `https://busybox.net/downloads/busybox-1.35.0.tar.bz2` (release dated 2021-12-26), then `git init` and commit.

| Asset | Used in |
|---|---|
| BusyBox 1.35.0 (C, Kconfig, applets) | M1 to M5 |
| GStreamer element `rot13filter` (C, meson) | M2, M4, M5 |
| GStreamer 1.24.9 source (reference, tools) | M2, M3 |
| Native gcc and ARM cross toolchain | M2 to M5 |
| Shell scripts `scripts/`, Python `tools/` | M2, M4 |

---

## Part B: CLAUDE.md

### B1. What it is

| Fact | Detail |
|---|---|
| Purpose | Persistent instructions loaded at the start of every session |
| Nature | Context, not enforcement. Claude can drift; hooks and settings enforce |
| Verify loaded | `/context` → list under **Memory files** |
| Generate | `/init` (suggests improvements if one exists) |
| Edit | `/memory` or any editor |
| Audit | `/doctor prompt-audit` (stale, conflicting or missing-path instructions) |

### B2. Locations and load order

Broad to specific; all files are concatenated, none overrides another.

| Scope | Location | Shared |
|---|---|---|
| Managed policy | macOS `/Library/Application Support/ClaudeCode/CLAUDE.md` · Linux, WSL `/etc/claude-code/CLAUDE.md` · Windows `C:\Program Files\ClaudeCode\CLAUDE.md` | whole organization |
| User | `~/.claude/CLAUDE.md` | you, all projects |
| Project | `./CLAUDE.md` or `./.claude/CLAUDE.md` | team (commit) |
| Local | `./CLAUDE.local.md` (gitignore it) | you, this project |
| Ancestors | `CLAUDE.md` in every parent directory of the working directory | loaded at launch |
| Subdirectories | `<subdir>/CLAUDE.md` | loaded on demand when Claude reads a file there |

Also: `AGENTS.md` is read when no CLAUDE.md exists; import it from CLAUDE.md to use both.

The managed policy file does not exist after installing Claude Code. Nothing creates `/etc/claude-code/` (Linux, WSL), `/Library/Application Support/ClaudeCode/` (macOS) or `C:\Program Files\ClaudeCode\` (Windows): an administrator deploys the file (MDM, Group Policy, Ansible). If it is missing, nothing is wrong; Claude Code simply skips it. To try it on a lab machine:

```bash
sudo mkdir -p /etc/claude-code
printf '%s\n' '# Organization rules' '- Do not use strcpy, strcat, sprintf, gets' | sudo tee /etc/claude-code/CLAUDE.md
```

Restart `claude`, run `/context`, and confirm the file is listed under **Memory files**. Remove it after the lab: `sudo rm /etc/claude-code/CLAUDE.md`. Windows: create `C:\Program Files\ClaudeCode\CLAUDE.md` from an administrator PowerShell; macOS: `/Library/Application Support/ClaudeCode/CLAUDE.md` with `sudo`.

### B3. What goes in, what stays out

| Include | Exclude |
|---|---|
| Build, test, lint commands Claude cannot guess | Anything Claude learns by reading the code |
| Code style that differs from defaults | Standard language conventions |
| Test runner and how to run one test | Long API documentation (link instead) |
| Branch, commit, PR conventions | Information that changes often |
| Architecture decisions specific to the project | Tutorials, long explanations |
| Required environment, toolchain, env vars | File-by-file codebase descriptions |
| Gotchas and "never touch" paths | "Write clean code" |

### B4. Writing rules

| Rule | Example |
|---|---|
| Concrete and checkable | "Run `make check` before committing", not "test your changes" |
| Under 200 lines per file | Move the rest to rules or skills |
| Headers and bullets | Group under Commands, Conventions, Do not touch |
| One owner per rule | Two contradicting rules: Claude picks one arbitrarily |
| Emphasis sparingly | "IMPORTANT" on the one line Claude keeps skipping |
| Hidden maintainer notes | `<!-- note -->` comments are stripped before injection |
| Imports | `@docs/git-workflow.md`; relative to the file; max 4 hops; still loaded at launch, so no context saving |
| Compaction hint | "When compacting, always preserve the list of modified files and test commands" |
| Prune | Ask: would removing this line cause a mistake? If not, cut it |
| Review in PRs | Treat edits like code changes |
| Rule ignored anyway | Convert it to a hook |

### B5. Where standards, compliance and linting go

| Need | Mechanism | Why |
|---|---|---|
| Coding standard summary, naming, style | CLAUDE.md | Always in context; advisory |
| Standard that applies to some files only (C security rules, shell rules) | `.claude/rules/*.md` with `paths:` | Loads only when Claude reads matching files |
| Multi-step procedure (release checklist, CVE triage) | Skill: `.claude/skills/<name>/SKILL.md` | Loads on demand |
| Must happen every time (lint after edit, block protected files) | Hook in `settings.json` | Deterministic; model cannot skip it |
| Must never happen (read secrets, edit generated files, force push) | `permissions.deny` in settings; sandbox | Enforced by the client |
| Pass or fail gate for the team | CI pipeline (`-Wall -Wextra -Werror`, linters, `make check`) | Last line of defense, independent of Claude |
| Organization-wide standard and compliance reminders | Managed CLAUDE.md | Cannot be excluded by users |
| Organization-wide enforcement | Managed settings (`permissions.deny`, `sandbox`, `allowManaged...Only`) | Cannot be overridden by users |

Rule of thumb: CLAUDE.md says it, hooks and settings enforce it, CI proves it.

### B6. Example: project CLAUDE.md (BusyBox lab)

```markdown
# Project: BusyBox 1.35.0 lab (C, Kconfig, multi-call binary)

## Commands
- Configure: make defconfig
- Build native: make -j$(nproc)
- Build ARM: scripts/build.sh   (CROSS=arm-linux-gnueabihf-, output build-arm/)
- Test: <verified test command>   (record after running it once)
- Run one applet: ./busybox <applet> ...

## Conventions
- Applets live in coreutils/, miscutils/ etc.; shared helpers in libbb/
- A new applet needs the //config:, //kbuild:, //usage: blocks (copy from coreutils/cat.c)
- Shell scripts: bash, `set -euo pipefail`, shellcheck clean
- Python tools: Python 3, stdlib only, `python3 -m py_compile` clean

## Standards (illustrative org rules, replace with yours)
- Do not use strcpy, strcat, sprintf, gets; use bounded forms and check lengths
- Check every allocation and every return value
- No new gcc warnings in files you change
- New code needs a test in testsuite/

## Do not touch
- .config, include/autoconf.h, include/applet_tables.h (generated; use make)
- Anything under build-arm/ (output)

## Verification
- Show the command and its output as evidence before saying a task is done

## Compaction
- When compacting, preserve the list of modified files and the test command
```

### B7. Example: path-scoped rules

`.claude/rules/c-security.md`

```markdown
---
paths:
  - "**/*.c"
  - "**/*.h"
---
- Bounded string functions only; no strcpy, strcat, sprintf, gets
- Validate every length from input before using it as an index or size
- Check integer overflow on size arithmetic before allocation
- Every malloc/xmalloc result and every read/write return value is checked
```

`.claude/rules/shell.md`

```markdown
---
paths:
  - "**/*.sh"
  - "scripts/**"
---
- Start with `#!/usr/bin/env bash` and `set -euo pipefail`
- Quote all variable expansions
- No hard-coded absolute paths; use variables from CLAUDE.md
```

`.claude/rules/python.md`

```markdown
---
paths:
  - "tools/**/*.py"
---
- Python 3 stdlib only
- Functions have type hints; scripts return non-zero on failure
```

### B8. Monorepo and large repositories

Example layout used in this lab:

```text
firmware/
  CLAUDE.md                      # org-wide rules, commit conventions, verification
  .claude/
    settings.json                # deny rules, hooks (repo root)
    rules/c-security.md          # path-scoped standards
  busybox/
    CLAUDE.md                    # busybox build, applets, tests
    .claude/skills/
  gst-plugin/
    CLAUDE.md                    # meson build, GStreamer conventions
  toolchain/
    CLAUDE.md                    # toolchain build, sysroot
  ci/
    CLAUDE.md                    # pipeline rules
```

| Question | Answer |
|---|---|
| What goes in the root file? | Rules for every package: standards, commit format, "never edit generated files" |
| What goes in a package file? | That package's build, test, conventions only |
| How do package files load? | Started in the package directory: its file and every ancestor's load at launch. Started at the root: package files load on demand when Claude reads there |
| Which directory to start in? | Work in one package: start there. Work across packages: start at the root |
| Other teams' files are noisy | `claudeMdExcludes` in `.claude/settings.local.json`: `["**/packages/web/**"]` style globs; managed CLAUDE.md cannot be excluded |
| Standards scattered over many paths | One `.claude/rules/*.md` with `paths:` instead of many package files |
| Package-specific procedures | `.claude/skills/` inside the package |
| Vendored or generated code | `permissions.deny` `Read(./**/vendor/**/*)` (deny rules are not inherited from parent directories: put them in the settings file of the directory you start from) |
| Sibling package access | `permissions.additionalDirectories` or `--add-dir` (file access only; CLAUDE.md of added dirs needs `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1`) |
| Files go stale | Review CLAUDE.md in PRs; `/doctor prompt-audit`; revisit after model upgrades |
| Verify what loaded | `/context` → Memory files |
| Centralize | Plugin in an internal marketplace; managed CLAUDE.md for the org |

Project `.claude/settings.json` is not inherited from parent directories (CLAUDE.md is). Each directory you start from needs its own self-contained settings file.

---

## Part C: settings.json

### C1. Purpose

Technical configuration that the client enforces, as opposed to CLAUDE.md which guides the model.

| Controls | Key |
|---|---|
| What Claude may do without asking, must ask, never do | `permissions.allow` / `ask` / `deny` |
| Starting permission mode | `permissions.defaultMode` |
| Extra accessible directories | `permissions.additionalDirectories` |
| Deterministic actions at lifecycle events | `hooks` |
| Process isolation | `sandbox` |
| Environment variables | `env` |
| Default model, effort | `model`, `modelSettings` |
| Plugins and marketplaces | `enabledPlugins`, `extraKnownMarketplaces` |
| Skip CLAUDE.md files | `claudeMdExcludes` |
| Update channel | `autoUpdatesChannel` |

### C2. Locations and precedence

| # | Layer | File | Who | Commit? |
|---|---|---|---|---|
| 1 (highest) | Managed | macOS `/Library/Application Support/ClaudeCode/managed-settings.json` · Linux, WSL `/etc/claude-code/managed-settings.json` · Windows `C:\Program Files\ClaudeCode\managed-settings.json` (+ `managed-settings.d/`, MDM, admin console) | Organization | n/a |
| 2 | Command line | `claude --settings <file\|json>` | This session | n/a |
| 3 | Project local | `.claude/settings.local.json` | You, this project | No |
| 4 | Project shared | `.claude/settings.json` | Team | Yes |
| 5 (lowest) | User | `~/.claude/settings.json` | You, all projects | No |

| Behavior | Detail |
|---|---|
| Lists merge | `permissions.allow` from several files combine; scalars: higher layer wins |
| Deny wins | Evaluation order deny → ask → allow; first match decides; allow cannot carve out of deny |
| Managed file | Not created by the installer. Absent by default on Linux (`/etc/claude-code/managed-settings.json`), macOS and Windows; an admin deploys it. Absent is normal |
| Strict JSON | No comments, no trailing commas; a broken file shows as a Settings Error |
| Live reload | Edits apply without restart |
| Trust | Project `allow` rules, `additionalDirectories` and most `env` values wait until you trust the folder; `deny` and `ask` apply immediately |
| `defaultMode` | `auto` and `bypassPermissions` do not take effect from project or local files |
| Written by Claude | "Yes, don't ask again" saves an allow rule to `.claude/settings.local.json`; `/config` writes `~/.claude/settings.json` |
| `~/.claude.json` | Claude's own state (sign-in, user-scope MCP, trust). Do not hand-edit |
| Inspect | `/status` (setting sources), `/permissions`, `/hooks`, `claude doctor` |

### C3. Guardrail layers

| Layer | Stops | Does not stop |
|---|---|---|
| `permissions.deny` | Built-in file tools, recognized file commands (cat, head, grep, find) on denied paths | Subprocesses that open files themselves; `grep -r` over a directory containing denied files |
| `permissions.ask` | Silent execution of risky commands | Different spelling of the same command |
| Bash rules | The command as written (`git push *`) | `git -C . push`, `/usr/bin/curl`, `sh -c '...'` |
| Hooks (PreToolUse) | Any matching tool call, whatever the model decided | Anything outside the matcher (e.g. an `Edit\|Write` hook does not see a Bash `sed -i`) |
| Sandbox | File and network access of Bash commands at OS level | Native Windows (not supported); WSL 2, Linux, macOS supported |
| Managed settings | User and project overrides | n/a |
| CI | Anything that reaches the main branch | n/a |

Bash deny rules are not a security boundary. Combine deny rules, a hook and the sandbox, then confirm in CI.

### C4. Example: team `.claude/settings.json` (commit it)

```json
{
  "permissions": {
    "allow": [
      "Bash(make *)",
      "Bash(git status)",
      "Bash(git diff *)",
      "Bash(git log *)",
      "Bash(bash scripts/*)",
      "Bash(python3 tools/*)",
      "Bash(file *)"
    ],
    "ask": [
      "Bash(git commit *)",
      "Bash(git push *)"
    ],
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Edit(/.config)",
      "Edit(/include/autoconf.h)",
      "Edit(/include/applet_tables.h)",
      "Bash(rm -rf *)",
      "Bash(curl *)"
    ]
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/protect-generated.sh"
          }
        ]
      }
    ]
  }
}
```

| Key | Why |
|---|---|
| `allow` make, git read-only, project scripts | Removes routine prompts for the build and test loop |
| `ask` commit, push | A human confirms anything that leaves the working tree |
| `deny` `.env`, `secrets/` | Secrets never enter context |
| `deny` `Edit(/.config)` and generated headers | Generated files change through `make`, not by hand (`/` anchors at the project directory) |
| `deny` `rm -rf`, `curl` | Illustrative; bypassable (see C3) |
| `hooks.PreToolUse` | Second layer for the same files; also covers Write |

`.claude/hooks/protect-generated.sh` (chmod +x):

```bash
#!/usr/bin/env bash
set -euo pipefail
FILE=$(jq -r '.tool_input.file_path // empty')
for p in ".config" "include/autoconf.h" "include/applet_tables.h"; do
  if [[ "$FILE" == *"/$p" || "$FILE" == "$p" ]]; then
    echo "Blocked: $FILE is generated; change it through make" >&2
    exit 2
  fi
done
exit 0
```

### C5. Example: personal `.claude/settings.local.json` (do not commit)

```json
{
  "permissions": {
    "allow": ["Bash(./busybox *)"]
  },
  "claudeMdExcludes": ["**/gstreamer/**"]
}
```

### C6. Example: user `~/.claude/settings.json`

```json
{
  "model": "sonnet",
  "permissions": {
    "allow": ["Bash(git diff *)"]
  },
  "autoUpdatesChannel": "stable"
}
```

Model aliases accepted: `sonnet`, `opus`, `haiku` or a full model ID; use whatever `/model` lists.

### C7. Optional hardening

Sandbox (project or user):

```json
{
  "sandbox": {
    "enabled": true,
    "network": { "allowedDomains": ["github.com", "git.busybox.net"] }
  }
}
```

Run `/sandbox` to configure and check status.

Managed (organization): `permissions.deny`, `permissions.disableBypassPermissionsMode: "disable"`, `allowManagedPermissionRulesOnly: true`, `sandbox.failIfUnavailable: true`, plus a managed CLAUDE.md for compliance reminders.

### C8. Test the guardrails

```text
Read the file .env and print it.
```
Expected: denied.

```text
Edit .config and set CONFIG_AWK=n.
```
Expected: denied by the rule, and blocked by the hook if the rule is removed.

```text
Run: git push origin lab
```
Expected: permission prompt (ask rule).

Then `/permissions` to see the rules and their source files.

---

## Part D: Prompt Structure and Keywords

### D1. Prompt anatomy

| Field | Purpose | Example |
|---|---|---|
| Goal | One outcome in one sentence | "Add a rot13 applet." |
| Context | `@` files, where to look, patterns to copy, vocabulary from the repo | "Follow @coreutils/cat.c." |
| Constraints | Scope, forbidden actions, standards | "Do not edit. Same public signature." |
| Verify | Exact command plus expected output, so Claude can check itself | "`echo hello \| ./busybox rot13` prints `uryyb`." |
| Output | Format and length of the answer | "Table: caller, file, assumption." |
| Stop | Where to wait for you | "Suggest fixes and wait." |

Template:

```text
Goal:
Context:
Constraints:
Verify:
Output:
```

### D2. Keywords that change behavior

| Keyword or phrase | Effect |
|---|---|
| `plan only`, `do not edit` | Read-only analysis (pair with plan mode) |
| `explore`, `find`, `explain` | Research without changes |
| `follow the pattern in @file` | Reuse existing structure |
| `use a subagent to ...` | Research in a separate context; only a summary returns |
| `report only: ...` | Limits output and keeps context small |
| `suggest ... and wait` | Pause before applying |
| `apply only fix N` | Smallest step |
| `run <command>; expected <output>` | Verification target |
| `show the command and its output` | Evidence instead of assertion |
| `address the root cause, do not suppress the error` | Prevents symptom fixes |
| `write a failing test that reproduces it, then fix it` | Test-first debugging |
| `same public signature` | Compatibility constraint |
| `list callers and tests first` | Blast-radius check |
| `say "not found" if you cannot see it` | Reduces guessing |
| `IMPORTANT` | Emphasis for one rule; overuse dilutes it |
| `@path`, `@dir/` | Attach file or folder |
| `/effort` levels | Reasoning depth is a setting, not a prompt word |

### D3. Prompt pattern per module

| Module | Pattern | Keywords |
|---|---|---|
| 1 Fundamentals | Orient, trace, record | explore, cite file paths, quote the exact lines, `/init`, `/context` |
| 2 System software | Plan → apply → verify; standards as rules and hooks | plan only, follow the pattern, same signature, run and expect, `/hooks` |
| 3 Legacy code | Broad → narrow → specific; blast radius | overview, in the project's words, use a subagent, callers, tests, do not edit |
| 4 Testing and QA | Test-first; precise bug report; checkpoints | failing test, reproduction, expected, consistent or intermittent, `/rewind` |
| 5 Model and cost | Filter output; right-size model; manage context | report only, do not print the log, `/model`, `/effort`, `/compact`, `/usage` |

### D4. Anti-patterns

| Anti-pattern | Fix |
|---|---|
| "Fix the bug." | Symptom, location, expected, how to verify |
| Kitchen-sink session | `/clear` between unrelated tasks |
| Correcting the same thing three times | `/clear`, rewrite the prompt with what you learned |
| Unscoped "investigate" | Name the subsystem or use a subagent |
| Long CLAUDE.md | Prune; move to rules, skills, hooks |
| Trusting plausible output | Always give a check Claude can run |
| Reviewer prompted for "gaps" | Ask for findings with a concrete failing input, correctness only |

---

## Part E: Labs

Prompt format: fields from D1. `/command` lines are typed as-is. Placeholders in `< >` are yours to fill.

### Module 1: Fundamentals and Setup

Assets: BusyBox. Prerequisite: Part A done.
Keywords: explore, cite file paths, quote the exact lines, record only what you ran.

**1.1 Map the codebase** (plan mode: `Shift+Tab` until plan, or `claude --permission-mode plan`)

```text
Goal: Give me a map of this codebase.
Context: BusyBox 1.35.0, a C multi-call binary. Start from @Makefile and @coreutils/.
Constraints: read-only; do not run a build.
Output: top-level directories, one line each; how an applet is declared and configured; how the build is configured. Cite file paths.
```

**1.2 Trace one applet**

```text
Goal: Explain how the cat applet is wired in from source to the final binary.
Context: @coreutils/cat.c
Constraints: quote exact lines; write "not found" for anything you cannot see.
Output: the //applet:, //config:, //kbuild: lines it uses, and the name of its main function.
```

**1.3 Generate, then record verified commands**

```text
/init
```

```text
Goal: Find how this repo builds and runs its tests, and record only commands that worked.
Context: Makefile*, README*, testsuite/
Constraints: use the defconfig; run the build once, then run the test suite once.
Output: update CLAUDE.md; list each command with its exit code.
```

**1.4 Tighten CLAUDE.md**

```text
Goal: Rewrite CLAUDE.md as a short, checkable file.
Context: @CLAUDE.md and the structure in Part B6 (Commands, Conventions, Standards, Do not touch, Verification, Compaction).
Constraints: under 60 lines; delete anything derivable from the code; every rule must be verifiable.
Verify: run /context and confirm CLAUDE.md is listed under Memory files.
```

**1.5 Add guardrails**

Create `.claude/settings.json` and `.claude/hooks/protect-generated.sh` from C4, `chmod +x` the script, then:

```text
/permissions
```

```text
/status
```

```text
/hooks
```

Run the three tests in C8.

**1.6 Modes and interrupt**

```text
Goal: List every file you would touch to add a new applet.
Constraints: plan only; no edits.
Output: file list, one-line reason each.
```

```text
Goal: Run the native build and list the first 5 gcc warnings.
```

Press `Esc` while it builds.

**1.7 Same task in VS Code**

Open the folder in VS Code, open the Claude panel, run prompt 1.1, then `/context`. Switch mode with the mode indicator. In a terminal run `claude --resume` and pick the panel conversation.

Done when:

- `claude --version` works and `/status` shows your account
- CLAUDE.md is under 60 lines, lists a verified build and test command, and appears in `/context`
- Settings file loads (`/permissions` shows the deny rules); `.env` read and `.config` edit are refused
- Plan mode produced a file list with no changes (`git status` clean)
- `Esc` stopped a running turn
- The panel and the terminal share the conversation

### Module 2: System Software Development

Assets: BusyBox, native gcc, ARM cross toolchain, shell, GStreamer tools.
Keywords: plan only, follow the pattern in, same signature, run and expect, no new warnings.

**2.1 Plan a new applet**

```text
Goal: Add a new applet rot13 that reads files or stdin and rotates ASCII letters by 13.
Context: Follow the declaration, config and build blocks of @coreutils/cat.c.
Constraints: plan only; follow Standards in CLAUDE.md.
Output: numbered plan; each step names a file; include the config symbol and the test you would add.
```

Read the plan as a design document: files, signature, verification. Reject or edit before approving. `Ctrl+G` opens the plan in your editor (terminal); in VS Code the plan opens as a Markdown document you can comment on.

**2.2 Implement and verify**

```text
Goal: Implement the approved plan.
Constraints: follow CLAUDE.md; introduce no new gcc warnings in files you change.
Verify: build, then run `echo hello | ./busybox rot13`; expected output is exactly `uryyb`.
Output: show the command and its output.
```

**2.3 Review loop**

```text
Goal: Review the rot13 applet for buffer handling and error returns.
Constraints: do not edit. Report only issues with a concrete failing input.
Output: numbered findings, file and line.
```

```text
Goal: Apply only fix 1.
Verify: rebuild; rerun the echo test; expected `uryyb`.
```

**2.4 Cross-build script (shell, custom toolchain)**

```text
Goal: Write scripts/build.sh that builds BusyBox for ARM with the custom toolchain.
Context: toolchain prefix is $CROSS (see CLAUDE.md).
Constraints: bash; set -euo pipefail; print `${CROSS}gcc --version` first; run `make O=build-arm defconfig` then `make O=build-arm CROSS_COMPILE=$CROSS -j$(nproc)`; no hard-coded paths.
Verify: run it; `file build-arm/busybox` reports ELF 32-bit ARM.
```

**2.5 GStreamer element (C)**

```text
Goal: Create a GStreamer element rot13filter in C (GstBaseTransform) that rotates ASCII letters by 13 in place.
Context: generate the skeleton with gst-project-maker and gst-element-maker from ~/work/gstreamer/subprojects/gst-plugins-bad/tools/ (run --help first).
Constraints: ANY caps on both pads; meson build in builddir; no extra dependencies; follow the C rules in .claude/rules/c-security.md.
Verify: `meson setup builddir && ninja -C builddir` succeeds; `GST_PLUGIN_PATH=$(dirname $(find builddir -name '*.so' | head -1)) gst-inspect-1.0 rot13filter` lists the element and its pads.
```

**2.6 Standards as rules**

Create `.claude/rules/c-security.md`, `shell.md`, `python.md` from B7. Then:

```text
Goal: Add a helper that copies argv[1] into a 16-byte stack buffer and prints it.
Constraints: follow the rules loaded for C files.
Output: the code, and which rule from the project rules you applied.
```

Expected: bounded copy and a length check; the answer names the rule.

**2.7 Hook: protect generated files**

Already installed in 1.5. Run:

```text
Edit include/autoconf.h and add a comment on the first line.
```

Expected: blocked with "Blocked: ... is generated".

Done when:

- `echo hello | ./busybox rot13` prints `uryyb`
- `file build-arm/busybox` reports ARM
- `gst-inspect-1.0 rot13filter` lists the element
- The C rule file shaped the generated helper
- The generated-file edit is refused

### Module 3: Legacy and Large Codebases

Assets: BusyBox `libbb/`, GStreamer 1.24.9 source (read-only).
Keywords: overview, in the project's own words, use a subagent, summary, callers, tests, do not edit.

**3.1 Broad, narrower, specific**

```text
Goal: Give me an overview of libbb/: what it provides to applets.
Constraints: read-only. Output: grouped list, one line per group.
```

```text
Goal: Find the files that copy data between file descriptors and explain each helper.
Constraints: read-only; cite file and function names.
```

```text
Goal: Explain how bb_copyfd_eof handles read errors and write errors.
Context: libbb/copyfd.c
Constraints: quote the lines that decide the return value.
```

**3.2 Blast radius with a subagent**

```text
Goal: Find every caller of bb_copyfd_eof and every test that covers it.
Constraints: use a subagent; read-only.
Output: table of caller, file, assumption about the return value; list callers with no covering test.
```

```text
Goal: If bb_copyfd_eof changed its error return value, which callers would break?
Constraints: do not edit. Base the answer on the table above.
```

**3.3 Large codebase: GStreamer**

```text
Goal: Explain how the qtdemux element is registered and where sample-group (sbgp) boxes are parsed.
Context: ~/work/gstreamer/subprojects/gst-plugins-good (added with --add-dir)
Constraints: use a subagent; read-only; name files and functions.
Output: file, function, one-line role.
```

Start with `claude --add-dir ~/work/gstreamer` to grant access.

**3.4 Toolchain**

```text
Goal: Show where this build chooses its compiler and how CROSS_COMPILE takes effect.
Constraints: read-only; quote the relevant Makefile lines.
```

**3.5 Layered CLAUDE.md (monorepo practice)**

```text
Goal: Create libbb/CLAUDE.md with 4 rules that apply only to shared helpers.
Constraints: concrete and checkable, for example "List all callers before changing a libbb function signature".
```

Start Claude from `libbb/`, run `/context`; start from the repo root, run `/context`; compare the Memory files list.

Add to `.claude/settings.local.json`: `{ "claudeMdExcludes": ["**/libbb/CLAUDE.md"] }`, restart, confirm it no longer loads.

**3.6 Scoped repetitive fix (optional)**

```text
Goal: List every call to strcpy, strcat, sprintf or gets under coreutils/.
Constraints: read-only.
Output: table of file, line, function.
```

```text
Goal: Replace those calls in the first file of the table with bounded forms.
Verify: native build, no new warnings.
```

Review the whole diff before accepting. If the table is empty, widen to `libbb/`.

Done when:

- A caller table exists; at least one caller flagged
- qtdemux registration and sbgp parsing located by file and function
- The `libbb/CLAUDE.md` loads when started in `libbb/`, not when excluded

### Module 4: Testing, Debugging and QA

Assets: BusyBox testsuite, rot13 applet, rot13filter, qemu-arm, Python.
Keywords: failing test, reproduction, expected, consistent, root cause, checkpoint.

**4.1 Find gaps, write tests**

```text
Goal: List 5 applets that have no test file under testsuite/.
Constraints: read-only.
```

```text
Goal: Write testsuite/rot13.tests in the existing format: testing "description" "command" "result" "infile" "stdin".
Context: copy the structure of one small existing *.tests file under @testsuite/.
Constraints: cases: lowercase, uppercase, digits unchanged, empty input, 4 KB input.
Verify: run the test command recorded in CLAUDE.md; all rot13 tests pass.
```

**4.2 Python cross-check**

```text
Goal: Write tools/check_rot13.py (Python 3, stdlib only).
Constraints: generate 50 random strings; compute rot13 with the codecs module; run ./busybox rot13 on each; print mismatches; exit non-zero if any.
Verify: python3 tools/check_rot13.py exits 0.
```

**4.3 Debug with a precise report**

Trainer: introduce an off-by-one in the rot13 loop bound.

Vague (compare the result):

```text
rot13 is broken, please fix it.
```

Precise:

```text
Goal: Fix the rot13 bug.
Context: `echo Hello | ./busybox rot13` prints "Uryy". Expected "Uryyb". Fails on every run.
Constraints: find the root cause, do not special-case the input. Suggest fixes and wait for my choice.
Verify: after I pick one, apply it and rerun the command until it prints Uryyb; then run the test command from CLAUDE.md.
```

**4.4 Checkpoints and their limits**

```text
Goal: Run via Bash: rm -rf build-arm && echo x > scratch.txt. Then add a comment line at the top of the rot13 source file.
```

```text
/rewind
```

Restore code and conversation.

```bash
ls build-arm scratch.txt
```

Expected: the comment is gone; `build-arm` and `scratch.txt` are not restored (Bash changes are not tracked; use git).

**4.5 Test on the target**

```text
Goal: Run the cross-built binary under qemu.
Verify: `echo Hello | qemu-arm -L $SYSROOT build-arm/busybox rot13` prints Uryyb.
Output: the command and its output.
```

**4.6 GStreamer element test (shell)**

```text
Goal: Write tests/rot13_gst.sh (bash) that tests the rot13filter element.
Constraints: set -euo pipefail; create in.txt; run gst-launch-1.0 filesrc location=in.txt ! rot13filter ! filesink location=out.txt with GST_PLUGIN_PATH pointing to the plugin build directory; compare out.txt to the expected rot13 text; exit non-zero on mismatch.
Verify: run it; it exits 0.
```

Done when:

- New rot13 tests pass in the BusyBox test run
- The precise report fixes the bug in fewer corrections than the vague one
- `/rewind` restores the edit but not the deleted files
- `tools/check_rot13.py` and `tests/rot13_gst.sh` exit 0; the qemu output is `Uryyb`

### Module 5: Model Selection and Cost

Assets: BusyBox build log, gcc warnings, rot13 sources.
Keywords: report only, do not print the log, `/model`, `/effort`, `/compact`, `/clear`, `/usage`.

**5.1 Measure context**

```text
/context
```

**5.2 Filter a build log**

```text
Goal: Rebuild natively and save the output.
Constraints: run `make clean`, then the native build with output to build.log; do not print the log.
Output: exit code, warning count, error count only.
```

```text
Goal: Group the gcc warnings in build.log by -W flag with counts.
Constraints: use grep, sort, uniq; do not read the whole file.
Output: table of flag, count.
```

**5.3 Right-size model and effort**

```text
/model
```
Pick a smaller model.

```text
/effort low
```

```text
Goal: Classify each warning type in build.log as ignore, fix or investigate.
Output: one line per type.
```

```text
/model
```
Pick a larger model.

```text
/effort high
```

```text
Goal: Review rot13 (BusyBox applet) and the rot13filter element for memory safety and undefined behaviour.
Constraints: report only findings with a concrete failing input.
Output: ranked list with file and line.
```

**5.4 Compact and clear**

Add to CLAUDE.md under Compaction (if not present): "When compacting, preserve the list of modified files and the test command."

```text
/compact focus on: rot13 design decisions and open warnings
```

```text
/context
```

```text
/clear
```

```text
Goal: Summarize the build and test rules from CLAUDE.md.
Output: 3 lines.
```

**5.5 Cost**

```text
/usage
```

Done when:

- `/context` before and after `/compact` shows the drop
- The warning classification ran on the smaller model and effort, the review on the larger
- `/usage` shows the session cost (an estimate, not an invoice)
