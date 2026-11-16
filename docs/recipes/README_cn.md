# 配方手册

每个家族页面都描述其支持的检查点、内核和运行限制，并说明覆盖的是哪种引擎：原生引擎，或冻结在 0.6.6 的 Python 引擎。

使用智能体辅助贡献时，在实现原生模型家族、内核或服务改动前，请阅读 [TensorFold 贡献者技能](SKILL_cn.md)。

| 家族 | 配方 |
| --- | --- |
| dots.mocr | [原生 HIP OCR，BF16，单 GPU](dots-mocr-hip.md) |
| dots.mocr | [原生 CUDA 文本解码器，BF16，单 GPU](dots-mocr-cuda.md) |
| Kimi K3 / K3.1 | [原生 Metal 初步适配，仅合成测试，常驻预算不足时拒绝](kimi-k3-metal.md) |
| Nemotron 3.5 Lightning | [MLX](nemotron-3.5.md)、[Intel Arc，实验性](xpu-arc.md) |
| Nemotron 3.5 Parse | [原生 HIP OCR，验证与资格认证限制](nemotron-parse.md) |
| Nemotron 3.5 Super VL | [在一台或两台 GB10 上使用原生 CUDA，支持图像](nemotron-3.5-super-vl.md) |
| Qwen3.8-27B | [MLX、量化和 CUDA](qwen3.8-27b.md) |
| Qwen3.5-2B | [原生 Metal 预览，权重绑定的 affine 4-bit](qwen3.5-2b-native.md) |
| Qwen3.8 Flash Next | [MLX prefill 和 CUDA](qwen3.8-flash-next.md)、[原生 CUDA](flash-next-cuda.md)、[CUDA 图像](flash-next-vision.md)、[Metal 上的精确 SSD 驻留](fluid-subgraph-residency.md) |
| Qwen4 dense | [原生准备工作，设备资格认证尚未完成](qwen4-dense.md) |
| Ternary Bonsai 2 27B | [MLX](ternary-bonsai-2.md) |
| GLM-5.3-Flash | [256 GB Mac 上的 MLX，双 rank CUDA](glm-5.3-flash.md)、[原生 CUDA](glm-5.3-flash-cuda.md) |
| Gemma 4 26B-A4B | [MLX，融合的单行解码](gemma-4.md) |
| Gemma 4 31B | [原生 Metal，affine 4-bit](gemma-4-native.md) |
| Gemma 4 12B | [原生 Metal，affine 4-bit，MTP 辅助模型](gemma-4-12b-native.md) |
| DeepSeek-V4-Flash | [256 GB Mac 上的 MLX、DSpark 和 MTP 草稿](deepseek-v4-flash.md) |
| Qwen3.6-35B-A3B | [带 MTP 草稿的 MLX，单 GPU CUDA](qwen3.6-moe.md) |
| Step-5-Preview | [单台 256 GB Mac 上的原生 Metal：2-bit 专家、MTP 草稿、图像](step-5-preview.md) |
| MiniMax-M3.1-Flash | [单台 256 GB Mac 上的原生 Metal：affine 3/4-bit 专家、精确 bf16 缓存、DFlash2 草稿、图像](minimax-m3.1-flash.md) |
| Kolibri-1 | [原生 CUDA、block FP8、研究检查点](kolibri-1.md) |
| LFM2.5-24B-A2B | [原生 Metal、MLX 4-bit、共享轮次](lfm2.5-24b-a2b.md) |
| Spark-X2.6-4B | [原生 CUDA](spark-x2.6-4b-cuda.md)、[原生 Metal、bf16 发布版、上下文最高 1M](spark-x2.6-4b-metal.md) |
| Nanbeige4.3-33B | [原生 Metal、bf16 和 INT4 发布版、Llama 家族](nanbeige-4.3-33b-metal.md) |

[持久化服务状态身份](persisted-serving-keys.md)列出了磁盘数据的归属模块、兼容性键和陈旧状态主机检查。

## 能力下限

原生发布版包含面向 sm_80、sm_86、sm_120 和 sm_121 的 CUDA 内核。这是构建目标列表，并不代表逐显卡完成了模型资格认证。Nemotron MLX 4-bit 已在 GB10、RTX 3090 和 RTX PRO 6000 上检查。
A100（sm_80）有来自 #846 的[内核检查证据](../../CHANGELOG.md#1012)，但没有真实检查点服务运行记录，因此未通过模型资格认证。测量覆盖范围参见[已认证模型](../../README.md#qualified-models)。
`tensorfold info REPO` 打印检查点的内存下限：权重加 2 GiB 运行时余量。在 macOS 上，Flash Next（`qwen4_exp`）检查点改为打印启动预算和冷 pack 预算。

| 引擎 | 下限 |
| --- | --- |
| 原生 CUDA，NVFP4 / FP8 | Nemotron 和 dense Qwen 为 sm_80 至 sm_89 提供非 cluster 回退代码（#901、#1470）。没有在 sm_80 至 sm_89 上运行 NVFP4/FP8 模型的记录；不宣称 A100 通过模型资格认证。其他家族保留各自的验收门槛。 |
| Flash Next，原生 CUDA | MLX 4-bit 已在 GB10 上提供服务；ModelOpt/两个本地 rank 路径是实验性的。多 rank 草稿推理要求关闭提示词缓存。不代表每个发布版 SM 都通过资格认证。 |
| Intel GPU（XPU），Nemotron | 实验性。权重需要 17.3 GiB 设备内存（上下文从 4k 到 128k 时使用 18 至 19.4 GB）；测量设备为 Arc Pro B70（32 GB）。低于 24 GB 的显卡未经测试，16 GB 显卡无法容纳权重。 |
| Python 0.6.6 MLX，Apple Silicon | 家族页面中的历史范围，并非实测最低配置：GLM-5.3-Flash 面向 256 GB Mac 编写，常驻约 151 GiB。DeepSeek-V4-Flash 同样如此，常驻约 151 GiB。Flash Next 默认命令按 128 GiB M4 Max 设定。32 GB Mac 上的 Qwen3.8-27B 需要超过默认的 22.4 GiB。这些是机器档位，不是实测最低配置。 |

新家族应基于 Zig 引擎实现：[添加 Zig 家族](adding-a-zig-family.md)按顺序列出了步骤、各步收益及代码归属。对于冻结在 0.6.6 的 Python 引擎，贡献指南包括[添加 MLX 家族](adding-a-family.md)、[添加 CUDA 家族](adding-a-cuda-family.md)和 [CUDA 实现规则](cuda.md)。
[训练草稿模型](training-a-drafter.md)介绍如何将目标模型的 `--record` 输出转成经过训练的块草稿模型，并列出各步在 Kolibri-1 上的收益。
[EXL3 权重](exl3.md)和[通用 EXL3 专家](exl3-universal-experts.md)介绍了 @vcruz305 在 [#42](https://github.com/ashhart/TensorFold/pull/42) 中为 Python 引擎的 CUDA 家族编写的共享 EXL3 模块：支持任意码本、每张量任意位宽，每个 MoE 投影只进行一次分组启动。

## 契约

草稿输出必须与同一引擎的串行输出一致。恢复的提示词必须与该提示词全新运行的结果一致，每个并发流也必须与其单独运行的结果一致。该契约适用于相同权重、运行时和设置；其他后端或量化可能采用不同算术。

采样器以种子、绝对位置和 token ID 作为每次抽样的键。只有草稿匹配目标模型的该次抽样时才会接受。每个验证行都必须采用串行行的算术，包括注意力边界、路由平局处理和递归更新。回滚必须将已接受路径之后的所有状态恢复，包括草稿头状态。

提示词复用同样严格。MLX 规划器根据渲染后的 token 计算分块，只缓存兼容边界。当分词器提供相应标记时，恢复点为 assistant 消息的起始位置和第二条消息的起始位置。分块会跳过距离前一起点不足 256 个 token 的恢复点，否则在首个合格点或达到 2,048 个 token 时结束。没有可识别标记时，分块采用 2,048-token 网格。改写较早的模板文本可能使复用失效。
复用已存储前缀前，分块方案以及内核/运行时身份必须匹配。

CUDA 检查点加载参见[加载时预取](cuda-load-prefetch.md)。

## 测量

`tools/bench_openai.py` 包含公开测试样例：一个原始 Fibonacci 函数提示词，以及一个询问矩阵乘法如何使用 GPU 的聊天提示词。聊天样例关闭思考。以 `--name bench` 启动服务后，从仓库根目录运行：

```bash
python3 tools/bench_openai.py http://127.0.0.1:8080 bench \
  --tokens 64 --reps 5 --temperatures 1.0,0 --output bench.json
```

客户端为每个测试单元预热，使用种子 1234 至 1238，并报告首 token 之后解码速率的中位数。采样测试单元使用温度 1、top-k 20 和 top-p 0.95。客户端发送 `ignore_eos: true`；MLX、GLM CUDA、Qwen3.8-27B 和 Qwen3.6 CUDA 引擎会遵循该设置，而 Flash Next 和 Nemotron CUDA 引擎可能在达到请求上限前遇到 EOS 停止。
该工具测量吞吐量，本身不能证明 token 一致。结果应记录检查点和分词器修订版本、运行时版本、后端、rank 数量、启动命令和输出哈希。另行比较串行输出与草稿输出。

对于并发 MLX 工作负载，使用 `tools/bench_concurrent.py` 并添加 `--alone --serial`，将每个并发回复与其单独运行比较，再将这些单独运行与 `draft: false` 比较。长上下文和内存测量还需要公开提示词样例及其渲染后精确 token 数。分别记录冷启动、恢复和重启后的请求。

0.3.5 的所有解码、prefill、并发和峰值内存结果尚未确定 [release-0.3.5]。CUDA 家族页面中更早的公开样例结果属于历史测量，不是发布资格认证。

## 内核检查

对于承诺机器码不变的 CUDA 重构，使用[五 SM SASS 一致性运行器](cuda-sass-identity.md)。

串行与多行内核、以及已提交缓存之间的比较使用精确相等。模型质量检查使用独立可信的前向计算；与自己的串行实现一致不能证明模型保真度。包含真实 head 维度、稀疏注意力边界、部分保留和不同长度的并发流。修改内核或运行时后，重新生成串行参考结果。

测量相互依赖的操作和完整请求。独立微基准可能隐藏启动成本、权重缓存效应，以及与其他工作重叠机会的损失。
