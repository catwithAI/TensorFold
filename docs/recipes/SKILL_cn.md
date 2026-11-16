---
name: tensorfold-contributor
description: Implement or review TensorFold native-engine contributions using the repository's model recipes, kernel contracts, exactness checks and reproducible performance gates. Use for model-family ports, GPU operators, weight loaders, native serving and their tests.
---

# TensorFold 贡献者

产出规模小、便于审查的原生贡献，并为每项受影响的契约提供证据。
本技能定义贡献必须满足的条件；不构成合并、发布或硬件资格认证的批准。

编辑前，遵循 [ATC 贡献者说明](../atc/SKILL_cn.md)，查看公开雷达并检查或认领 issue。
在获准报告此项工作时，使用 `atc start <issue> --public https://tensorfold.dev --json` 发布匿名会话；只有用户明确选择时才显示 GitHub 名称。
仅在工作期间发送心跳，打开拉取请求前检查重叠，会话停止时结束会话。

## 确定贡献契约

编辑前先找到 TensorFold 检出目录及其 Git 根目录。
原生检出目录包含 `build.zig`、`.zig-version`、`zig/src/native/` 和 `docs/recipes/`。
阅读涉及目录内的所有 AGENTS.md（应用有自己的说明）、[CONTRIBUTING.md](../../CONTRIBUTING.md)、[RUNBOOK.md](../../RUNBOOK.md)和[配方索引](README_cn.md)。
GPU 改动还须在选择实现前完整阅读[内核契约](../kernel-contract.md)。
以唯一分支 `main` 为目标，保留无关改动；其他本地 TensorFold 检出目录可能是冻结的 Python 引擎。

实现前，在工作笔记或 PR 草稿中写下简短契约：

- 所需行为、复现方式和现有失败，或新增的缺失能力。
- 目标提交、家族、后端、检查点修订版本、量化、精度模式、GPU 档位和 rank 数量。
- 负责该功能的模块，以及需要扩展的现有共享算子或家族适配器。
- 适用的正确性检查、性能指标和已商定的接受门槛，包括冷 prefill 要求。
- 可用硬件、无法在其上运行的必需检查，以及需要其他测试者提供的证据。

新家族、后端和算术改动需要 CONTRIBUTING.md 中所述、与维护者商定的门槛。
如果缺少该门槛，应在宣称工作可落地前指出尚未决定的事项。
不得悄悄降低门槛、改变工作负载，或用更容易的测试替代无法执行的测试。
当前 `zig/gate.json` 可能没有条目；这不意味着可以自行设定覆盖整个发布版的速度目标。
创建 issue、发送 PR、上传结果或修改共享机器仍需用户授权。
每个拉取请求只承担一项工作；依赖关系以堆叠组织，说明各自基于什么，并保持堆叠简短。
维护者会在落地时变基，因此不要仅为跟上 main 而对审查中的分支强制推送。

## 阅读正确配方及其描述的代码

原生初步适配和优化阶段使用[添加 Zig 家族](adding-a-zig-family.md)。
从配方索引阅读受影响家族的页面，再追踪实际原生加载器、后端注册、算子和测试。
涉及检查点算术时还要阅读[量化](../quantization.md)；涉及传输改动时阅读[加速设置](../speed-up-mode.md)。

区分原生说明与明确标为 Python 0.6.6、历史测量或未经证实的后续步骤的章节。
Python 引擎已冻结；`tools/` 下的 Python 客户端、测试样例构建器和可信参考实现是开发工具，不是原生推理依赖。
不要将仅适用于 Python 的参数、环境变量、传输方式或支持表复制到原生移植中。
无关 bug 修复或文档改动不必执行每个优化阶段。

某些家族页面的介绍落后于较新的原生代码。
检查目标构建的 `--version`、`--help` 和 `capabilities --json`、原生注册表与加载器，以及匹配的设备/模型记录。
家族已注册或源码能编译，既不能证明整个检查点受支持，也不能证明每种 GPU 都已通过资格认证。
文档与实现不符时，记录差异和建议修正；不要将此视为放宽精度或精确一致性要求的许可。

## 保持原生架构的职责归属

| 职责 | 检查并扩展 |
| --- | --- |
| 请求生命周期、事件和对外声明的能力 | [引擎 API](../../zig/src/core/engine_api.zig)和[服务适配器](../../zig/src/server/engines.zig) |
| 共享调度、目标验证、采样和每流状态 | [lane 核心](../../zig/src/core/lanes/) |
| 后端选择、准入和家族注册 | [原生适配器](../../zig/src/native/)，包括相应 Metal、CUDA、HIP 或 XPU 适配器 |
| 家族配置、张量映射、层图和家族专用混合器 | [模型家族](../../zig/src/families/) |
| 可复用投影、布局、缓存和后端算子 | [共享核心](../../zig/src/core/)、[CUDA 算子](../../zig/src/cuda/)和 [GPU 内核](../../zig/kernels/) |
| 构建注册和可部署资源 | [build.zig](../../build.zig)、[构建模块](../../zig/build/)和[打包](../../packaging/README.md) |

让家族适配现有引擎接口；不要另加并行调度器，也不要复制共享验证/缓存机制。
不支持的家族属于初步适配任务，不能以此为由将生产推理转交 Python、CPU 替代实现或仅串行的变通方案。
普通解码始终是正确性判据；已宣称支持的草稿路径不应依赖关闭草稿才能工作。
并发请求通过 lane 核心共享轮次，家族允许的每个流都应启用草稿；在某个宽度或流数下关闭草稿或回退串行是 bug，不是应记录为限制的正常行为。
测试原生服务实际调用的路径，包括能力报告和有意拒绝的行为。
配置声明了 vision tower 的检查点，只有通过引擎自身内核支持图像输入，家族才算完整；在此之前，应明确拒绝图像部分和 `--vision`，并测试该拒绝。

保持存储的权重和分词器资源不变。
尊重每个张量实际的位宽、组大小、步长、缩放、偏置约定和格式元数据；混合投影不能共用不兼容的融合内核。
显式拒绝不支持的布局和格式错误的几何形状，并测试这些拒绝。
将临时空间和预处理 pack 放在加载器拥有的缓存中，遵循其现有身份标识和原子发布规则。

## 注册并证明 GPU 内核

新增或修改的 GPU 算子必须能通过所属后端的契约运行器调用，不能只有独立单元测试。
注册类型为 [Spec](../../zig/src/core/contract/contract.zig) 和 [Kernel/VTable](../../zig/src/core/contract/gates.zig)。
当公式与算子匹配时，使用共享 affine、dense 或 rows 参考辅助函数。
声明实际服务形状、tile 边缘、输出类型、累加和舍入深度、NaN 行为及允许的别名。
保持位级一致的替代实现须声明 `replaces`，并提供在相同输入上运行的 `replaced` 钩子；其契约要求逐字节一致。
如果预期的归约重排改变了字节，不再将其视为这种可直接替换的实现，并在落地前与维护者解决算术/比较契约。
单纯改善 FP64 误差既不能豁免替换一致性，也不能授权新增串行基准，更不能通过重命名内核规避门槛。
根据需要将算子注册到 [Metal 运行器](../../zig/tests/contract/metal_main.zig)、[CUDA 运行器](../../zig/tests/cuda/contract_tests.zig)或 [HIP 运行器](../../zig/tests/hip/contract_main.zig)。
在 [registry.zig](../../zig/src/core/contract/registry.zig) 中单独注册启动限制与编译记录；HIP 使用 [hip_records.zig](../../zig/src/core/contract/hip_records.zig)。

| 验收门槛 | 必需证据 |
| --- | --- |
| 非有限输入 | 每种输出类别均符合 FP64 公式，或符合算子显式测试过的 NaN 规则。 |
| 重复与别名 | 五次运行逐字节一致；输入和保护区不变；不存在未声明的输出/输入重叠。 |
| 形状 | 服务维度和有效 tile 边缘满足推导出的参考误差界限，或替代输出与被替换内核逐字节相同。 |
| 累加 | 压力测试样例可区分低精度求和，并证明采用了要求的累加器；报告已声明限制或确实不适用的情况。 |
| 启动限制 | 更新的编译记录和设备检查覆盖相关服务芯片、线程、寄存器、共享内存和 cluster。 |

将 [Metal 契约测试](../../zig/tests/contract/)和 [CUDA 契约测试](../../zig/tests/cuda/contract_tests.zig)作为具体集成示例。
每个植入对照都必须在目标门槛处失败；对照出错、跳过或意外通过都不能证明任何事情。
附上后端完整的 `SUMMARY`、退出状态、已知问题和未运行芯片。
诊断用的 `--only`、`--no-gates` 或 `--no-limits` 运行不构成完整资格认证记录。
不要添加 `known` 条目掩盖新回归；现有已知问题必须保持可复现，直到修复将该条目移除。
Metal、CUDA 或 HIP 运行器的覆盖不能证明其他后端的正确性，也不能证明 XPU；应列出缺失后端覆盖，并另行商定其检查。

保持检查点的激活精度和要求的累加精度。
不要仅为通过速度门槛而添加 FP8/int8 激活、有损缓存、低精度求和或 fast-math 改动。
由检查点定义的算术是另一种情况：[Nemotron Super VL](nemotron-3.5-super-vl.md)显式定义了 `checkpoint` 和 `full` 模式。
每个声称支持的模式都应对照其自身普通输出完成认证，在记录中标明模式，并单独测量模式间的保真度。
不要把旧配方中笼统的精度声明解释成对已认证检查点模式的禁令。

归约顺序改变时，还须为每种受影响投影类别提供独立 FP64 参考。
原生误差不得劣于当前参考。
分别比较提示词行和解码行；保留最差列误差，以及教师强制历史下最高 token 的差异和分差。
同引擎草稿/普通输出一致不能替代这份独立的模型保真度证据。
不得扩大容差，或超出文档规定的候选及分差范围应用平局策略，以人为制造通过结果。

### 保持设备顺序和缓冲区生命周期

除了 shader，还要追踪启动器。
对于生成或捕获的内核，更新其所属源码或生成器，并重新生成匹配资源和清单；只修补某个生成副本不会更新运行时路径。
[Metal lane 投影](../../zig/kernels/metal/core/lane_projection.metal)在共享权重读取时保留各行求和；[CUDA lane GEMV](../../zig/kernels/cuda/lane_gemv.cu)保留 group 和 K-slice 顺序。
在所选契约要求的所有场合，保持每行算术独立于验证宽度、分组、提示词分块和并发流顺序。
检查真实维度、尾部、稀疏注意力边界和已接受路径的状态提交，不能只检查方便的完整 tile。

GPU 资源、借用视图和每次调用的元数据必须在命令完成前保持有效。
修改命令缓冲区或队列时，保留驻留声明和现有 Metal fence/event 顺序。
CUDA 上，在消费数据的流中上传，并保留 graph 捕获/重放的所有权和稳定缓冲区。
未测量依赖操作及真实服务成本前，不要向热路径添加主机回读或同步。
芯片专用快速路径必须为贡献所宣称支持的旧芯片保留有效、受能力检查控制的路径。

## 证明引擎契约

在相同检查点、后端、运行时、rank 数量、精度和请求设置下，检查每个受影响用例：

- 在支持的窗口/深度下，草稿输出与 `"draft": false` 逐 token 一致。
- 支持的恢复、编辑、重新生成和重启后路径，与完整渲染提示词的全新运行一致。
- 并发回复与各自单独运行一致，包括对请求排队的后端。
- 提示词/解码衔接处保留所需 KV、递归、卷积、n-gram、草稿模型和最终 logit 状态。
- 拒绝、部分接受、取消和重置之后，下个请求仍有效。
- 对外声明的思考、采样、工具、结构化输出和媒体功能触达预期原生路径，并保留现有 API 行为。

只宣称实际执行过的支持组合；不要假定不同后端、量化或精度模式等价。
冷检查以及易受未初始化状态影响的衔接检查，使用全新进程。
保留改动前后的失败，以及 token 哈希或完整 token 数组，而不只是看似合理的文本。
缓存改动须验证身份/失效、缓存 token 计数和真正关闭缓存的对照。
多主机改动须依据受影响传输的配方，纳入匹配的构建/检查点/精度检查，以及对端失败/清理。

## 按改动选择检查

使用 `.zig-version` 中的精确 Zig 版本；不要通过升级工具链走移植捷径。
原生代码的基准主机检查见[无 GPU CI](../../.github/workflows/no-gpu.yml)：

```sh
zig build test test-kernel-contract test-golden fuzz-cortheon
git diff --check
python3 tools/zig/lean_check.py
```

使用 [lean_new.py](../../tools/zig/lean_new.py) 和 CI 流程，以同一检查器与 PR 合并基准比较。
必须报告继承的 lean 问题；接受条件是零新增问题，不能虚称干净退出。
源码模块不超过 600 行，单行注释/docstring 不超过 120 列；测量数据放在记录或配方中。
根据实际构建目标及受影响配方运行额外家族/算子测试。
CI 无条件运行 `fuzz-cortheon`；聚焦局部的本地迭代不等于完整无 GPU 记录。
还应运行不带目标的 `zig build`，因为命名步骤会跳过部分工具，只有普通构建才会编译所有目标。
资源或归档改变时使用现有打包检查。

Mac 构建需要 Xcode 的 Metal 工具链（`xcrun metal`）；仅有 Command Line Tools 无法构建 metallib。
在 Mac 上，打算跨支持的 Apple 芯片运行的原生构建使用：

```sh
zig build native -Dcpu=apple_m1
zig build kernel-contract -Dcpu=apple_m1
```

结构化输出和必需工具调用需要发布归档中的 `lib/`，或运行 `zig build grammar -Dxgrammar=DIR`；使用 `git clone --branch v0.2.8 --recurse-submodules https://github.com/mlc-ai/xgrammar` 准备 DIR（grammar 使用与原生构建相同的 `--prefix`）。

第二条命令会运行 GPU 门槛检查，要求测试设备可用。
编译限制记录使用内核契约指南中 Metal 契约的 `--offline DIR` 工作流。
在获准使用的 CUDA 测试机器上，必要时调整合格编译器路径：

```sh
zig build kernel-contract -Dnvcc=/usr/local/cuda/bin/nvcc
zig build fatbins kernel-contract-fatbins -Dnvcc=/usr/local/cuda/bin/nvcc -Dsm=80,86,89,120,121
zig-out/bin/tf-cuda-test kernel-contract-limits zig-out/fatbin
```

在 AMD 测试机器上，针对指定芯片的 GPU，以及无需 GPU 的每种服务 RDNA 芯片记录，运行：

```sh
zig build hip-kernel-contract -Dhip-arch=gfx1100 -Dhipcc=/opt/rocm/bin/hipcc
zig build hip-kernel-contract-limits -Dhipcc=/opt/rocm/bin/hipcc
```

运行平台命令前，检查[内核契约构建注册](../../zig/build/kernel_contract.zig)（[HIP 注册](../../zig/build/hip_contract.zig)）及目标构建选项。
主机测试、编译限制记录、仅主机归档和伪引擎 golden 语料不能证明 GPU 上的真实模型推理。
仅文档改动需要链接/声明检查，不需要编造模型记录或新增 GPU 工作负载。

## 附上可复现性能记录

使用原生服务、issue 中商定的工作负载，以及现有[服务](../../tools/bench_openai.py)、[并发](../../tools/bench_concurrent.py)和[冷缓存](../../tools/bench_cache.py)客户端。
匹配机器、基准/候选二进制身份、检查点/分词器/草稿模型修订版本、精确渲染提示词 ID、输出长度、种子和生成设置。
记录思考模式、精度、rank 数量、缓存状态、并发、预热和重复次数。
测量普通贪心自然语言及代码；改动路径需要时，还要测量采样或功能专用测试单元。
解读速度前，先检查该次运行的 token。

分别报告解码速率、冷 prefill/TTFT 和并发总吞吐。
提示词处理须保留允许长度下的冷 2k/8k/32k/64k token 数组和请求体，更长的模型原生窗口单独标注。
冷测量保留 `--prompt-cache-gib 0` 对照和全新 KV 状态；热前缀不算冷 prefill 改善。
对于小差异，交替运行基准/候选，保留所有尝试并解释中断或漂移。
不要将输出 token 或量化变化称为独立内核优化。
独立微基准补充相互依赖的操作和完整服务请求，但不能替代它们。

包含峰值物理进程占用，以及可获得的设备/固定页内存分配，还要注明实际 RAM 档位和上下文。
在更大机器上配置预算，不能证明能装进更小机器。
使用现有 GPU 锁和准入保护，每台测试机器只运行一个模型进程。
不要仅为完成记录而停止别人的服务、提高共享内存限制、改变驱动或对外开放监听。
除非明确获准上传，否则记录保留在本地。

## 不改变门槛地判断就绪状态

遵循现有 [PR 模板](../../.github/pull_request_template.md)。
说明改变的行为、issue/门槛、所属模块、命令、退出状态、通过/失败/跳过计数、精确一致性结果、内核摘要、前后速率、冷 prefill 影响和未运行平台。
相关配方或能力文档的更新范围不得超过新证据支持的范围。
为 TensorFold 编写代码，不要从其他推理引擎或项目复制源码、内核或表格。
如果确实需要外部内容，在 PR 中注明来源和许可证，并等待维护者决定。
保留贡献者署名、必需许可证和第三方声明；源码、样例和公开记录不得包含私有路径、主机、凭据和机密数据。
以贡献者的 GitHub noreply 地址提交，不添加 AI 工具共同作者或“Generated with”行；包含这些内容的 PR 会通过新提交落地，以 Co-authored-by 署名，并显示为已关闭而非已合并。

只有所需检查和已商定的性能门槛都满足，贡献才在其声称范围内就绪。
略低于最低要求仍然是不达标；跳过或设备不可用不等于通过。
未经资格认证的支持可以作为草稿或受限实验展示，并明确剩余门槛。
报告阻碍及最小下一步检查，不要宣称完成或削弱测试。
不要因为使用本技能而顺带合并、发布、分配发布版本或部署。
