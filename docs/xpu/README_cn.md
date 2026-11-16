# Intel Arc Pro B70 (XPU) backend for the TensorFold Zig engine（中文摘要）

> 原文：[README.md](README.md)

## 文档用途

Intel GPU/XPU 相关说明或实验配方，重点区分已测范围、实验性路径和未验证硬件。

这是中文摘要版，方便快速定位信息；它不是逐句翻译。原文涉及的实现细节、命令、参数、路径、URL、模型名、版本号、issue/PR 编号和测量数字均以[英文原文](README.md)为准。

## 主要主题

原文涉及：XPU、原生 Zig 引擎、API 接口。

主要章节：

- `What it serves`
- `Building`
- `Running`
- `Numbers (B70 at 230 W, server `--parallel 8 --context 16384`, validation of 2026-10-13)`
- `Correctness: what is verified`
- `Code layout`
- `Test executable`
- `Limitations`

## 阅读边界

- 配方和资格结论只适用于原文写明的 checkpoint、后端、硬件、运行时和参数组合。
- receipt、benchmark 和性能数字是特定提交和机器上的记录，不能直接推广到其他平台。
- `experimental`、`historical`、`unrun`、`pending` 等状态必须按原文理解，摘要不把它们改写成已完成或已支持。
- 需要执行命令、修改配置或实现代码时，请回到英文原文、源码和当前 CI 结果核对。

## 原文

继续阅读：[README.md](README.md)。
