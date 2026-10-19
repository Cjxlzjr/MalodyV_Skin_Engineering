# MalodyV Skin Engineering

> 面向 **MalodyV 皮肤（Skin）工程** 的 Agent Skill：以可追溯的资料和目标运行时证据，辅助 AI 编码助手分析、开发、迁移、调试与审查 Lua 脚本和 Composer 配置。

**仓库名称：** `malodyv-lua`  
**技能名称：** `malodyv-skin-engineering`（见 [`SKILL.md`](SKILL.md) 中的 `name`）

## 项目简介

MalodyV 的皮肤工程往往同时涉及 Lua 回调、Composer 元件、`info.asm`、资源文件、谱面与判定事件，以及特定版本和模式下的宿主行为。仅凭单段 Lua 代码，容易遗漏入口、元件 ID、时间单位、状态重置和部署副本等关键依赖。

本仓库将相关工程规则整理为一套 **AI Agent Skill**：

- [`SKILL.md`](SKILL.md) 负责识别任务、选择参考文档并定义修改与验证边界；
- [`references/`](references/) 提供 API 资料、工程约定、运行时观察、案例和验证方法；
- [`test-prompts.json`](test-prompts.json) 提供包含期望结果的技能行为评测场景；
- [`agents/openai.yaml`](agents/openai.yaml) 提供 Agent 界面展示与默认提示信息。

**这是工程知识与工作流仓库，不是可直接安装到 MalodyV 的成品皮肤，也不包含可独立运行的 Lua 程序。** 使用时仍需提供具体皮肤工程、目标版本、运行平台及实际问题。

## 主要能力

| 方向 | 适用任务 |
| --- | --- |
| Lua 脚本与回调 | 追踪 `Init`、`Update`、`OnHit`、`OnInput`、`OnRetry` 等入口、事件和状态生命周期 |
| Free Mode | 分析 `CanPlay`、`OnProcessNote`、`OnDrawNote`、`OnNoteMove`、虚拟 Note、自定义轨迹与判定 |
| Composer / `info.asm` | 核对皮肤元数据、元件配置、布局、Scenes、Triggers、动画及 Lua 与静态属性的所有权 |
| 游玩数据与时序 | 分析判定事件、长条 Note、分数/准确率、音频与游戏时钟、暂停及重试后的状态恢复 |
| 开发与迁移 | 新皮肤结构规划、历史 UIS 皮肤迁移、现有模块复用与兼容性审查 |
| 性能与 Native | 排查高频回调、Composer 属性写入、缓存、Lua/native 接口与固定负载下的性能问题 |
| 验证与部署 | 设计最小化探针、回归样例、实机验收步骤、备份和回滚方案 |

技能遵循一条核心原则：**先确认实际运行链路和证据，再进行最小范围的修改与验证。**

## 安装与使用

### 1. 在支持 Agent Skills 的编码助手中安装

以支持用户级 Skill 目录的 **Codex（macOS / Linux / WSL）** 为例：

```bash
mkdir -p "$HOME/.agents/skills"
git clone https://github.com/Cjxlzjr/malodyv-lua.git \
  "$HOME/.agents/skills/malodyv-skin-engineering"
```

目录结构应满足：

```text
~/.agents/skills/malodyv-skin-engineering/
├── SKILL.md
├── agents/
│   └── openai.yaml
├── references/
├── test-prompts.json
└── README.md
```

如希望仅在特定项目中使用，可将完整技能目录放入该项目的 `.agents/skills/malodyv-skin-engineering/`。不同 Agent 客户端的发现路径和启用方式可能不同，请以其自身的 Skill 文档为准。

更新用户级安装：

```bash
git -C "$HOME/.agents/skills/malodyv-skin-engineering" pull --ff-only
```

无需安装独立的 Python/Lua 依赖来**阅读或加载本技能文档**；实际皮肤脚本的解析、测试和运行依赖目标项目及其宿主环境。

### 2. 在任务中调用

安装并让客户端发现该 Skill 后，可明确指定技能名称，例如：

```text
使用 $malodyv-skin-engineering 审查这个 MalodyV 皮肤。
请先追踪 info.asm 指定的入口、实际加载的 Lua 模块及 Composer 元件，
列出已确认的问题与待验证事项；这次仅分析，不修改或部署文件。
```

其他典型请求：

```text
使用 $malodyv-skin-engineering 分析 Free Mode 中 OnNoteMove 的轨迹计算，
检查百分比索引、时间单位与 Retry 后的状态重置，并给出最小修复方案。
```

```text
使用 $malodyv-skin-engineering 排查高密度谱面中的掉帧问题。
保留判定和输入反馈行为；先建立可重复的测量基线，再提出优化。
```

```text
使用 $malodyv-skin-engineering 将旧 UIS 皮肤迁移到 MalodyV Composer。
优先保持视觉布局与动画效果，并区分已验证映射和待实机确认的行为。
```

### 3. 提供任务上下文

为获得准确结果，尽量提供：

- **目标环境：** MalodyV 版本、Windows/其他平台、Basic/Free 等模式及键位数；
- **工程文件：** `info.asm`、实际生效的 Lua 入口、被引用的模块与相关资源；
- **问题描述：** 预期表现、实际表现、触发步骤、日志或截图；
- **操作范围：** 仅审查、允许改源码、允许测试，或明确要求部署到哪一份皮肤。

对于涉及运行时的结论，技能会尽可能区分官方文档、目标工程代码、实机探针、推断及未解决的问题。

## 工作流程

1. **识别任务与范围：** 确认当前工作是解释、审查、修改、迁移、性能排查还是部署。
2. **定位实际皮肤链路：** 从元数据指定入口，追踪 Lua 模块、Composer 元件、资源及 Native 依赖；区分源码、生成文件、备份和已部署副本。
3. **按需读取参考资料：** 只读取与问题相关的 `references/` 文档，而不是将整套资料视作当前宿主的事实。
4. **确认关键契约：** 核对版本、模式、回调时机、索引起点、时间单位、事件身份、属性所有权与重置规则。
5. **执行授权范围内的最小改动：** 保留现有接口、序列化结构、资源映射与其他协作者的修改。
6. **分层验证并交付：** 区分静态检查、确定性样例、模拟环境和真实 MalodyV 宿主验收；报告改动、已验证结果及剩余问题。

**源码修改与部署是两种独立操作。** 未得到明确部署要求时，不应自动覆盖正在使用的皮肤；审查任务应保持只读。

## 参考文档导航

所有专题资料位于 [`references/`](references/)；以下按任务分类，避免一次性阅读全部文件。

| 主题 | 对应文档 |
| --- | --- |
| Lua API 与版本差异 | [`official-basic.md`](references/official-basic.md)、[`official-free-mode.md`](references/official-free-mode.md)、[`version-ledger.md`](references/version-ledger.md) |
| Free Mode 与游玩规则 | [`free-mode-runtime.md`](references/free-mode-runtime.md)、[`player-gameplay-rules.md`](references/player-gameplay-rules.md)、[`gameplay-clock-anomalies.md`](references/gameplay-clock-anomalies.md) |
| Composer 与 ASM | [`composer-components.md`](references/composer-components.md)、[`asm-creation.md`](references/asm-creation.md)、[`ui-data-animation.md`](references/ui-data-animation.md)、[`textapi.md`](references/textapi.md) |
| 运行与安全 | [`runtime-deployment.md`](references/runtime-deployment.md)、[`implementation-safety.md`](references/implementation-safety.md)、[`validation.md`](references/validation.md) |
| 优化与可复用实现 | [`performance-engineering.md`](references/performance-engineering.md)、[`native-architecture.md`](references/native-architecture.md)、[`reusable-modules.md`](references/reusable-modules.md) |
| 旧版迁移 | [`legacy-uis-migration.md`](references/legacy-uis-migration.md)、[`legacy-beattube-case-study.md`](references/legacy-beattube-case-study.md) |
| 案例与来源追踪 | [`basic-mode-case-study.md`](references/basic-mode-case-study.md)、[`reference-module-provenance.md`](references/reference-module-provenance.md)、[`chart-identity.md`](references/chart-identity.md) |
| 特定工程约定 | [`project-conventions.md`](references/project-conventions.md)（仅适用于文档所述的历史 Analyzer / MinaCalc 工程） |

其中 `official-basic.md` 与 `official-free-mode.md` 保存的是**历史官方文档摘录**；`version-ledger.md` 汇总已记录的版本变化及特定宿主探针。使用这些资料时，应以任务涉及的目标版本和实机证据确认兼容性。

## 测试与质量检查

仓库中的 [`test-prompts.json`](test-prompts.json) 包含 **16 组提示及预期结果**，用于检查 Skill 在不同任务场景下的规则选择和回答质量，覆盖：

- Composer/ASM 语义及未知字段处理；
- Free Mode、Note 绘制与自定义判定；
- 错误入口、部署副本、宿主不可用等失败场景；
- 性能优化、Lua/native 兼容性、缓存与时钟回退；
- 只读任务与允许修改任务的权限边界；
- 与 MalodyV 无关的 Lua 任务及简单文档修改的非触发场景。

这些是**技能行为评测样例**，不是自动化 Lua 单元测试，也不是实机兼容性证明。可以逐条向支持 Skill 的 Agent 输入 `prompt`，按 `expected` 检查输出，再记录回归差异。

针对实际皮肤修改，请参考 [`validation.md`](references/validation.md) 选择验证层级：

| 验证层级 | 主要检查内容 |
| --- | --- |
| 静态检查 | 文件编码、Lua 语法、回调、模块依赖、文档链接和配置结构 |
| 确定性样例 | 时间边界、状态重置、判定事件、缓存键和算法前后结果 |
| 模拟 / Mock | 在受控输入下检查纯逻辑和调用次序 |
| 目标宿主验收 | 在指定 MalodyV 版本、模式和平台上检查实际加载、显示、游玩、重试及日志 |

## 使用边界

- **宿主差异：** API 是否存在、回调触发时机、`require` 支持、Note/轨道索引、毫秒与秒的转换，都应基于目标环境确认。
- **Composer 所有权：** 静态 ASM 动画与 Lua 写入不应在未定义优先级的情况下同时控制同一属性。
- **事件与时钟：** 判定、Hold、回放、暂停、Retry 和时间回退需要保留事件身份与完整状态重置策略。
- **性能结论：** 本地模拟的调用次数下降不应直接写成实机 FPS 提升。
- **Native 依赖：** 只有在确有需求并验证加载方式、ABI 和结果一致性后，才引入原生计算边界。
- **安全修改：** 保留不相关文件和已有工作；部署前确认目标副本、备份范围与恢复步骤。

## 维护与贡献

建议在新增或修订规则时同步检查：

1. `SKILL.md` 中的适用范围、任务路由及相对链接；
2. 对应 `references/` 文档的资料来源、版本、模式和证据级别；
3. `test-prompts.json` 中是否需要补充正例、反例或回归场景；
4. `agents/openai.yaml` 中的显示信息与技能调用名称是否一致。

对于尚未在目标宿主验证的接口，不应根据单个历史案例直接给出跨版本保证。

## 许可

当前仓库未包含 `LICENSE` 文件。如计划明确允许第三方复制、修改或再分发，建议由仓库维护者选择并添加合适的开源许可证。