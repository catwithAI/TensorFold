# TensorFold 中文文档导航

本目录的中文文档遵循两种形式：

- `*_cn.md`：核心贡献流程文档的完整中文翻译。
- `*_cn.md`：其他专题、模型配方、API 和实验记录的中文摘要版；摘要保留原文链接、命令、代码、路径、参数名、URL、版本号和测量数字。

中文版本不会替代英文原文。涉及代码行为、精度、性能资格、硬件支持或安全边界时，应以当前英文原文、源码和 CI 结果为准。

## 按贡献流程阅读

- [ATC 协作流程](atc/SKILL_cn.md)：检查 board、检查 issue、认领工作、报告 session 和处理冲突。
- [ATC review exchange](atc/exchange_cn.md)：请求外部 review、构建、测试、benchmark 或复现工作。
- [PR pre-flight](pr/SKILL_cn.md)：准备 PR、填写 receipt、运行 lean 和平台检查。
- [原生贡献流程](recipes/SKILL_cn.md)：模型族、共享核心、GPU kernel、精度、测试和性能证据。
- [模型配方索引](recipes/README_cn.md)：按模型族、后端和硬件查找部署配方。
- [评测 TensorFold](evaluating_cn.md)：固定版本、校验下载、运行无 GPU 检查和复现请求。
- [性能优化](optimizing_cn.md)：先估算收益上限，再 profiling、验证精度和测量性能。

## 按内容类型阅读

| 内容 | 目录或入口 | 适合了解 |
| --- | --- | --- |
| API | [`api.md`](api.md) / [`api_cn.md`](api_cn.md) | HTTP 路由、请求字段、响应和拒绝条件 |
| 环境与配置 | [`environment.md`](environment.md) / [`environment_cn.md`](environment_cn.md) | 环境变量、默认值、取值范围和读取时机 |
| 模型配方 | [`recipes/`](recipes/) | checkpoint、启动参数、平台和资格边界 |
| 性能收据 | [`receipts/`](receipts/) | 同机器、同模型、同参数的验证记录 |
| 运行时专题 | [`runtime/`](runtime/) | CUDA、复制、加载和运行时资产 |
| XPU | [`xpu/`](xpu/) | Intel GPU 的实验性支持和配方 |
| 视觉、语音和 Living Weights | [`vision.md`](vision.md)、[`speech-controls.md`](speech-controls.md)、[`living-weights.md`](living-weights.md) | 多模态、音频和实验性学习功能 |

## 文件命名约定

例如：

```text
docs/optimizing.md       英文原文
docs/optimizing_cn.md    中文版本
```

中文摘要版不改变英文文档的语义，不复制可能过时的完整数据表；需要精确数值时请打开同名英文原文核对。

## 贡献中文文档

文档改动也遵循仓库的贡献流程：

1. 修改前查看 [ATC board](atc/SKILL_cn.md) 并确认没有重复工作。
2. 保留原文的命令、路径、代码块、链接、参数名和数字。
3. 不把实验结果、硬件验证或模型资格从摘要扩展成原文没有的结论。
4. 运行 `git diff --check`，并检查相对链接和 Markdown 代码块。
5. 一个 PR 保持一个清晰的文档行为；在 PR 描述中说明翻译版或摘要版的范围。
