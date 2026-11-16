# TensorFold 中文贡献者入门

这是一份面向新贡献者的阅读导航和工作流摘要。它帮助你快速理解项目，但不能替代原始文档；提交代码时以英文原文、当前分支和 CI 结果为准。

## 先记住四件事

1. TensorFold 的主线是 Zig 原生推理引擎；Python 0.6.6 已冻结，不再接受主线功能修改。
2. 引擎直接读取模型 checkpoint，并在 Apple Silicon 上使用 Metal、在 NVIDIA 上使用 CUDA；服务模型不需要 Python 或 MLX。
3. 改代码前必须通过 Agent Traffic Control（ATC）检查工作是否已被其他贡献者占用。
4. 推理相关改动首先保证 token 级一致性，然后才讨论速度、显存和吞吐量。

## 必读文档

请按下面的顺序阅读：

| 顺序 | 文档 | 你需要掌握的内容 |
| --- | --- | --- |
| 1 | [`README.md`](README.md) | 项目定位、安装和运行方式、支持的模型、API、构建入口 |
| 2 | [`CONTRIBUTING.md`](CONTRIBUTING.md) | 分支和 PR 规则、测试、精度、性能收据、代码和隐私规范 |
| 3 | [`docs/atc/SKILL.md`](docs/atc/SKILL.md) | 如何查看 board、检查 issue、认领工作、报告 session、检查冲突 |
| 4 | [`docs/recipes/SKILL.md`](docs/recipes/SKILL.md) | 原生引擎、模型族、GPU kernel 和性能改动的完整流程 |
| 5 | [`docs/pr/SKILL.md`](docs/pr/SKILL.md) | 创建或更新 PR 前的 pre-flight、receipt、lean 检查和平台覆盖 |

按贡献方向继续阅读：

- 桌面应用：[`app/README.md`](app/README.md)
- Web 控制台：[`web/README.md`](web/README.md)
- API 行为：[`docs/api.md`](docs/api.md)
- 模型和硬件配方：[`docs/recipes/README.md`](docs/recipes/README.md)
- GPU kernel 合约：[`docs/kernel-contract.md`](docs/kernel-contract.md)
- 性能测量：[`docs/optimizing.md`](docs/optimizing.md)

## 项目结构

```text
zig/       Zig 原生引擎、服务器、模型族、Metal/CUDA/HIP 运行时和测试
app/       Electron + React 桌面应用，直接管理本地引擎和 Agent 工作区
web/       FastAPI + React Web 控制台，用于模型管理、聊天和基准测试
tools/     benchmark、验证、发布和开发辅助脚本
docs/      API、模型配方、性能收据、设计说明和贡献流程
packaging/ 发布包、运行时库和平台分发逻辑
assets/    品牌和发布所需的静态资源
```

原生请求的大致路径是：

```text
HTTP/OpenAI 请求
  → zig/src/server
  → tokenizer、chat template、sampling
  → zig/src/core 的 engine_api、lane host、KV cache、prompt cache
  → zig/src/families 中对应的模型族
  → zig/src/metal 或 zig/src/cuda
  → GPU 执行并流式返回结果
```

建议从这些源码入口开始：

- [`zig/src/server/main.zig`](zig/src/server/main.zig)：`tensorfold-native` 的 CLI 和启动流程。
- [`zig/src/server/serve.zig`](zig/src/server/serve.zig)：HTTP 服务和请求生命周期。
- [`zig/src/core/engine_api.zig`](zig/src/core/engine_api.zig)：服务器调用引擎的共享接口。
- [`zig/src/core/lane_host.zig`](zig/src/core/lane_host.zig)：并发 stream、prefill、decode 和状态切换。
- [`zig/src/families/`](zig/src/families/)：不同模型族的 checkpoint、前向计算和后端实现。
- [`zig/src/metal/`](zig/src/metal/) 与 [`zig/src/cuda/`](zig/src/cuda/)：平台运行时和 kernel 调度。

## 开始贡献前

项目当前要求使用 `.zig-version` 中锁定的 Zig 版本；当前版本是 0.17.0。macOS 原生构建还需要完整 Xcode 和 Metal Toolchain。

先安装或确认 ATC 0.9.4 或更高版本，然后在更新后的仓库中运行：

```sh
atc board --json
atc check ISSUE --json
```

只有当检查结果明确显示工作是 `free` 时才可以认领：

```sh
atc claim ISSUE --json
```

如果工作已经属于自己，使用：

```sh
atc start ISSUE --json
```

工作期间按需检查文件重叠，并在停止工作时结束 session：

```sh
atc who path/to/file --json
atc overlap --json
atc pulse ISSUE SESSION working --json
atc land ISSUE SESSION --json
```

不要在已有贡献者占用 issue 时直接实现同一个方案；可以提出 pair、split、takeover 或 review。

## 常用构建和测试

Apple Silicon 上的原生引擎：

```sh
zig build native -Dcpu=apple_m1
zig build test test-golden -Dcpu=apple_m1
python3 tools/zig/lean_check.py
```

生成的服务端位于 `zig-out/native/bin/tensorfold-native`。如果修改了开发工具调用的 API，还要运行：

```sh
zig build dev-tools
```

桌面应用：

```sh
cd app
npm ci
npm run deps:trusted-scripts
npm test
npm run typecheck
npm run verify
```

Web 控制台：

```sh
cd web
uv venv .venv
. .venv/bin/activate
uv pip install -e '.[dev]'
pytest
ruff check .
cd ui && npm install && npm test
```

## 代码正确性要求

推理相关改动必须验证：

- draft 输出与 `"draft": false` 输出逐 token 相同。
- 缓存恢复路径与新鲜执行结果相同。
- 并发请求与单独运行结果相同。
- prompt、decode、KV、最终 logits 在相关边界上保持要求的精度。
- cancellation、reset 和 partial acceptance 不会污染下一次请求。

GPU kernel 或算术改动还需要对应的 kernel contract、参考实现、平台覆盖和性能 receipt。一个 host build 不能代替真实 GPU、checkpoint 和 runtime 的资格验证。

## 代码风格和 PR 规则

- 从 `main` 开始；一个 PR 只处理一个行为。
- 新模型族、新后端或算术改动要先开 issue 并确认 gate。
- 模块保持单一职责；源文件通常不超过 600 行。
- 注释和 docstring 使用单行短注释，最长 120 列。
- 新测试应证明真实失败和修复结果，不能只重复实现逻辑。
- 提交 PR 前运行 `atc overlap --json`，解决所有真正的文本冲突。
- 运行 `zig fmt --check build.zig zig`、`git diff --check` 和适用的 host/device/app/Web 测试。
- 使用 `python3 tools/pr_check.py --base origin/main --pr NUMBER` 做 PR 检查。
- PR 描述必须写清 issue、测试结果、base/head commit、平台覆盖和未运行的平台。

## 适合第一次贡献的方向

推荐从可本地验证、硬件依赖较少的工作开始：

1. 文档、API 示例和运行指南修正。
2. Web 控制台或桌面应用的小功能和测试。
3. 不依赖 GPU 的 Zig host test。
4. 可复现的 bug 报告、benchmark 或性能 receipt。

熟悉请求生命周期、lane host 和模型族之后，再进入 KV cache、调度器、模型实现或 GPU kernel。性能改动必须同时证明结果没有变化，并给出同机器、同 checkpoint、同参数的前后测量。

## 推荐学习路径

```text
读 README 和 CONTRIBUTING
  → 安装 Zig 与 ATC
  → 跑通 host tests
  → 读 server/main.zig 和 engine_api.zig
  → 选择 App、Web、文档/测试或原生引擎路线
  → 检查并认领一个空闲 issue
  → 做一个小而完整的 PR
```

遇到不确定的行为时，先查现有 test、recipe、receipt 和 issue，再决定是否改代码。TensorFold 的 review 重点是可复现的证据，而不是只看“代码能编译”。
