# OpenClaw on Termux (Android): Install Notes and Issues Encountered
# OpenClaw 在 Termux（Android）上的安装记录与问题汇总

This repository is a practical field note for getting OpenClaw working on an Android/Termux machine, plus a running log of the problems encountered so far and the fixes/workarounds that actually worked.

这个仓库是一份实战记录：主要整理 OpenClaw 在 Android/Termux 环境中的安装过程、到目前为止遇到的问题，以及已经验证可行的修复方法和绕路方案。

---

## Environment | 环境

- Device environment | 设备环境: Android + Termux
- CPU/arch | 架构: `android-arm64`
- Shell: `bash`
- Node.js: `v25.x`
- OpenClaw version discussed here | 本文涉及的 OpenClaw 版本: `2026.3.24`

---

## What was installed successfully | 已成功安装的内容

### 1. OpenClaw itself | OpenClaw 本体

OpenClaw was installed and used successfully in the Termux environment.

OpenClaw 本体已经可以在 Termux 环境中安装并正常使用。

### 2. Gmail-related local tooling | Gmail 相关本地工具链

For Gmail/OpenClaw work, additional local tooling was installed successfully:

用于 Gmail / OpenClaw 相关工作的额外本地工具也已成功安装：

- Google Cloud SDK (`gcloud`)
- Go
- `gog` built from source in the local home directory | `gog` 已在本地 home 目录源码编译成功

### 3. Lark / Feishu tooling | Lark / 飞书工具链

`lark-cli` was installed successfully on Termux, but not through the normal npm prebuilt path.

`lark-cli` 已成功安装到 Termux，但不是通过普通 npm 预编译包路径完成的。

What worked | 实际可用方案：

- build/install `lark-cli` from source on the Android/Termux machine
- run `lark-cli config init --new`
- complete `lark-cli auth login`
- verify with `lark-cli doctor`

- 在 Android/Termux 机器上源码构建/安装 `lark-cli`
- 执行 `lark-cli config init --new`
- 完成 `lark-cli auth login`
- 用 `lark-cli doctor` 验证

Result | 结果：

- `lark-cli` worked on the machine
- Feishu auth succeeded
- self-test Feishu messaging worked

- `lark-cli` 可以在这台机器上正常工作
- 飞书认证成功
- 飞书自发消息测试成功

---

## Problems encountered so far | 到目前为止遇到的问题

### Problem 1: npm/global install paths can break on Android platform detection
### 问题 1：npm / 全局安装路径会被 Android 平台识别卡住

Some tools assume standard Linux/macOS targets and do not properly support `android-arm64` in package metadata or prebuilt binary selection.

有些工具默认只考虑标准 Linux/macOS 目标平台，没有正确处理 `android-arm64`，会在包元数据或预编译二进制选择阶段直接出问题。

#### Symptoms | 现象

- npm install paths fail even though the tool is otherwise buildable
- package metadata rejects Android platform/arch
- prebuilt binary expectations do not match Termux

- npm 安装流程直接失败，但工具本身其实是能编译的
- 包元数据拒绝 Android 平台/架构
- 预编译二进制和 Termux 环境不匹配

#### What worked | 可行做法

- prefer source builds when npm/global prebuilt install is blocked by platform detection
- do not assume a package that works on Linux will install cleanly on Termux without rebuilding

- 一旦 npm / 全局预编译安装被平台识别拦住，优先改走源码构建
- 不要默认“Linux 能装 = Termux 也能直接装”

---

### Problem 2: OpenClaw self-update can fail because of `sharp` / `libvips`
### 问题 2：OpenClaw 自更新可能卡在 `sharp` / `libvips`

OpenClaw updates on this Termux setup can fail if `sharp` tries to compile or link incorrectly against the environment.

在这个 Termux 环境里，OpenClaw 更新时如果 `sharp` 编译或链接方式不对，就可能直接失败。

#### Working workaround remembered | 当前记住的有效绕法

Ensure global build dependencies are present, then reinstall OpenClaw with:

先确保全局构建依赖存在，然后用下面的方式重装 OpenClaw：

```bash
SHARP_IGNORE_GLOBAL_LIBVIPS=1 npm install -g openclaw@<version> --no-fund --no-audit --loglevel error
```

Also helpful to ensure | 同时最好确认环境里有：

- `node-addon-api`
- `node-gyp`

---

### Problem 3: Feishu plugin path changed after upgrading OpenClaw
### 问题 3：升级 OpenClaw 后 Feishu 插件路径发生变化

After upgrading to `2026.3.24`, the Feishu plugin path in `~/.openclaw/openclaw.json` needed to be changed.

升级到 `2026.3.24` 后，`~/.openclaw/openclaw.json` 里的 Feishu 插件路径需要修改。

#### Old path | 旧路径

```text
.../openclaw/extensions/feishu
```

#### Correct path after upgrade | 升级后正确路径

```text
.../openclaw/dist/extensions/feishu
```

If the Feishu extension appears missing after upgrade, check this path first.

如果升级后发现 Feishu 扩展像是“没了”，第一时间先检查这个路径。

---

### Problem 4: OpenClaw built-in image reading broke on Termux/Android
### 问题 4：OpenClaw 内置识图/读图在 Termux/Android 上失效

This was one of the most important issues encountered so far.

这是到目前为止最关键的问题之一。

#### Symptoms | 现象

Reading images inside OpenClaw failed with an error like:

在 OpenClaw 里读取图片时报错，典型报错类似：

```text
Could not load the "sharp" module using the android-arm64 runtime
```

This broke built-in image analysis / vision workflows even though the rest of OpenClaw was working.

这会直接导致 OpenClaw 的内置识图/视觉分析流程失效，哪怕其他功能还在正常工作。

#### Root cause | 根因

OpenClaw's bundled `sharp` dependency did not have a usable runtime path for `android-arm64` in this environment.

OpenClaw 自带的 `sharp` 依赖在这个环境下没有可用的 `android-arm64` 运行时路径。

#### What worked | 实测可用修法

Install the WebAssembly sharp runtime into OpenClaw's own dependency tree, forcing npm to accept the wasm package:

把 WebAssembly 版 sharp 运行时装进 OpenClaw 自己的依赖树里，并强制 npm 接受 wasm 包：

```bash
cd /data/data/com.termux/files/usr/lib/node_modules/openclaw
npm_config_cpu=wasm32 npm_config_force=true npm install --no-save @img/sharp-wasm32@0.34.5
```

Then restart the running OpenClaw process/session so it reloads the dependency.

然后重启正在运行的 OpenClaw 进程 / 会话，让它重新加载依赖。

#### Verification | 验证结果

After the fix:

修复后：

- `sharp` could be loaded from Node successfully
- image metadata could be read successfully from the CLI
- after restarting OpenClaw, built-in image reading worked again
- image/vision analysis recovered successfully

- `sharp` 可以在 Node 里成功加载
- 命令行已经可以正常读取图片 metadata
- 重启 OpenClaw 后，内置读图恢复正常
- 图像/视觉分析功能恢复正常

---

### Problem 5: Android/Termux does not support OpenClaw gateway service management the same way
### 问题 5：Android/Termux 不支持常规 OpenClaw gateway 服务管理方式

Trying to use:

尝试使用：

```bash
openclaw gateway restart
```

failed with a service-management error on Android.

会在 Android 上报服务管理相关错误。

#### Meaning | 这意味着

On this setup, service-style gateway management is not the right restart path.

在这个环境里，不能把 gateway 当成普通桌面/Linux 服务那样去重启。

#### Practical workaround | 实际绕法

Restart the actual running OpenClaw session/process manually instead of relying on daemon service commands intended for other platforms.

应当手动重启实际运行中的 OpenClaw 进程 / 会话，而不是依赖其他平台才适用的 daemon 服务命令。

---

## Practical lessons learned | 实战经验总结

- Termux is workable, but assume some packages will need source builds or manual fixes.
- `android-arm64` is the key incompatibility surface.
- Anything involving native Node modules deserves suspicion first.
- If image support breaks, check `sharp` immediately.
- After OpenClaw upgrades, re-check plugin paths and any native-module-dependent features.
- Local file-based memory is still the most reliable baseline.

- Termux 能用，但要默认有些包需要源码构建或手动修。
- `android-arm64` 是最核心的不兼容面。
- 只要涉及 Node 原生模块，优先怀疑环境兼容问题。
- 一旦识图挂了，先查 `sharp`。
- OpenClaw 升级后，要重新检查插件路径和所有依赖原生模块的功能。
- 本地文件型记忆依然是最稳的基线方案。

---

## Current status | 当前状态

At the time of writing:

写下这份记录时，当前状态如下：

- OpenClaw is usable on this Termux setup
- Feishu/Lark support is working
- local image reading / vision has been recovered
- GitHub CLI auth is working on the machine

- OpenClaw 已可在这套 Termux 环境中使用
- Feishu / Lark 支持可用
- 本机识图 / 读图功能已经恢复
- GitHub CLI 认证可用

---

## Suggested future additions | 后续可补充内容

This repo can be expanded later with:

后面还可以继续往这个仓库里补：

- a clean step-by-step install script
- a fresh-machine bootstrap guide
- a troubleshooting checklist
- version-specific notes for future OpenClaw releases

- 一份更干净的逐步安装脚本
- 一份新机器初始化指南
- 一份故障排查清单
- 后续 OpenClaw 新版本的专项兼容记录
