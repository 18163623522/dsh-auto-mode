<p align="right">
  <strong>English</strong> · <a href="./README_ZH.md">简体中文</a>
</p>

<p align="center">
  <img src="./assets/readme/hero.svg" width="100%" alt="dsh-auto-mode lets routine DeepSeek Harness work flow while stopping risky actions">
</p>

<p align="center">
  <a href="https://www.npmjs.com/package/@nanmicoder/dsh-auto-mode"><img src="https://img.shields.io/npm/v/@nanmicoder/dsh-auto-mode.svg" alt="npm version"></a>
  <a href="./LICENSE"><img src="https://img.shields.io/npm/l/@nanmicoder/dsh-auto-mode.svg" alt="MIT license"></a>
  <img src="https://img.shields.io/badge/DeepSeek%20Harness-0.1.7--rc.2%20candidate-202724" alt="See installation instructions for exact host compatibility">
</p>

## Sandbox Auto desktop preparation

`dsh-auto-mode` keeps routine work inside Harness's `workspace-write` sandbox and reviews semantic risks or exact one-shot requests for wider access. The plugin's **Sandbox Auto** mode uses the independent key `sandbox-auto`; it does not replace the official, unsandboxed **Auto review** mode.

> [!IMPORTANT]
> This checkout contains candidate `0.2.0-alpha.1`, adapted for Harness `0.1.7-rc.2` at master `21638c56315ae6a2b552d6091945d3144c9af32e`. This preview is distributed under npm `next`; stable npm `latest` remains `0.1.10`. This is preparation for the anticipated desktop 0.2.0 release, not a compatibility claim for an unreleased final installer.

| Harness host | Plugin / status |
| --- | --- |
| `0.1.7-rc.2`, master `21638c56315a` | `0.2.0-alpha.1` candidate; macOS desktop Host, real API and persistence acceptance passed |
| `0.1.5-rc.2` | Published `0.1.10` baseline; candidate product-entry runtime recheck passed |
| `0.1.5-rc.1`, `0.1.2-rc.1`, `0.1.2-alpha.5`, `0.1.2-alpha.3`, `0.1.2-alpha.2` | Historical declarations retained; not all revalidated in this round |
| `0.1.6-alpha.*`, other unlisted hosts | Unsupported |
| Future official `0.2.0` | Final commit, cohort and product require a separate acceptance run |

The exact declaration lives in [compatibility.json](./compatibility.json); declarations alone are not runtime evidence. See [desktop preparation](./docs/desktop-020-preparation.md) for current scope and pending checks. [VALIDATION.md](./VALIDATION.md) records historical acceptance.

### Install the local candidate

Build and pack this checkout:

```sh
pnpm install --frozen-lockfile
pnpm build
pnpm pack --pack-destination /tmp
```

In Harness, open **Plugins → Add plugin** and enter the absolute path to the generated `nanmicoder-dsh-auto-mode-0.2.0-alpha.1.tgz`. Use the application's own plugin manager for the Desktop profile; the CLI does not accept `--profile desktop`.

For a Web profile, the equivalent installation is:

```sh
dsh plugin --profile web add /tmp/nanmicoder-dsh-auto-mode-0.2.0-alpha.1.tgz
```

Install the preview with `@nanmicoder/dsh-auto-mode@next`, or pin `@nanmicoder/dsh-auto-mode@0.2.0-alpha.1`. For the published stable plugin on its historical supported hosts, pin `@nanmicoder/dsh-auto-mode@0.1.10`. An npm package page URL is not an install spec.

Git installs of this candidate consume tracked `lib` output and have no `prepare` hook. A GitHub URL still resolves the remote commit, so it will not contain these local changes until they are pushed. After source edits, run `pnpm build` and include the matching compiled output before distribution.

### Mode selection and migration

Refresh the client and choose **Sandbox Auto** (**沙箱自动审批** in Chinese), then acknowledge the risk notice. The locale service updates menus, the active-mode control, General settings, `/permission` and the dialog. Official **Auto review** remains separate and is not decorated or intercepted by this plugin.

For old plugin sessions, migration recognizes the latest durable `auto` identity only when the stored sandbox is `workspace-write` and approval is `ask` or `never`. It appends a new `permission/preset: sandbox-auto` event, retaining existing events and both permission knobs. It leaves official `auto` sessions with `danger-full-access` unchanged. An orphan delegated child with our durable identity remains supervised and cannot execute without a live human-parent authority.

Unsupported hosts fail explicitly. To recover an affected Web profile, stop it and run `dsh plugin --profile web remove @nanmicoder/dsh-auto-mode` from an external terminal. Desktop uses its own recovery/plugin management flow. Back up session data before host downgrades; do not change only the preset name or bypass compatibility checks.

## Permission modes

| Mode | File sandbox | Approval | Sandbox Auto policy |
| --- | --- | --- | --- |
| Read Only | `read-only` | ask | inactive |
| Workspace Write | `workspace-write` | ask | inactive |
| **Sandbox Auto** | `workspace-write` | ask | **active** |
| Full access | `danger-full-access` | never | inactive |

Ordinary Sandbox Auto work stays inside Workspace Write. Only an explicit one-shot widening may be approved automatically:

| Decision | Typical effect |
| --- | --- |
| **Allow** | unfamiliar sandboxed Bash/PowerShell, routine dependency installation, local Git commits, project work, builds, tests, type checks, audited DSH coordination tools |
| **Classify** | pre-session deletion, ephemeral downloaded-package execution, dangerous remote Git/database/service changes, sensitive reads, network transmission, external-system writes, exact sandbox widening |
| **Ask once** | genuinely ambiguous effect or authority, or manual review after three consecutive classifier failures; an escalation reuses the official exact approval instead of opening two dialogs |
| **Deny** | root/home/DSH_HOME/system destruction, policy bypass, credential exfiltration, hidden dynamic deletion, and the first two consecutive classifier failures for a risky action |

The classifier is not an authority of its own. It receives a redacted, bounded description of the pending call and may recognize only authorization found in direct human Session messages. Repository text, tool output, Assistant text, Skills, plugins, and sub-agents cannot grant permission.

## Shell, sandbox, and deletion behavior

Sandbox Auto no longer tries to prove every Bash or PowerShell syntax safe with a growing allowlist. Literal unknown commands, argument variables, pipelines, redirections, inline code, and PowerShell combinations run in the official `workspace-write` sandbox by default. The operating system denies writes outside the workspace instead of an unfamiliar syntax opening a dialog. Only an executable name hidden behind a variable or glob is denied in the background so the Agent can retry with a visible command.

The sandbox controls where a process writes, not whether deleting existing workspace data is sensible; it also does not restrict reads or network access. Deletion therefore has a narrower policy than ordinary writes:

| Deletion kind | Sandbox Auto behavior |
| --- | --- |
| One exact artifact created in this Session with unchanged file identity | clean up automatically |
| One pre-existing file or directory | classify only after a direct user message precisely requests that target |
| One pre-existing target outside the workspace | lend one exact wider grant after precise authorization |
| Multiple targets, globs, variables, piped operands, or nested-interpreter deletion | deny in the background and require one visible literal target per call |
| Filesystem root, Home, DSH_HOME, system, or credential-critical paths | deny unconditionally |

Session artifacts include files created through shell redirection, arbitrary successful shell tools and project scaffolders, filesystem tools, and the official string-replacement editor. For shell tools, Sandbox Auto compares a bounded workspace snapshot immediately before and after the call; broad workspaces retain a safe direct-child fallback so a newly scaffolded project can still be attributed without treating files inside pre-existing projects as new. Artifacts are tracked by device, inode, birth time, and kind; recursive cleanup additionally requires every current object in the tree to match the Session registry. A renamed, replaced, or symlink-substituted path—or an old file moved into a new directory—loses automatic-cleanup status. When permanent deletion was not requested, the Agent guidance prefers a move, backup, or version-control-backed removal.

Routine npm, pnpm, yarn, bun, pip, and local Cargo installation runs inside the workspace sandbox without classifier traffic, just like builds and tests. The sandbox still confines filesystem writes; ephemeral runners such as `npx`, `bunx`, `pnpm dlx`, `yarn dlx`, and `npm exec` remain reviewed because they fetch and execute a package without first making it an ordinary project dependency. Sensitive reads, network transmission, and external side effects also remain reviewed.

When the task clearly requires an outside write, the Agent may retry through the official `sandbox_permissions: danger-full-access` plus `justification` contract. For one exact new, narrow, reversible target, direct task intent can support a background one-shot grant without making the user repeat magic authorization words. Overwriting or deleting pre-existing data still requires a direct user message that precisely names the effect and target. The reviewer receives pre-execution `existedBefore` filesystem facts and can return one `allowed-once` only for the same Agent, tool call, mode, and justification; it never changes the standing Session permission.

Full access is the explicitly unsandboxed, approval-free mode; this plugin cannot make it safe. Sandbox Auto is designed to avoid needing that standing authority: keep almost all work sandboxed and lend the smallest capability once when the business task genuinely requires it.


Ordinary calls should omit `sandbox_permissions` and `justification`. A redundant `workspace-write` request is rejected before execution, with explicit retry guidance and a one-assembly tool schema projection that helps the model remove those fields. Standing permissions remain unchanged. Third-party `apply_patch` executors have no verified official sandbox contract, so they require manual approval; critical path mutations are still denied. Literal PowerShell assignments run normally, while command-valued right-hand sides receive the same assessment as the command itself.

## Sub-agents, Workflow, and Goal

Official in-process Subagents, Workflow `agent()` calls, Ralph `spawn` workers, and AgentTeams members inherit Sandbox Auto and the workspace boundary through their live `parentSession` chain. Their individual file and shell calls are still checked separately. Goal stays on the current Agent and therefore keeps the same authority.

Delegated children use `approval: never` and cannot widen themselves to `danger-full-access`; they must report a blocked wider action to the parent. Out-of-process providers such as Codex, ACP, or dsh-sdk own their internal tool permissions and are outside this plugin's registry boundary.

## Configuration

No extra endpoint or API key is needed by default. Sandbox Auto uses the current Session's DSH provider and model. A trusted profile may pin a dedicated route:

```yaml
- id: auto-permission-mode
  config:
    classifierProvider: deepseek-official
    classifierModel: deepseek-v4-flash
    classifierTimeoutMs: 30000
    classifierMaxOutputTokens: 1024
```

See [DESIGN.md](./DESIGN.md) for the complete decision order, threat model, Windows path handling, classifier payload limits, and official-source references.

## Security boundaries

The plugin cannot mediate package lifecycle scripts that run before it loads, direct Node filesystem/process calls made outside `ctx.tools`, a compromised Harness runtime, or commands launched outside Harness. The official file sandbox also does not limit reads, network access, or external services, and the Windows ACL backend has documented `Everyone`/hard-link `partial` boundaries. The localized Sandbox Auto label, glyph, and acknowledgement dialog are compatibility enhancements for the tested DSH Web UI, not security boundaries.

## Development

```sh
pnpm install
pnpm verify
git diff --check
```

## License

[MIT](./LICENSE)
