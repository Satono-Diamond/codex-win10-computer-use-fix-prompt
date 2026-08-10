# Codex Windows 10 Computer Use Fix Prompt

This repository contains an unofficial self-repair Goal prompt for diagnosing
and fixing Codex Computer Use compatibility issues on Windows 10.

It is a prompt-based repair workflow, not a prebuilt binary patch.

The prompt instructs Codex to investigate the CURRENT local Codex installation,
build a machine-specific compatibility solution, test it, and provide rollback.

## Known problem classes

- SetIsBorderRequired / 0x80004002 screenshot failures
- shell:AppsFolder / 0x80070490 failures
- list_apps / application discovery failures
- launch_app compatibility failures

## Important

- No OpenAI binaries are distributed.
- Do not copy compatibility binaries between machines.
- Each user should run the Goal against their own current Codex installation.
- Codex and Computer Use internals may change after updates.
- Windows 10 support is best-effort.
- This is an unofficial community workaround and is not affiliated with or
  endorsed by OpenAI or Microsoft.

## Usage

1. Open Codex on the affected Windows 10 machine.
2. Select a capable coding model and Full access if appropriate for the task.
3. Open PROMPT.md.
4. Copy the Goal prompt into a new Codex project/thread.
5. Let Codex investigate and build against that machine's local installation.
6. Review the generated preflight/install/rollback steps before applying them.

