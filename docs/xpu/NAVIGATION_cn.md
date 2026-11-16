# XPU port: navigation guide（中文摘要）

> 原文：[NAVIGATION.md](NAVIGATION.md)

## 文档用途

Intel GPU/XPU 相关说明或实验配方，重点区分已测范围、实验性路径和未验证硬件。

这是中文摘要版，方便快速定位信息；它不是逐句翻译。原文涉及的实现细节、命令、参数、路径、URL、模型名、版本号、issue/PR 编号和测量数字均以[英文原文](NAVIGATION.md)为准。

## 主要主题

原文涉及：XPU、原生 Zig 引擎、API 接口。

主要章节：

- `1. Where things are`
- `2. Building`
- `3. The runner: `qwen27.zig``
- `Load`
- `One forward`
- `Runner API`
- `4. The drafter: `drafter.zig``
- `5. Kernels, by file`
- `6. The server hosts`
- `7. A request, end to end (lane host)`
- `8. The prompt GEMM generator: `w4_ngen.cpp``
- `9. Environment variables`
- `10. The test runner and the correctness contracts`

## 阅读边界

- 配方和资格结论只适用于原文写明的 checkpoint、后端、硬件、运行时和参数组合。
- receipt、benchmark 和性能数字是特定提交和机器上的记录，不能直接推广到其他平台。
- `experimental`、`historical`、`unrun`、`pending` 等状态必须按原文理解，摘要不把它们改写成已完成或已支持。
- 需要执行命令、修改配置或实现代码时，请回到英文原文、源码和当前 CI 结果核对。

## 原文

继续阅读：[NAVIGATION.md](NAVIGATION.md)。
