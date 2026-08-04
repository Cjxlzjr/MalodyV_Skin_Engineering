# MalodyV Analyzer 项目脚本规范

## 1. 目录与职责

- `master_analysis.lua`：皮肤唯一入口/调度器；只暴露引擎回调，负责状态机、UI 绑定、事件分发和重置。
- `MinaCalc.lua`：难度计算总线；将输入的有序 `NoteInfo` 行数据转换为技能维度结果。
- `MinaHelpers.lua`：数学、浮点降级、位图和通用工具。
- `MinaStructures.lua`：枚举、数据结构、滑动窗口等基础类型。
- `MinaSequencing.lua` / `MinaAdvSequencing.lua`：序列特征识别。
- `MinaBaseDiff.lua` / `MinaModsHA.lua` / `MinaModsHD.lua` / `MinaModsHDWide.lua`：基础难度和模式修正。
- 其他 `*_algorithm.lua`：可替换的独立算法模块，不直接依赖 UI 生命周期。

## 2. 模块边界

- 模块通过 `local M = {}` 定义公开 API，内部实现使用 `local function`，文件末尾 `return M`。
- `master_analysis.lua` 通过 `require("module_name")` 加载算法；不要把模块内部临时变量写入 `_G`。
- 算法输入输出应使用普通 Lua table；不要在计算模块中直接操作 `Module`、`Play` 或 `Note`。
- UI 和游戏引擎 API 只在入口脚本或显式传入的适配器中调用。
- 对可选模块或可选引擎接口使用 `pcall`、类型检查和默认值，避免初始化失败拖垮整个皮肤。

## 3. 生命周期

推荐保持以下入口职责：

- `Init()`：查找元件、读取 Composer 配置、初始化状态、解析谱面、建立缓存；只做一次性工作。
- `Update()`：每帧处理轻量更新；重计算使用时间间隔或 dirty 标记节流。
- `OnHit()`：读取 `Game:HitEvent()`，处理判定、偏移、连击、统计和视觉反馈。
- `OnInput()`：读取输入事件，处理按下/抬起、键位映射和按键持续时间。
- `OnRetry()`：清理所有游玩状态、指针、统计、图表、粒子和谱面缓存，再重新初始化速度/谱面相关数据。
- `OnDrawNote(info)`：只负责选择 Note 类型并添加子元件；不要在这里做全谱扫描。
- `OnNoteMove(info, mod, percent)`：按 `percent[0]` 更新 Note 的位置、缩放或透明度；避免创建对象。

## 4. 状态设计

- 按功能拆分状态表，例如 `UR_STAT`、`DUR`、`FREQ`、`ADV`、`ACF`、`ANOVA`、`VIS`、`SR_MANAGER`。
- 状态初始化、事件更新、UI刷新分成独立函数。
- 使用 `next_update`、`dirty`、环形队列、预分配对象池控制每帧开销。
- 重试时显式重置所有指针和累计量；不要依赖 Lua 局部变量重新创建。

## 5. 数值语义

- MalodyV API 文档中的动画时间、Note 时间和事件时间通常以毫秒为单位；项目内部的 MinaCalc `row_time` 使用秒，转换入口必须明确 `/ 1000.0`。
- `Game:HitEvent():Offset()` 是引擎报告的命中偏移，项目会结合 `GLOBAL_TURBO` 还原真实偏移。
- `JudgeResult()` 使用官方判定枚举；当前项目按 `1..4` 处理 Best/Cool/Good/Miss，修改时必须先核对文档和现有 UI 映射。
- `info.x`、`info.time`、`info.endtime` 等字段的单位和索引必须以官方文档为准；项目当前 `OnNoteMove` 按 1-based 轨道计算 `(info.x - 2.5) * TRACK_WIDTH`，若实际环境为 0-based 必须整体调整。
- `NoteInfo.row_notes` 是 4K 位图：列 1/2/3/4 对应 `1/2/4/8`；同一时间容差内的列合并为 chord。
- 任何 `Chart:ChartInfo()` 字符串都先 `tonumber` 再参与数值计算；缺失值要提供默认值。

## 6. 性能与安全

- 不要在 `Update` 中反复 `Module:Find`、解析整张谱面或创建大量元件。
- 使用 `Module:Find` 缓存元件引用；使用 `Module:GetString/GetNumber/GetBool` 读取 Composer 配置。
- 自由模式的 `Chart`、`Note`、`Play`、`Score` 接口必须只在自由模式启用并且对应回调允许的场景使用。
- `Note:GetNoteModule`、`Note:AddSprite`、`Note:AddGrid` 只在 `OnDrawNote` 相关流程使用；`Chart:AddVirtual` 只在 `OnProcessNote` 使用。
- 长条 Note 必须根据 `endtime > time` 选择 `hold` 模块，并分别添加 head/body/tail 等子部件。
- 批量处理前检查 `nil`、空表、长度、数值范围；计算模块错误用 `pcall` 隔离并更新 debug 面板，而不是让入口函数报错。

## 7. 调试流程

1. 先确认脚本是否能完成 `Init()`；启动失败优先检查顶层 `require`、语法、不可用标准库和乱码注释吞掉的代码。
2. 再确认 `Chart`/`Play`/`Game` 接口是否在当前模式可用。
3. 用最小谱面验证 Note 数量、轨道索引、时间单位和 Hold 字段。
4. 再启用图表、统计和 MinaCalc；每次只加入一个模块。
5. 将关键状态写入现有 `debug_text`，不要在每帧无界输出。
6. 对修改后的 Lua 文件执行语法解析，并检查所有回调是否仍然存在。

## 8. 修改原则

- 优先最小补丁和根因修复，不重写已经工作的算法。
- 保留现有注释和模块命名；新增代码遵循 snake_case 局部变量、全大写配置常量、功能前缀状态表。
- 不把官方文档未确认的行为当成事实；遇到索引、单位、事件触发时机不确定时，先查 references 或向用户询问。
