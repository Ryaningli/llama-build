# llama.cpp Legacy Build

为 [llama.cpp](https://github.com/ggml-org/llama.cpp) 构建兼容老版本 glibc 的二进制产物。

## 为什么需要

llama.cpp 官方 release 在 ubuntu-24.04 上构建，链接 glibc 2.39，无法在 glibc 2.31 的系统上运行。本项目通过 Docker 容器（ubuntu:20.04）编译，产物兼容 glibc >= 2.31 的 Linux 系统。

## 使用方法

1. 关注上游 Release 页面：https://github.com/ggml-org/llama.cpp/releases
2. 进入本仓库的 **Actions** 页面，选择 **Build Legacy**
3. 点击 **Run workflow**，填写参数：
   - **upstream_tag**：上游的 release tag，如 `b4050`（必填）
   - **targets**：构建目标，默认 `ubuntu-arm64,ubuntu-x64`

### 可用 targets

| target | 架构 | glibc 要求 |
|--------|------|-----------|
| `ubuntu-arm64` | ARM64 (aarch64) | >= 2.31 |
| `ubuntu-x64` | x86_64 | >= 2.31 |

### 示例

```
upstream_tag:  b4050
targets:      ubuntu-arm64,ubuntu-x64
```

## 下载产物

构建完成后会自动创建 Release，直接从 Release 页面下载 `llama-b*.tar.gz`，解压即用：

```bash
tar -xzvf llama-b4050-bin-ubuntu-arm64.tar.gz
cd llama-b4050
./llama-server -m /path/to/model.gguf
```

也可以在 **Actions** 页面的 **Artifacts** 区域下载。

## 与官方 release 的区别

| | 官方 release | 本项目 |
|--|-------------|--------|
| 构建环境 | ubuntu-24.04 | ubuntu:20.04 (Docker) |
| glibc 要求 | >= 2.39 | >= 2.31 |
| GCC | 14 | 9 |
| 源码版本 | 与 tag 一致 | 与 tag 一致 |
| 后端动态加载 | 支持 | 支持 |
| Release | GitHub Release | 自动创建 Release |

## 工作原理

1. 从上游仓库 `ggml-org/llama.cpp` 拉取指定 tag 的源码
2. 在 ubuntu:20.04 Docker 容器中原生编译（ARM runner 用 ARM 容器，x86 runner 用 x86 容器）
3. 使用 `$ORIGIN` RPATH，共享库从二进制所在目录加载，无需设置 `LD_LIBRARY_PATH`
4. 打包并上传 artifact
5. 自动创建 GitHub Release

## 添加新的构建 target

在 `.github/workflows/build-legacy.yml` 的 matrix 中添加条目即可：

```yaml
- target: ubuntu-x86
  os: ubuntu-latest
  container: ubuntu:18.04    # glibc 2.27
  arch: x86
```
