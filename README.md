# OpenClaw on Termux (Android): install notes and issues encountered

This repo is a practical field note for getting OpenClaw working on an Android/Termux machine, plus a running log of the problems encountered so far and the fixes/workarounds that actually worked.

## Environment

- Device environment: Android + Termux
- CPU/arch: `android-arm64`
- Shell: `bash`
- Node.js: `v25.x`
- OpenClaw version discussed here: `2026.3.24`

## What was installed successfully

### 1. OpenClaw itself

OpenClaw was installed and used successfully in the Termux environment.

### 2. Gmail-related local tooling

For Gmail/OpenClaw work, additional local tooling was installed successfully:

- Google Cloud SDK (`gcloud`)
- Go
- `gog` built from source in the local home directory

### 3. Lark / Feishu tooling

`lark-cli` was installed successfully on Termux, but not through the normal npm prebuilt path.

What worked:

- build/install `lark-cli` from source on the Android/Termux machine
- run `lark-cli config init --new`
- complete `lark-cli auth login`
- verify with `lark-cli doctor`

Result:

- `lark-cli` worked on the machine
- Feishu auth succeeded
- self-test Feishu messaging worked

## Problems encountered so far

### Problem 1: npm/global install paths can break on Android platform detection

Some tools assume standard Linux/macOS targets and do not properly support `android-arm64` in package metadata or prebuilt binary selection.

#### Symptoms

- npm install paths fail even though the tool is otherwise buildable
- package metadata rejects Android platform/arch
- prebuilt binary expectations do not match Termux

#### What worked

- prefer source builds when npm/global prebuilt install is blocked by platform detection
- do not assume a package that works on Linux will install cleanly on Termux without rebuilding

### Problem 2: OpenClaw self-update can fail because of `sharp` / `libvips`

OpenClaw updates on this Termux setup can fail if `sharp` tries to compile or link incorrectly against the environment.

#### Working workaround remembered

Ensure global build dependencies are present, then reinstall OpenClaw with:

```bash
SHARP_IGNORE_GLOBAL_LIBVIPS=1 npm install -g openclaw@<version> --no-fund --no-audit --loglevel error
```

Also helpful to ensure:

- `node-addon-api`
- `node-gyp`

are available in the environment when needed.

### Problem 3: Feishu plugin path changed after upgrading OpenClaw

After upgrading to `2026.3.24`, the Feishu plugin path in `~/.openclaw/openclaw.json` needed to be changed.

#### Old path

```text
.../openclaw/extensions/feishu
```

#### Correct path after upgrade

```text
.../openclaw/dist/extensions/feishu
```

If the Feishu extension appears missing after upgrade, check this path first.

### Problem 4: OpenClaw built-in image reading broke on Termux/Android

This was one of the most important issues encountered so far.

#### Symptoms

Reading images inside OpenClaw failed with an error like:

```text
Could not load the "sharp" module using the android-arm64 runtime
```

This broke built-in image analysis / vision workflows even though the rest of OpenClaw was working.

#### Root cause

OpenClaw's bundled `sharp` dependency did not have a usable runtime path for `android-arm64` in this environment.

#### What worked

Install the WebAssembly sharp runtime into OpenClaw's own dependency tree, forcing npm to accept the wasm package:

```bash
cd /data/data/com.termux/files/usr/lib/node_modules/openclaw
npm_config_cpu=wasm32 npm_config_force=true npm install --no-save @img/sharp-wasm32@0.34.5
```

Then restart the running OpenClaw process/session so it reloads the dependency.

#### Verification

After the fix:

- `sharp` could be loaded from Node successfully
- image metadata could be read successfully from the CLI
- after restarting OpenClaw, built-in image reading worked again
- image/vision analysis recovered successfully

### Problem 5: Android/Termux does not support OpenClaw gateway service management the same way

Trying to use:

```bash
openclaw gateway restart
```

failed with a service-management error on Android.

#### Meaning

On this setup, service-style gateway management is not the right restart path.

#### Practical workaround

Restart the actual running OpenClaw session/process manually instead of relying on daemon service commands intended for other platforms.

## Practical lessons learned

- Termux is workable, but assume some packages will need source builds or manual fixes.
- `android-arm64` is the key incompatibility surface.
- Anything involving native Node modules deserves suspicion first.
- If image support breaks, check `sharp` immediately.
- After OpenClaw upgrades, re-check plugin paths and any native-module-dependent features.
- Local file-based memory is still the most reliable baseline.

## Current status

At the time of writing:

- OpenClaw is usable on this Termux setup
- Feishu/Lark support is working
- local image reading / vision has been recovered
- GitHub CLI auth is working on the machine

## Suggested future additions

This repo can be expanded later with:

- a clean step-by-step install script
- a fresh-machine bootstrap guide
- a troubleshooting checklist
- version-specific notes for future OpenClaw releases
