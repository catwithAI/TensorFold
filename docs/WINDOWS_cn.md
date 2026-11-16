# TensorFold on Windows (native)（中文摘要）

> 原文：[WINDOWS.md](WINDOWS.md)

## 文档用途

项目专题文档，覆盖 API、环境、评测、缓存、模型行为、性能、语音、视觉或其他运行时能力。

这是中文摘要版，方便快速定位信息；它不是逐句翻译。原文涉及的实现细节、命令、参数、路径、URL、模型名、版本号、issue/PR 编号和测量数字均以[英文原文](WINDOWS.md)为准。

## 主要主题

原文涉及：Metal、CUDA、HIP、原生 Zig 引擎、验证收据、API 接口。

主要章节：

- `What you need`
- `Build`
- `The host tests on a machine with no usable GPU`
- `HIP`
- `Run`
- `Structured output and required tool calls`
- `Models on CUDA`
- `Mixed-width checkpoints`
- `How the port works`
- `Differences from Linux`

## 阅读边界

- 配方和资格结论只适用于原文写明的 checkpoint、后端、硬件、运行时和参数组合。
- receipt、benchmark 和性能数字是特定提交和机器上的记录，不能直接推广到其他平台。
- `experimental`、`historical`、`unrun`、`pending` 等状态必须按原文理解，摘要不把它们改写成已完成或已支持。
- 需要执行命令、修改配置或实现代码时，请回到英文原文、源码和当前 CI 结果核对。

## 原文

继续阅读：[WINDOWS.md](WINDOWS.md)。
