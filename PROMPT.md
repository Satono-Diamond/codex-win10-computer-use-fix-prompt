# Codex Windows 10 Computer Use Compatibility — Self Repair Goal

> Unofficial community troubleshooting / compatibility prompt.
>
> This prompt is intended for developers experiencing Codex Computer Use
> compatibility problems on Windows 10.
>
> It does NOT contain OpenAI binaries, patched executables, private runtime
> files, authentication data, or machine-specific paths.
>
> The Goal must investigate and build a compatibility solution against the
> CURRENT LOCAL installation instead of assuming a fixed Codex version.

---

/goal

PROJECT:

CodexWin10ComputerUseCompat

GOAL:

Investigate, implement, test, and install a machine-local compatibility layer
for Codex Computer Use on Windows 10 without upgrading to Windows 11.

The implementation must be derived from the CURRENT machine's:

- Windows version
- Codex AppX version
- Computer Use plugin version
- bundled runtime
- @oai/sky implementation
- local plugin structure

Do NOT assume that files, paths, hashes, or implementations from another
computer are compatible.

============================================================
SUPPORTED PROBLEM CLASSES
============================================================

This Goal should investigate at least the following commonly observed
Windows 10 Computer Use failures.

CASE A — Screenshot capture failure:

SetIsBorderRequired failed:
interface not supported
0x80004002
E_NOINTERFACE

Typical behavior:

- application enumeration works
- activation works
- Accessibility/UI text works
- keyboard/mouse may work
- screenshot=true fails

CASE B — Application discovery / launch failure:

Examples may include:

failed to fetch shell:AppsFolder item
0x80070490

Get-StartApps unexpectedly returning zero or incomplete results

list_apps failing to discover applications

sky.launch_app() failing to resolve or launch an application

CASE C — Other Windows 10 Computer Use compatibility failures.

Do not force all failures into Case A or Case B.

Investigate the actual failure boundary first.

============================================================
FIXED OPERATING MODE
============================================================

This Goal is designed to run in:

FULL ACCESS

unless the user explicitly chooses another permission model.

Do NOT require switching between:

- Full access
- Ask for approval
- Approve for me

as part of the compatibility fix.

Do NOT repeatedly experiment with:

- elevated sandbox
- unelevated sandbox
- private desktop
- workspace-write

unless evidence proves that one of those mechanisms is directly responsible
for the observed failure.

Permission/sandbox behavior is OUT OF SCOPE unless it is itself the confirmed
failure boundary.

============================================================
SAFETY RULES
============================================================

Do NOT:

1. modify OpenAI signed executables
2. modify binaries inside WindowsApps
3. hex-patch codex.exe
4. hex-patch codex-computer-use.exe
5. replace official node.exe
6. replace official node_repl.exe
7. disable Windows security features
8. install kernel drivers
9. modify authentication data
10. modify Codex conversation/history data
11. read or print access tokens, cookies, credentials, or API keys
12. copy binaries from another computer
13. use an older OpenAI binary from another machine as the fix
14. silently modify unrelated Codex plugins
15. modify user Unity/projects during compatibility testing

If modifying the current compatibility configuration is required:

- back it up first
- record hashes
- record every changed path
- provide rollback
- fail closed on unexpected versions or hashes

============================================================
PHASE 0 — CREATE AN ISOLATED PROJECT
============================================================

Create a new project under a user-controlled directory.

Do NOT build directly inside:

WindowsApps
the Codex installation directory
the official plugin directory

Suggested project structure:

src/
bin/
tests/
tools/
logs/
reference/
backups/

README.md
ARCHITECTURE.md
ROOT_CAUSE.md
HYPOTHESES.md
TEST_RESULTS.md
INSTALL.md
ROLLBACK.md
COMPLETION_AUDIT.md

All experiments should occur inside this project until installation is
explicitly required.

============================================================
PHASE 1 — DISCOVER THE CURRENT ENVIRONMENT
============================================================

Collect the CURRENT machine state.

Record:

Windows:
- ProductName
- edition
- DisplayVersion
- build
- UBR
- architecture

Codex:
- AppX package version
- InstallLocation

Computer Use:
- bundled plugin version
- cached plugin versions
- current Skill location
- current runtime location

Find:

codex.exe
codex-command-runner.exe
codex-computer-use.exe
node.exe
node_repl.exe

For each relevant executable record:

- full path
- file size
- last write time
- SHA256
- Authenticode status

Do NOT hardcode these values into the implementation before understanding
what they represent.

Create:

ENVIRONMENT.md

============================================================
PHASE 2 — ESTABLISH THE BASELINE
============================================================

Open a safe application such as Notepad.

Test Computer Use without screenshot if supported.

Determine whether the following work:

- application enumeration
- window enumeration
- window activation
- Accessibility/UI text
- keyboard
- mouse

Then perform exactly one screenshot baseline request.

Record the exact result.

If screenshot fails, preserve:

- error string
- HRESULT
- stack if available
- runtime/helper path
- timestamp

Do not repeatedly trigger a known failure.

Then test:

list_apps

and, if safe:

launch_app for an application that is currently closed.

Again, capture the exact failure rather than guessing.

Create:

BASELINE.md

============================================================
PHASE 3 — BUILD THE REAL CALL GRAPH
============================================================

Investigate the local implementation.

Trace:

Codex
  ↓
Computer Use Skill/plugin
  ↓
sky / @oai/sky
  ↓
node / node_repl
  ↓
native helper
  ↓
Windows APIs

For screenshots determine:

- where screenshot requests are created
- where Windows Graphics Capture is invoked
- where IsBorderRequired / SetIsBorderRequired is called
- whether the call is JS, native addon, or native helper
- screenshot request protocol
- screenshot result protocol

For application launching determine:

- where list_apps is implemented
- how applications are discovered
- whether Get-StartApps is involved
- whether shell:AppsFolder is involved
- how Win32 applications are represented
- how packaged applications/AUMIDs are represented
- where launch_app resolves application identifiers
- where process/window association occurs

Allowed diagnostic methods include:

- source inspection
- JS/MJS inspection
- strings
- dumpbin
- PowerShell
- ProcMon if installed
- Process Explorer if installed
- ETW if useful
- WinDbg if useful
- public Codex source
- official OpenAI documentation

Do not modify signed OpenAI executables.

Document the architecture in:

ARCHITECTURE.md

============================================================
PHASE 4 — MAINTAIN A HYPOTHESIS TABLE
============================================================

Create:

HYPOTHESES.md

Every hypothesis must have one of these states:

UNTESTED
TESTING
CONFIRMED
REJECTED

Suggested hypotheses:

H1:
Windows Graphics Capture itself works on Windows 10 but the
IsBorderRequired property is unsupported.

H2:
The native screenshot helper treats an optional API failure as fatal.

H3:
Computer Use application discovery relies on Windows behavior that differs
between Windows 10 and Windows 11.

H4:
shell:AppsFolder resolution is failing.

H5:
Get-StartApps is incomplete or unreliable in the Computer Use context.

H6:
AUMID resolution is incorrect.

H7:
Win32 and packaged applications are incorrectly routed through the same
resolver.

H8:
The existing Computer Use wrapper changes runtime context.

H9:
The observed error originates somewhere else entirely.

Do not assume these hypotheses are true.

============================================================
ANTI-LOOP RULE
============================================================

The same hypothesis may be tested at most twice.

If two equivalent experiments produce the same result:

mark it either:

CONFIRMED

or

REJECTED

and move on.

Do not repeat the same configuration changes indefinitely.

Do not cycle between permission/sandbox modes without new evidence.

============================================================
PHASE 5 — WINDOWS 10 SCREENSHOT COMPATIBILITY
============================================================

Only perform this phase if screenshot capture is confirmed broken.

Preferred strategy:

Keep Windows Graphics Capture.

If the current Windows version supports the relevant border-control API:

use normal behavior.

If it does NOT support it:

skip the unsupported call.

Use capability detection rather than:

if WindowsVersion == 10

where possible.

Examples:

API presence detection
QueryInterface capability detection
HRESULT handling

The ideal Windows 10 flow is:

Create Graphics Capture session
        ↓
Check whether IsBorderRequired is available
        ↓
available → normal behavior
        ↓
unavailable → skip property
        ↓
continue Windows Graphics Capture

Do not replace Windows Graphics Capture unnecessarily.

============================================================
PHASE 6 — SCREENSHOT FALLBACK
============================================================

Only if Windows Graphics Capture cannot be made reliable:

implement a compatibility screenshot backend.

Preferred fallback order:

1. Windows Graphics Capture without unsupported APIs
2. PrintWindow / PW_RENDERFULLCONTENT
3. BitBlt using appropriate capture flags

The compatibility backend must handle:

- DPI scaling
- multiple monitors
- negative coordinates
- window bounds
- GPU rendered windows
- Unity
- Chromium/Electron
- layered windows

Do not consider "one successful screenshot" complete.

============================================================
PHASE 7 — APPLICATION DISCOVERY COMPATIBILITY
============================================================

Only perform this phase if list_apps / application resolution is broken.

Implement a Windows 10 discovery adapter only if required.

Suggested normalized representation:

AppInfo {
  displayName
  appType
  executable
  arguments
  aumid
  packageFamilyName
  shellAppsFolderId
  source
}

Support at minimum:

- traditional Win32 applications
- packaged applications
- common system applications

Possible discovery sources:

- native Windows application discovery
- Start Menu shortcuts
- AppX registration
- AUMID registration
- executable resolution

Do not depend on Get-StartApps alone if evidence proves it unreliable.

============================================================
PHASE 8 — APPLICATION LAUNCH COMPATIBILITY
============================================================

Keep the official launch path as the preferred path.

Conceptually:

official resolver
      ↓
success?
  yes → official launch

  no
   ↓
Windows 10 compatibility resolver
   ↓
valid executable / AUMID
   ↓
compatibility launch backend

Possible native launch methods:

Win32:
- appropriate native process/shell APIs

Packaged applications:
- AUMID / package activation APIs

PowerShell Start-Process may be used as a diagnostic test.

Do NOT use a PowerShell subprocess as the final implementation unless there
is no suitable native option and the limitation is documented.

============================================================
PHASE 9 — PRESERVE OFFICIAL BEHAVIOR
============================================================

The compatibility layer should be conditional.

Do not replace working official behavior.

Preferred architecture:

official Computer Use
       ↓
try official behavior
       ↓
success ──────────────→ return official result
       │
     known Win10 failure
       ↓
compatibility fallback
       ↓
return equivalent result

Windows 11 behavior should remain untouched wherever possible.

============================================================
PHASE 10 — COMPUTER USE INTEGRATION
============================================================

The final implementation must integrate with the existing Computer Use flow.

For screenshots, the upper layer should continue receiving a usable
screenshot/window state rather than requiring the user to run a separate
manual screenshot command.

For app launch, the desired flow is:

Computer Use
   ↓
resolve application
   ↓
launch
   ↓
wait for process/window
   ↓
discover top-level window
   ↓
activate
   ↓
screenshot
   ↓
continue interaction

Launching a process alone is NOT sufficient.

============================================================
PHASE 11 — SCREENSHOT ACCEPTANCE TESTS
============================================================

If screenshot compatibility was modified:

Test:

Notepad
Explorer
Chrome
Unity Editor

Run at minimum:

10 consecutive captures

then:

100 consecutive captures

Record:

success count
failure count
capture method
p50
p95
maximum latency

Target:

captureFailures = 0

Check:

memory growth
GDI handle growth
USER handle growth
zombie helper processes

============================================================
PHASE 12 — APPLICATION ACCEPTANCE TESTS
============================================================

Test with safe applications available on the machine.

Recommended:

Notepad
Calculator
Explorer
Paint

For each supported app:

closed
  ↓
resolve
  ↓
launch
  ↓
window discovered
  ↓
screenshot

Also test:

nonexistent application
→ should return NotFound or equivalent
→ must not retry forever

For an already running application:

detect existing window
→ activate
→ do NOT start an unnecessary second instance

============================================================
PHASE 13 — END-TO-END COMPUTER USE TEST
============================================================

Perform at least one harmless closed-loop test.

Example:

Notepad closed
  ↓
launch
  ↓
screenshot
  ↓
click editor
  ↓
type temporary test text
  ↓
screenshot
  ↓
verify text
  ↓
close without saving

Also test one graphical system app if available.

Do not modify user projects or valuable data.

============================================================
PHASE 14 — RESTART TEST
============================================================

Fully exit Codex.

Confirm test helpers are not left running.

Restart Codex.

Repeat:

- application discovery
- launch
- screenshot
- click
- screenshot

The implementation must survive a fresh Codex process.

Do not rely on REPL globals or state left from the development session.

============================================================
PHASE 15 — INSTALLATION ENGINEERING
============================================================

If installation is required, generate:

install.ps1
rollback.ps1

The installer must:

- default to preflight/read-only
- require explicit -Apply for writes
- dynamically locate the current Codex installation
- validate Windows build
- validate Codex/plugin structure
- validate expected hashes/structure where appropriate
- create a machine-local backup
- record modified files
- fail closed if assumptions do not match
- never copy binaries from another computer
- never modify WindowsApps signed binaries
- never modify authentication data
- never modify unrelated projects
- provide complete rollback

Do not hardcode a machine-specific user profile path.

Use:

$env:USERPROFILE
$env:LOCALAPPDATA
$env:APPDATA

and dynamically resolved paths.

============================================================
PHASE 16 — FINAL AUDIT
============================================================

Generate:

README.md
ARCHITECTURE.md
ROOT_CAUSE.md
HYPOTHESES.md
TEST_RESULTS.md
INSTALL.md
ROLLBACK.md
COMPLETION_AUDIT.md

The final audit must explicitly state:

- Windows build tested
- Codex version tested
- Computer Use plugin version tested
- what was modified
- what was NOT modified
- whether official signed binaries changed
- hashes of project-owned runtime components
- screenshot results
- launch results
- restart results
- rollback results
- known limitations

============================================================
FINAL ACCEPTANCE CRITERIA
============================================================

Only claim success if all functionality relevant to the diagnosed failure
passes.

For screenshot-related failures:

- Windows 10 remains installed
- screenshots work
- Notepad works
- Explorer works
- Unity works if available
- 100 consecutive screenshots succeed
- no SetIsBorderRequired failure
- restart still works

For launch/discovery-related failures:

- list_apps provides useful results
- common Win32 application resolves
- packaged application resolves where available
- launch works
- resulting window is discovered
- screenshot after launch works
- existing applications are not unnecessarily duplicated
- nonexistent applications fail cleanly
- restart still works

General requirements:

- no OpenAI signed executable patched
- no WindowsApps binary modified
- no binary copied from another machine
- no authentication data modified
- rollback works
- compatibility code is derived against the CURRENT local version

============================================================
IF THE GOAL CANNOT BE COMPLETED
============================================================

Do not fake success.

Report:

- exact failure boundary
- HRESULT / Win32 error
- relevant call chain
- hypotheses tested
- confirmed root cause
- why a safe compatibility layer cannot proceed
- closest successful PoC
- smallest upstream change OpenAI would need to make

The goal is not merely to suppress an error.

The goal is to restore reliable Computer Use behavior on the current
Windows 10 installation while preserving as much official behavior as
possible.

Begin.

