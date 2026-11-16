# Qwen3.8-27B（中文摘要）

> 原文：[qwen3.8-27b.md](qwen3.8-27b.md)

## 文档用途

模型或后端部署配方，重点说明 checkpoint、启动命令、硬件条件、支持范围和资格限制。

这是中文摘要版，方便快速定位信息；它不是逐句翻译。原文涉及的实现细节、命令、参数、路径、URL、模型名、版本号、issue/PR 编号和测量数字均以[英文原文](qwen3.8-27b.md)为准。

## 主要主题

原文涉及：Metal、CUDA、HIP、原生 Zig 引擎、实验性路径、历史数据、资格验证、验证收据、基准测试、API 接口。

主要章节：

- `GGUF host adapter`
- `MLX`
- `Weights other than 4-bit`
- `CUDA`
- `NVFP4 checkpoints`
- `EXL3 checkpoints (experimental)`
- `MLX 4-bit, one or two ranks`
- `Concurrent requests`
- `Structured output`
- `Historical public-fixture results`
- `A 64 GB M5 Pro on 0.3.5.1`
- `A 48 GB M5 Pro Mac mini on 1.2.1`
- `Calibration and checks`
- `Long-document prompt reuse`

## 阅读边界

- 配方和资格结论只适用于原文写明的 checkpoint、后端、硬件、运行时和参数组合。
- receipt、benchmark 和性能数字是特定提交和机器上的记录，不能直接推广到其他平台。
- `experimental`、`historical`、`unrun`、`pending` 等状态必须按原文理解，摘要不把它们改写成已完成或已支持。
- 需要执行命令、修改配置或实现代码时，请回到英文原文、源码和当前 CI 结果核对。

## 原文

继续阅读：[qwen3.8-27b.md](qwen3.8-27b.md)。
