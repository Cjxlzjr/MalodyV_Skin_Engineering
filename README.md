# malodyv-lua

这是一个面向 **MalodyV 皮肤/Lua 脚本工程**的参考仓库，主要提供：

- 项目规范与工作约束（`SKILL.md`）
- 各类主题参考文档（`references/`）
- 测试提示样例（`test-prompts.json`）

## 仓库结构

- `SKILL.md`：任务边界、证据标准与执行流程说明
- `references/`：MalodyV 相关能力说明、兼容性记录、性能与安全实践等文档
- `agents/`：代理配置文件
- `test-prompts.json`：测试/验证用提示数据

## 适用场景

- 了解 MalodyV Lua 回调与工程约束
- 查阅版本兼容与能力变更（如 `version-ledger.md`）
- 作为编写/审查皮肤脚本时的参考资料库

## 使用方式

1. 先阅读 `SKILL.md` 了解整体规则与流程。
2. 按需求进入 `references/` 中对应主题文档查阅。
3. 修改或新增脚本时，遵循文档中的边界、单位、生命周期与验证要求。

## 说明

本仓库当前以文档与参考资料为主，不包含完整的可直接运行皮肤工程。
