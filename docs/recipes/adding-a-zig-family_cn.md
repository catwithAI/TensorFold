# Adding a Zig family, and taking it to full speed（中文摘要）

> 原文：[adding-a-zig-family.md](adding-a-zig-family.md)

## 文档用途

模型或后端部署配方，重点说明 checkpoint、启动命令、硬件条件、支持范围和资格限制。

这是中文摘要版，方便快速定位信息；它不是逐句翻译。原文涉及的实现细节、命令、参数、路径、URL、模型名、版本号、issue/PR 编号和测量数字均以[英文原文](adding-a-zig-family.md)为准。

## 主要主题

原文涉及：Metal、CUDA、HIP、XPU、原生 Zig 引擎、API 接口。

主要章节：

- `How to use it`
- `Build each step once, in the core`
- `Tokenizer byte API`
- `The exactness checks`
- `Stage 0: exact on day one`
- `Stage 1: the decode round`
- `Stage 2: fill the lanes`
- `Stage 3: the prompt path`
- `Stage 4: serve`
- `Across several Macs`
- `On CUDA: Nemotron served on a GB10`
- `What to attack first, by model type`
- `Don't bother: built, run, and lost`
- `Traps that cost a day each`
- `Next steps, not yet proven`
- `Toolchain: Zig is pinned to 0.17.0`

## 阅读边界

- 配方和资格结论只适用于原文写明的 checkpoint、后端、硬件、运行时和参数组合。
- receipt、benchmark 和性能数字是特定提交和机器上的记录，不能直接推广到其他平台。
- `experimental`、`historical`、`unrun`、`pending` 等状态必须按原文理解，摘要不把它们改写成已完成或已支持。
- 需要执行命令、修改配置或实现代码时，请回到英文原文、源码和当前 CI 结果核对。

## 原文

继续阅读：[adding-a-zig-family.md](adding-a-zig-family.md)。
