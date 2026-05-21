# Ollama Docker 快速部署与 API 使用教程（含 GPU 支持）

本教程详细介绍如何在宿主机上使用 Docker 部署 Ollama，并通过 API 调用模型，无需每次进入容器即可使用 GPU 加速。适合进行大语言模型（如 `qwen3:4b`）测试和 SOC/SOH 等科研任务。

---

## 1. 准备工作

确保宿主机满足以下条件：

* 安装 **Docker**（版本 >= 20）
* 安装 **NVIDIA GPU 驱动** 并配置 **nvidia-docker2** 或 Docker 支持 GPU
* 至少一块 **NVIDIA 显卡**（推荐 8GB 显存以上）
* 网络连通性良好（用于拉取 Docker 镜像）

---

## 2. 拉取并运行 Ollama Docker 容器

```bash
docker run -d \
  --gpus all \                   # 启用 GPU
  -p 11434:11434 \               # 映射 API 端口
  -v ollama:/root/.ollama \      # 持久化模型数据到宿主机
  --name ollama \                # 容器命名
  ollama/ollama                  # 镜像名称
```

> 说明：
>
> * `--gpus all`：确保容器可以访问所有 GPU。
> * `-v ollama:/root/.ollama`：将容器内的 Ollama 数据目录映射到宿主机卷，避免每次重建容器都重新下载模型。
> * `-p 11434:11434`：映射端口，方便宿主机直接访问 API。

---

## 3. 验证容器运行状态

```bash
docker ps
```

你会看到类似输出：

```
CONTAINER ID   IMAGE           COMMAND                  ...   PORTS
xxxxxxxxxxxx   ollama/ollama   "/bin/sh -c 'ollama …"   ...   0.0.0.0:11434->11434/tcp
```

确认容器处于 **Up** 状态。

---

## 4. GPU 使用情况检查

进入容器或直接执行：

```bash
docker exec -it ollama nvidia-smi
```

你应该看到：

```
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 551.86       Driver Version: 551.86       CUDA Version: 12.4    |
|-------------------------------+----------------------+----------------------+
| GPU Name                    ...   Memory-Usage         ...
|-------------------------------+----------------------+----------------------+
| 0   NVIDIA XXX             ...   6–8GB / XXGB       ...
+-----------------------------------------------------------------------------+
```

> 确认显存占用 ~6–8GB 左右，表示模型在 GPU 上运行，而不是 CPU。

---

## 5. 进入容器测试模型（可选）

```bash
docker exec -it ollama bash
```

进入容器后，可以直接运行 Ollama 模型：

```bash
ollama run qwen3:4b
```

示例交互：

```
>>> 用一句话解释 UKF 在锂电池 SOC 估计中的作用
```

退出模型：`Ctrl + D`
退出容器：`exit`

> **注意**：进入容器主要用于调试，生产或常规调用推荐直接使用 API。

---

## 6. 宿主机直接使用 API（推荐方式）

无需进入容器，直接在宿主机调用模型接口：

```bash
curl http://localhost:11434/api/generate -d '{
  "model": "qwen3:4b",
  "prompt": "解释SOC与SOH的区别",
  "stream": false
}'
```

示例返回（JSON）：

```json
{
  "id": "xxxxxxxx",
  "object": "text_completion",
  "text": "SOC（State of Charge）表示电池当前剩余电量，SOH（State of Health）表示电池健康状况或衰减程度。"
}
```

---

## 7. 重要提示

1. **显存占用确认**
   使用 `nvidia-smi` 确认模型确实使用 GPU。
   Ollama / Ollama_llama_server 显存占用约 6–8GB。

2. **数据持久化**
   `-v ollama:/root/.ollama` 映射卷，保证模型下载后无需重复下载。

3. **端口配置**
   默认 API 端口为 `11434`，可以通过 `-p HOST_PORT:11434` 自定义。

4. **容器管理**

   * 停止容器：`docker stop ollama`
   * 启动容器：`docker start ollama`
   * 删除容器（慎用）：`docker rm -f ollama`

5. **调试**
   容器内可使用 `bash` 进行调试，查看日志或手动运行模型。

---

## 8. 总结

通过以上步骤，你可以：

* 快速部署 Ollama Docker 容器并启用 GPU
* 持久化模型数据，避免重复下载
* 直接在宿主机使用 API 调用大模型
* 确认 GPU 是否生效，确保高效推理

> 这套流程适用于科研、实验或小型生产部署，尤其适合 SOC/SOH 估计、问答系统、自然语言处理等任务。

