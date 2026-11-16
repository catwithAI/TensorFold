# CUDA implementation（中文摘要）

> 原文：[cuda.md](cuda.md)

## 文档用途

模型或后端部署配方，重点说明 checkpoint、启动命令、硬件条件、支持范围和资格限制。

这是中文摘要版，方便快速定位信息；它不是逐句翻译。原文涉及的实现细节、命令、参数、路径、URL、模型名、版本号、issue/PR 编号和测量数字均以[英文原文](cuda.md)为准。

## 主要主题

原文涉及：Metal、CUDA、HIP、冻结的 Python 0.6.6、原生 Zig 引擎、实验性路径、历史数据、资格验证、验证收据、基准测试、API 接口。

主要章节：

- `Engine versions`
- `Python 0.6 CUDA families`
- `Checkpoints`
- `Native prompt controls`
- `Arithmetic and state`
- `Shared kernels and prefill`
- `Prompt precision`
- `NVFP4 precision`
- `Requests and memory`
- `Multi-GPU P2P startup report`
- `Measuring`

## 阅读边界

- 配方和资格结论只适用于原文写明的 checkpoint、后端、硬件、运行时和参数组合。
- receipt、benchmark 和性能数字是特定提交和机器上的记录，不能直接推广到其他平台。
- `experimental`、`historical`、`unrun`、`pending` 等状态必须按原文理解，摘要不把它们改写成已完成或已支持。
- 需要执行命令、修改配置或实现代码时，请回到英文原文、源码和当前 CI 结果核对。

## 原文

继续阅读：[cuda.md](cuda.md)。
