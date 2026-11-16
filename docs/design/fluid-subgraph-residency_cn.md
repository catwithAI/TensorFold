# Fluid Subgraph Residency（中文摘要）

> 原文：[fluid-subgraph-residency.md](fluid-subgraph-residency.md)

## 文档用途

架构或设计说明，重点解释组件职责、数据流、边界和实现约束。

这是中文摘要版，方便快速定位信息；它不是逐句翻译。原文涉及的实现细节、命令、参数、路径、URL、模型名、版本号、issue/PR 编号和测量数字均以[英文原文](fluid-subgraph-residency.md)为准。

## 主要主题

原文涉及：Metal、原生 Zig 引擎、资格验证、验证收据。

主要章节：

- `Physical limits`
- `Memory ownership`
- `C0: locality profiler`
- `C1: persistent expert addressing`
- `C2: route-driven residency`
- `C3: bounded next-layer prefetch`
- `PLE rows on SSD`
- `Qualification`
- `Prior art`

## 阅读边界

- 配方和资格结论只适用于原文写明的 checkpoint、后端、硬件、运行时和参数组合。
- receipt、benchmark 和性能数字是特定提交和机器上的记录，不能直接推广到其他平台。
- `experimental`、`historical`、`unrun`、`pending` 等状态必须按原文理解，摘要不把它们改写成已完成或已支持。
- 需要执行命令、修改配置或实现代码时，请回到英文原文、源码和当前 CI 结果核对。

## 原文

继续阅读：[fluid-subgraph-residency.md](fluid-subgraph-residency.md)。
