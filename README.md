# OpenClaw on Termux (Android): Install Notes and Issues Encountered
# OpenClaw 在 Termux（Android）上的安装记录与问题汇总

> Beginner-friendly practical guide for running OpenClaw on Android/Termux, including install steps, WhatsApp pairing notes, OpenAI/Codex model setup, local-memory guidance, and fixes for common native-module failures like `sharp`.
>
> 面向新手的实战指南：把 OpenClaw 跑在 Android/Termux 上，包含安装步骤、WhatsApp 配对、OpenAI/Codex 模型接入、本地记忆方案，以及像 `sharp` 这类原生模块故障的修复记录。

This repository is a practical field note for getting OpenClaw working on an Android/Termux machine, plus a running log of the problems encountered so far and the fixes/workarounds that actually worked.

这个仓库是一份实战记录：主要整理 OpenClaw 在 Android/Termux 环境中的安装过程、到目前为止遇到的问题，以及已经验证可行的修复方法和绕路方案。

## TL;DR | 一句话看点

- Install OpenClaw on Android/Termux | 在 Android/Termux 上安装 OpenClaw
- Set up OpenAI API key or Codex login | 配置 OpenAI API key 或 Codex 登录
- Connect WhatsApp and handle QR/pairing issues | 连接 WhatsApp 并处理二维码/配对问题
- Understand online memory risks and build local memory instead | 了解在线 memory 风险，并搭建本地记忆
- Fix common Termux-native dependency breakage, especially `sharp` image runtime failures | 修复常见 Termux 原生依赖问题，尤其是 `sharp` 识图运行时故障

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

## Memory skills: online memory risk, and why local memory matters | Memory skill 风险、本地记忆为什么重要

### Why this matters | 为什么这块值得单独写

In real use, memory is one of the first things people want from OpenClaw.

在真实使用里，memory 往往是大家最早想要的能力之一。

People usually want the agent to remember:

大家通常希望 agent 记住：

- personal preferences | 个人偏好
- ongoing projects | 正在推进的项目
- past decisions | 过去的决策
- recurring tasks | 重复性任务
- relationship context / communication style | 关系上下文 / 说话风格

But “memory” is also one of the easiest places to create privacy, reliability, and maintenance problems.

但“记忆”也是最容易同时踩到隐私、稳定性、维护成本这些坑的地方。

---

## Risks of online/cloud memory skills | 在线 / 云端 memory skill 的风险

### 1. Privacy and data exposure | 隐私与数据暴露风险

If memory content is sent to external embedding APIs, vector databases, hosted retrieval services, or third-party memory providers, then your personal information is no longer staying only on your own device.

如果 memory 内容需要发到外部 embedding API、向量数据库、托管检索服务，或者第三方 memory 服务，那么你的个人信息就已经不再只停留在你自己的设备里。

Potential risks include:

潜在风险包括：

- personal notes leaving the machine | 个人笔记离开本机
- message history being embedded remotely | 聊天历史被远程做 embedding
- unclear retention policies | 数据保留策略不透明
- logs or telemetry collected by upstream services | 上游服务可能额外收集日志或遥测
- accidental storage of sensitive content | 敏感内容被意外长期保存

### 2. Vendor dependency | 供应商依赖风险

Online memory systems often depend on at least one external service:

在线 memory 系统通常至少依赖一种外部服务：

- embedding provider | embedding 提供商
- hosted vector store | 托管向量库
- cloud database | 云数据库
- proprietary API gateway | 专有 API 网关

If any of those changes pricing, rate limits, quotas, APIs, or account policy, memory can partially or completely stop working.

这些组件里只要有一个改了价格、速率限制、配额、API、账号策略，memory 就可能部分失效，甚至整体失效。

### 3. Cost creep | 成本悄悄上涨

Memory systems that rely on embeddings and retrieval can look cheap at first, but over time the cost can grow with:

依赖 embedding 和检索的 memory 系统，一开始看起来可能不贵，但长期成本会随着下面这些东西一起涨：

- number of notes | 笔记数量
- re-indexing frequency | 重建索引频率
- query volume | 查询频率
- higher-quality embedding models | 更高质量 embedding 模型
- storage and bandwidth | 存储和流量

For a personal setup, this can easily become annoying overhead.

对个人环境来说，这很容易变成烦人的持续开销。

### 4. Reliability and offline failure | 稳定性与离线失效

A cloud-backed memory system can fail because of:

云端 memory 可能因为这些原因失效：

- network problems | 网络问题
- expired credentials | 凭证过期
- API outages | API 故障
- account limits | 账号限制
- provider-side changes | 服务商侧变更

That means the exact moment you want memory most may be the moment it is unavailable.

这意味着：你最需要 memory 的时候，反而可能正好不可用。

### 5. Harder debugging | 更难排查

When memory quality is bad in a cloud pipeline, the failure can be anywhere:

当云端 memory 质量差时，问题可能出在很多层：

- bad chunking | 切块策略不好
- stale embeddings | embedding 过期
- wrong retrieval thresholds | 检索阈值不对
- vector DB indexing problems | 向量库索引问题
- prompt formatting issues | prompt 组装问题

This makes troubleshooting much harder than checking local files.

相比直接检查本地文件，这类问题排查难度高很多。

### 6. Security boundary confusion | 安全边界容易模糊

A lot of users think “memory” means “the agent remembers me locally”.

很多用户天然会把“memory”理解成“agent 在本地记住我”。

But some memory systems really mean:

但有些 memory 系统实际上是：

- upload content
- transform it externally
- retrieve it later through remote infrastructure

- 先上传内容
- 在外部处理
- 再通过远端基础设施取回

That difference matters.

这个区别非常重要。

---

## Why local memory is the safer baseline | 为什么本地记忆更适合作为基线

For a personal OpenClaw setup, local memory is usually the best default.

对于个人 OpenClaw 环境，本地记忆通常是最好的默认方案。

### Advantages | 优势

- data stays on your own device | 数据留在自己的设备上
- easier to inspect | 更容易检查内容
- easier to edit | 更容易手改
- easier to back up | 更容易备份
- easier to version-control | 更容易纳入 git 版本管理
- works without network | 无网络也能工作
- fewer moving parts | 依赖链更短
- much easier to debug | 更容易排查问题

In practice, plain local files are often more trustworthy than a fancy memory stack.

在实践里，朴素的本地文件系统，往往比花哨的 memory 栈更可信。

---

## How to build local memory in a practical way | 如何实用地构建本地记忆

There are multiple reasonable local-memory patterns.

本地记忆有好几种合理做法。

### Option A: Plain files first | 方案 A：先用纯文件

This is the simplest and most robust baseline.

这是最简单也最稳的基线方案。

Recommended structure:

推荐结构：

```text
workspace/
  MEMORY.md
  memory/
    README.md
    2026-03-30.md
    2026-03-31.md
```

Suggested roles:

建议分工：

- `MEMORY.md`: long-term distilled memory | 长期、提炼过的记忆
- `memory/YYYY-MM-DD.md`: daily raw notes | 每日原始记事
- `memory/README.md`: rules for how memory is maintained | memory 维护规则

This approach works well because it is:

这个做法好用，是因为它：

- human-readable | 人能直接看懂
- easy to search with grep/ripgrep | 可以直接 grep/ripgrep 搜
- easy to manually curate | 容易人工整理
- easy to sync with git | 容易和 git 同步

### Option B: Local SQLite memory | 方案 B：本地 SQLite memory

If you want something more structured without introducing cloud dependency, SQLite is a strong local choice.

如果你想在不引入云依赖的前提下做得更结构化，SQLite 是很强的本地方案。

Typical fields might include:

典型字段可以包括：

- id
- timestamp
- source
- summary
- raw_text
- tags
- importance
- privacy_level

Advantages:

优点：

- single local file | 单文件数据库
- portable | 可迁移
- queryable | 可查询
- good enough for personal scale | 足够覆盖个人使用规模

For many users, SQLite is the sweet spot between plain markdown and cloud memory infrastructure.

对很多用户来说，SQLite 正好卡在“纯 markdown”和“云 memory 基础设施”之间的甜点位置。

### Option C: Hybrid memory | 方案 C：混合式 memory

A practical hybrid approach is:

一种很实用的混合方式是：

- keep daily notes in markdown | 每日日志放 markdown
- keep durable facts in `MEMORY.md` | 长期事实放 `MEMORY.md`
- optionally index selected items into SQLite | 可选地把部分内容索引进 SQLite

This gives you readability plus structure without overcomplicating the system.

这样能同时保留可读性和结构化，又不会把系统搞得太复杂。

---

## Recommended design rules for local memory | 本地记忆的设计建议

### 1. Separate raw notes from durable memory | 原始记录和长期记忆分开

Do not dump everything into one file.

不要把所有东西都塞进一个文件。

A good split is:

一个比较合理的拆法：

- daily notes = raw, chronological | 日记：原始、按时间顺序
- long-term memory = curated facts | 长期记忆：提炼后的事实

### 2. Prefer editable text over opaque storage | 优先可编辑文本，不要黑盒存储

If you cannot easily open and inspect the memory, it will eventually become annoying or untrustworthy.

如果一套 memory 不能被轻松打开、查看、手改，那它迟早会变得烦人或者不可信。

### 3. Add privacy judgment | 给记忆加隐私判断

Not every fact should be stored automatically.

不是所有信息都适合自动写进记忆。

A useful rule is:

一个实用规则是：

- durable + non-sensitive → can usually store automatically
- sensitive / personal / financial / identity data → confirm first

- 持久且不敏感 → 通常可以自动记
- 敏感 / 个人 / 财务 / 身份信息 → 先确认再写

### 4. Keep memory searchable | 让记忆可检索

Whether you use files or SQLite, retrieval matters more than volume.

不管你用文件还是 SQLite，检索能力都比堆内容更重要。

If notes cannot be retrieved reliably, they are not useful memory.

如果笔记不能被稳定找回，那它们就不算真正有用的 memory。

### 5. Version it if possible | 尽量纳入版本管理

Git is underrated for personal memory systems.

Git 对个人 memory 系统来说其实很有价值。

Version history helps with:

版本历史可以帮助：

- tracking changes | 跟踪变更
- recovering mistakes | 找回误改
- understanding how context evolved | 理解上下文怎么演变的

---

## My practical recommendation | 我的实用建议

For a personal OpenClaw deployment, I currently recommend this order:

对个人 OpenClaw 部署，我目前建议的优先顺序是：

1. local markdown memory first
2. add SQLite only if needed
3. use online/cloud memory only when there is a clear reason

1. 先上本地 markdown memory
2. 真有需要再加 SQLite
3. 只有在理由非常明确时，才考虑在线 / 云端 memory

If privacy, controllability, cost, and reliability matter, local memory wins most of the time.

如果你在意隐私、可控性、成本和稳定性，大多数时候本地 memory 都更优。

---

## Beginner-friendly install guide for Android/Termux | 给新手的 Android/Termux 安装照抄版

This section is intentionally written for beginners who want something close to “copy this, then fix what breaks.”

这部分是专门给新手写的，目标就是尽量做到“照着抄，哪里炸了就按这里排查”。

### Reality check first | 先说大实话

OpenClaw on Termux is usable, but it is not yet a totally frictionless one-click experience.

OpenClaw 在 Termux 上是能用的，但目前还不是完全无脑一键装好那种体验。

What this means:

这意味着：

- you may need to try more than one install path | 你可能需要尝试不止一种安装路径
- some plugins work immediately, some need manual repair | 有些插件一把过，有些需要手修
- native Node dependencies are the main trouble source | Node 原生依赖是主要故障源
- channel login and model auth are usually separate steps | 渠道登录和模型认证通常是两步事

---

## Step 0: Basic environment prep | 第 0 步：先把基础环境准备好

Suggested baseline:

建议的基础环境：

```bash
pkg update && pkg upgrade -y
pkg install -y git nodejs-lts python clang make pkg-config
```

If a package later fails to build, you may also need:

如果后面遇到构建失败，常见还要补：

```bash
pkg install -y libvips rust
npm install -g node-gyp node-addon-api
```

Notes:

说明：

- `nodejs-lts` is often safer than aggressively new Node versions.
- If you already use a newer Node and things work, you do not have to downgrade immediately.

- `nodejs-lts` 往往比特别新的 Node 版本更稳。
- 如果你已经在用更高版本 Node 且目前没炸，不一定要立刻降级。

---

## Step 1: Install OpenClaw | 第 1 步：安装 OpenClaw

The simplest attempt:

最简单的第一把：

```bash
npm install -g openclaw
```

If this fails around `sharp` / native build issues, try the workaround path:

如果这里卡在 `sharp` / 原生构建问题，试下面这个绕法：

```bash
SHARP_IGNORE_GLOBAL_LIBVIPS=1 npm install -g openclaw --no-fund --no-audit --loglevel error
```

If needed, ensure build helpers exist first:

必要时，先补构建依赖：

```bash
npm install -g node-gyp node-addon-api
```

Check install:

安装后检查：

```bash
openclaw --version
openclaw help
```

---

## Step 2: Run onboarding | 第 2 步：先跑一次 onboarding

For most beginners, this is the easiest entry point:

对大多数新手来说，这一步最省脑子：

```bash
openclaw onboard
```

If you already know you want OpenAI API key auth:

如果你已经明确知道自己要用 OpenAI API key：

```bash
openclaw onboard --auth-choice openai-api-key
```

If you want Codex / ChatGPT subscription auth instead:

如果你想走 Codex / ChatGPT 订阅登录：

```bash
openclaw onboard --auth-choice openai-codex
```

What onboarding usually helps you do:

onboarding 通常会帮你处理：

- workspace setup | 工作区初始化
- model/provider auth | 模型提供商认证
- channel setup | 渠道配置
- gateway basics | gateway 基础配置

---

## Step 3: Choose your model path | 第 3 步：选择模型接入方式

### Option A: OpenAI API key | 方案 A：OpenAI API key

Best if you want stable API usage and direct billing.

适合想要稳定 API 调用、直接按量计费的人。

```bash
openclaw onboard --auth-choice openai-api-key
```

Or set the environment variable first:

或者先设环境变量：

```bash
export OPENAI_API_KEY="sk-..."
openclaw onboard --auth-choice openai-api-key
```

### Option B: Codex / ChatGPT subscription login | 方案 B：Codex / ChatGPT 订阅登录

Best if you want to use Codex-style OpenAI auth rather than normal API-key billing.

适合想走 Codex 风格登录，而不是普通 API key 计费的人。

```bash
openclaw onboard --auth-choice openai-codex
```

You can also run auth directly:

也可以直接单独登录：

```bash
openclaw models auth login --provider openai-codex
```

Important note:

重要提醒：

Codex OAuth and OpenAI API keys are not the same thing.

Codex OAuth 和 OpenAI API key 不是一回事。

### Option C: Claude / Anthropic or other providers | 方案 C：Claude / Anthropic 或其他模型商

If you want Claude, Gemini, OpenRouter, or another provider, `openclaw onboard` / `openclaw configure` is usually the easiest path.

如果你想接 Claude、Gemini、OpenRouter 或别的模型商，通常最简单的做法还是跑 `openclaw onboard` 或 `openclaw configure`。

Useful follow-up commands:

常用后续命令：

```bash
openclaw configure
openclaw models status
openclaw models list
```

If you are unsure whether the model is really configured, check it instead of guessing.

如果你不确定模型到底配好了没，直接查，不要猜。

---

## Step 4: Link a chat channel | 第 4 步：连接聊天渠道

If you only want local testing first, you can skip channels and use dashboard/UI first.

如果你一开始只想本地试，不一定要立刻接聊天渠道，可以先用 dashboard / UI。

But for many people, the first real target is WhatsApp.

但很多人第一目标都是 WhatsApp。

### WhatsApp quick start | WhatsApp 快速开始

Install/login path:

安装/登录路径：

```bash
openclaw channels login --channel whatsapp
```

If the plugin is not installed yet, OpenClaw should offer the install flow.

如果插件还没装，OpenClaw 一般会顺手给出安装流程。

Manual install is also possible:

也可以手动安装：

```bash
openclaw plugins install @openclaw/whatsapp
```

Then start the gateway/listener:

然后启动 gateway / 监听：

```bash
openclaw gateway
```

---

## Common beginner problem: WhatsApp QR code does not show or pairing fails
## 新手高频坑：WhatsApp 二维码不出来 / 配对失败

This is common enough that it deserves its own section.

这个问题很常见，值得单独写。

### Possible symptoms | 常见现象

- QR code does not appear | 二维码不显示
- QR flashes and disappears too fast | 二维码一闪而过
- scan succeeds but session does not stay connected | 扫码成功但会话没连上
- plugin/login command stalls | 登录命令卡住
- WhatsApp Web session dies after initial link | 刚连上就掉线

### What to try first | 第一轮排查建议

1. **Run login again in a clean shell**

```bash
openclaw channels login --channel whatsapp
```

2. **Make sure the WhatsApp plugin is really installed**

```bash
openclaw plugins install @openclaw/whatsapp
```

3. **Start OpenClaw/gateway in the foreground and watch logs**

```bash
openclaw gateway
```

4. **Do not assume Android service commands work the same as desktop Linux**

On Termux, service-management behavior is different.

在 Termux 上，服务管理行为和桌面 Linux 不一样。

5. **Try again after restarting the current OpenClaw process/session**

如果是当前进程状态脏了，直接重开当前 OpenClaw 会话往往比瞎试配置更有用。

### Practical notes | 实战提醒

- WhatsApp linking is often more reliable when you let OpenClaw own a dedicated number.
- Personal-number setups are possible, but they are easier to confuse.
- If the QR flow behaves strangely, suspect the running session state, plugin state, or network/session instability before suspecting your phone.

- 用独立号码给 OpenClaw 挂 WhatsApp，通常更稳。
- 个人号也能用，但更容易绕进混乱状态。
- 如果二维码流程很怪，优先怀疑当前运行进程状态、插件状态、网络/会话不稳，而不是先怀疑手机坏了。

---

## Step 5: Verify model + channel separately | 第 5 步：模型和渠道分开验

A lot of beginners mix these together.

很多新手会把这两件事混成一件。

But they are different:

但其实它们是两件不同的事：

- **Model works** ≠ channel works
- **Channel linked** ≠ model configured correctly

Useful checks:

实用检查命令：

```bash
openclaw models status
openclaw status
```

If chat works but answers fail, check model auth.

如果渠道能连、但回复不正常，先查模型认证。

If model is configured but messages do not arrive, check the channel side.

如果模型没问题、但消息进不来，先查渠道侧。

---

## Step 6: Expect native-module trouble on Android | 第 6 步：默认提防 Android 上的原生模块问题

This is the main theme of Termux/OpenClaw troubleshooting.

这是 Termux/OpenClaw 排错的主线。

Typical examples include:

典型例子包括：

- `sharp`
- image/video/media dependencies
- prebuilt binary mismatch
- npm packages that assume Linux != Android

If something feels mysteriously broken, especially around media handling, check native dependencies first.

如果有东西莫名其妙坏了，特别是和媒体处理有关，优先查原生依赖。

---

## Real example: image reading broke and was later fixed | 真案例：识图坏过，后来修好了

One concrete failure encountered on this setup:

这台机器上已经实际踩过的一个坑：

```text
Could not load the "sharp" module using the android-arm64 runtime
```

This broke OpenClaw image reading / vision.

这会直接让 OpenClaw 的识图/读图失效。

The practical fix that worked:

最后实测有效的修法：

```bash
cd /data/data/com.termux/files/usr/lib/node_modules/openclaw
npm_config_cpu=wasm32 npm_config_force=true npm install --no-save @img/sharp-wasm32@0.34.5
```

Then restart the running OpenClaw process/session.

然后重启当前运行中的 OpenClaw 进程 / 会话。

This is a good example of why Termux users should not panic at the first failure.

这就是一个典型例子：Termux 用户第一次炸时不要慌，很多时候不是“没救”，而是“要补环境”。

---

## Minimal newbie command list | 给新手的最小指令清单

If you only want the shortest practical checklist, use this:

如果你只想要最短的实用清单，可以抄这个：

```bash
pkg update && pkg upgrade -y
pkg install -y git nodejs-lts python clang make pkg-config
npm install -g openclaw
openclaw --version
openclaw onboard
openclaw models status
openclaw channels login --channel whatsapp
openclaw gateway
```

If install fails around native modules:

如果安装卡在原生模块：

```bash
pkg install -y libvips rust
npm install -g node-gyp node-addon-api
SHARP_IGNORE_GLOBAL_LIBVIPS=1 npm install -g openclaw --no-fund --no-audit --loglevel error
```

If image reading later breaks with `sharp` on Android:

如果后面 Android 上识图又因为 `sharp` 炸了：

```bash
cd /data/data/com.termux/files/usr/lib/node_modules/openclaw
npm_config_cpu=wasm32 npm_config_force=true npm install --no-save @img/sharp-wasm32@0.34.5
```

---

## Suggested future additions | 后续可补充内容

This repo can be expanded later with:

后面还可以继续往这个仓库里补：

- a clean step-by-step install script
- a fresh-machine bootstrap guide
- a troubleshooting checklist
- version-specific notes for future OpenClaw releases
- a local-memory starter template
- a minimal SQLite memory schema example
- a dedicated WhatsApp troubleshooting matrix
- a provider-by-provider auth cheat sheet

- 一份更干净的逐步安装脚本
- 一份新机器初始化指南
- 一份故障排查清单
- 后续 OpenClaw 新版本的专项兼容记录
- 一份本地 memory 起步模板
- 一个最小可用的 SQLite memory schema 示例
- 一份专门的 WhatsApp 故障矩阵
- 一份不同模型商认证方式速查表
