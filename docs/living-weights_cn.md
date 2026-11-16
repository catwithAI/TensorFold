# Living Weights: how to use it（中文摘要）

> 原文：[living-weights.md](living-weights.md)

## 文档用途

项目专题文档，覆盖 API、环境、评测、缓存、模型行为、性能、语音、视觉或其他运行时能力。

这是中文摘要版，方便快速定位信息；它不是逐句翻译。原文涉及的实现细节、命令、参数、路径、URL、模型名、版本号、issue/PR 编号和测量数字均以[英文原文](living-weights.md)为准。

## 主要主题

原文涉及：CUDA、实验性路径、资格验证、验证收据、API 接口。

主要章节：

- `Repeating a lesson`
- `What you need`
- `Quick start`
- `1. Install or upgrade TensorFold`
- `2. Get the model and make a copy to teach`
- `3. Make a key and start the server with `--slide``
- `4. Teach it a fact`
- `5. Ask`
- `How it works`
- `Teaching it more`
- `From omp`
- `See what it learned`
- `Audit a pocket`
- `Delete one fact`
- `Keep, move or reset what it learned`
- `Undo what it learned`
- `Who can teach it`
- `When something goes wrong`
- `What to expect`
- `Fixed drift and supplied recall`

## 阅读边界

- 配方和资格结论只适用于原文写明的 checkpoint、后端、硬件、运行时和参数组合。
- receipt、benchmark 和性能数字是特定提交和机器上的记录，不能直接推广到其他平台。
- `experimental`、`historical`、`unrun`、`pending` 等状态必须按原文理解，摘要不把它们改写成已完成或已支持。
- 需要执行命令、修改配置或实现代码时，请回到英文原文、源码和当前 CI 结果核对。

## 原文

继续阅读：[living-weights.md](living-weights.md)。
