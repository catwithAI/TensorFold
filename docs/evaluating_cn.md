# 评估 TensorFold

本页面向从项目外部审查、测试 TensorFold 或开展基准测试的人。
内容包括需要固定哪些信息、如何检查下载文件、哪些测试无需 GPU，以及哪些请求值得重放。

## 固定测试对象

每份结果都记录以下四项：

- 发布版本，来自 `tensorfold --version`（Linux 上为 `tensorfold-native --version`）。
- 检查点及其修订版本，在 `tensorfold pull` 中表示为 `REPO@revision`。
- 后端：Mac 上的 Metal，或 Linux、Windows 上的 CUDA。
- 从[选择模型](../README.md#pick-a-model)中采用的配方及其参数。

从 1.0.0 开始的发布版本使用原生 Zig 引擎。Python 引擎停在 0.6.6，仅保留于 `v0.6.6` 标签；它的测试需要 MLX，结果不能代表 1.x 发布版本。在仓库根目录运行 `pip install .` 会停止，并提示从哪里获取原生二进制文件。

## 安装并检查下载文件

在 Mac 上，`brew install ashhart/tensorfold/tensorfold` 会依据 formula 中的校验和检查归档。

在 Linux 上，[快速开始](../README.md#linux-with-an-nvidia-gpu)会下载归档并检查其 `.sha256`。`.sha256` 文件本身也使用发布密钥签名。要进一步检查该签名，可使用以下命令（需要 OpenSSL 3.0 或更新版本）：

```sh
A="tensorfold-$V-$PLATFORM.tar.gz"
curl -fLO "https://github.com/ashhart/TensorFold/releases/download/v$V/$A.sha256.sig"
curl -fsSL https://raw.githubusercontent.com/ashhart/TensorFold/main/packaging/runtime-trust.json \
  | python3 -c 'import json, sys; print(json.load(sys.stdin)["ed25519PublicKey"])' > tensorfold-release.pem
{ printf 'tensorfold-native-checksum-v1\n'; cat "$A.sha256"; } > signed-checksum
openssl pkeyutl -verify -pubin -inkey tensorfold-release.pem -rawin -in signed-checksum -sigfile "$A.sha256.sig"
```

OpenSSL 会打印 `Signature Verified Successfully`。解压后，在归档目录内运行 `sha256sum -c SHA256SUMS`，可检查其中每个文件。

## 无需模型的冒烟检查

以下检查无需 GPU、驱动或检查点：

- `tensorfold --version` 打印发布版本。
- `tensorfold capabilities --json` 列出后端、模型家族、格式、服务参数及 `optional_libraries`，每个可选库均带有 `found` 标记。
- `tensorfold info REPO` 打印已拉取检查点的家族、格式、上下文、权重大小和内存下限。

## 容器

根目录的 [Dockerfile](../Dockerfile) 将 Linux 发布归档安装到 Ubuntu 24.04 镜像中。只有归档校验和、该校验和的发布签名以及所有解压文件的校验和全部匹配，构建才会继续。
容器以镜像中的 `ubuntu` 用户运行，并将 Hugging Face 缓存保存在应挂载的 `/huggingface` 目录。

```sh
docker build -t tensorfold .
docker run --rm tensorfold
docker run --rm tensorfold capabilities --json
```

`--build-arg TENSORFOLD_VERSION=1.2.4` 用于选择发布版本；Docker 根据构建平台选择 `linux-x86_64` 或 `linux-aarch64` 归档。提供推理服务时，宿主机需要 NVIDIA 驱动 R580 或更新版本，以及向容器提供 `libcuda.so.1` 的 NVIDIA Container Toolkit。在容器内监听所有地址：

```sh
docker run --rm --gpus all -p 8080:8080 \
  -v "$HOME/.cache/huggingface:/huggingface" \
  tensorfold serve mlx-community/Qwen3.5-2B-MLX-4bit --host 0.0.0.0 --name local-model --context 8192
```

如果你的用户 ID 不是 1000，添加 `--user "$(id -u):$(id -g)"`，使服务能够写入挂载的缓存。镜像仅包含发布归档。需要 cuBLASLt、NCCL 或图像解码器的功能（参见[可选共享库](../packaging/RUNTIME.md#optional-shared-libraries)），需要使用额外安装了这些库的镜像。

## 无需 GPU 的测试

- 在仓库根目录运行 `python3 -m pytest` 只需要 Python 3 和 pytest。依赖 GPU、numpy 或已构建测试二进制文件的测试会跳过，并说明原因。
- `zig build test` 使用 `.zig-version` 中的 Zig 版本构建并运行主机测试。在 Mac 上还会运行可执行的 Metal 设备检查。
- `zig build test-golden` 检查冻结的等价性答案和 HTTP 安全用例。

[CI](../.github/workflows/no-gpu.yml) 对每个拉取请求在 Linux 和 Windows 上运行这些测试，其中根目录 pytest 在 Linux 上运行。GPU 内核和真实模型服务检查会在改动落地前于维护者的 Mac 和 NVIDIA 机器上执行；[CONTRIBUTING.md](../CONTRIBUTING.md) 列出了这些验收门槛。

## 自行检查精确一致性

草稿推理绝不能改变输出。在温度为 0 时发送两次相同请求，其中一次带 `"draft": false`，并比较文本和 `usage`。利用提示词缓存重复发送的请求必须与首次运行一致，并发请求也必须分别与各自单独运行的结果一致。[精确解码](../README.md#exact-decoding)说明了这一规则及其解决的问题。

## 值得重放的请求

| 请求 | 预期结果 |
| --- | --- |
| 提示词长度超过上下文窗口 | 返回 HTTP 400，`code` 为 `context_length_exceeded`；不消耗 GPU 工作。 |
| `tool_choice: "required"` 或指定名称的函数 | 回复会调用工具。需要归档中的 `lib/`；缺少时返回 HTTP 400。 |
| `thinking_budget` 或 `reasoning_effort` | 推理在预算内停止；`reasoning_effort: none` 关闭思考。 |
| 带 JSON schema 的 `response_format` | 回复符合 schema，且与 `"draft": false` 的回复相同。同样需要 `lib/`。 |

[API 参考](api.md)列出了所有字段和拒绝情形。

## 内存

`tensorfold info REPO` 打印检查点的内存下限：权重加上服务保留的余量。上下文长度、`--parallel`、提示词缓存和草稿模型都需要额外内存。模型加载后，`GET /v1/memory` 报告已使用的内存：macOS 上为进程占用，其他情况使用后端设备计数器（参见[内存诊断](memory-diagnostics.md)）。
[选择模型](../README.md#pick-a-model)列出各模型接受检查时所用的机器；未测量最低配置时，不会发布最低配置。

## 依赖

提供推理服务无需安装 Python 包，因此 `pip-audit` 没有可扫描的服务端依赖。
`build.zig.zon` 未声明 Zig 包，因此 Zig 代码仅依赖本仓库即可构建；内核需要 Xcode 的 Metal 编译器或 CUDA 13 的 `nvcc`。Linux 归档另附 `libtfgrammar`（xgrammar v0.2.8）。所有随包附带的许可证均见 [THIRD_PARTY_NOTICES.md](../THIRD_PARTY_NOTICES.md)。
某些仓库工具，例如检查点转换器和参考结果生成器，会使用 numpy、PyTorch、MLX 或 Transformers。构建、运行主机测试或提供服务均不需要它们，根目录 pytest 会跳过依赖它们的测试。`app/` 下的 Mac 应用有自己的依赖。

## 报告结果

使用 issue 模板报告 bug 和性能结果，并附上前述四项固定信息。
按照 [SECURITY.md](../SECURITY.md) 的说明私下报告安全问题。
