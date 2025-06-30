## 2. 启停命令

| 动作 | 命令 |
|------|------|
| 启动网关 | `docker compose --profile proxy up -d` |
| 停止网关 | `docker compose stop litellm` |
| 查看日志 | `docker logs -f --tail=100 litellm-proxy` |
| 重载配置 | 修改 `litellm/litellm.yaml` → `docker compose restart litellm` |
| 更新镜像 | `docker pull ghcr.io/berriai/litellm:main-latest && docker compose up -d` |

---

## 3. 健康检查

| 途径 | 判定 |
|------|------|
| Docker Health | `docker inspect -f '{{.State.Health.Status}}' litellm-proxy` 应为 `healthy` |
| HTTP | `curl -f http://127.0.0.1:4000/v1/models` 返回 200 |

---

## 4. 常见故障排查

| 现象 | 排查步骤 |
|------|---------|
| 前端卡在 *Connecting…* | 1. `docker logs litellm-proxy` 是否有请求记录<br>2. Nginx 是否配置 `proxy_buffering off` |
| 502 Bad Gateway | 1. LiteLLM 容器是否在运行<br>2. `curl 127.0.0.1:4000/v1/models` 能否通 |
| 401 / Invalid API key | 检查 `litellm/config-secret.env` 是否填错、行尾有空格 |
| 流式中断 (499) | 增大 `proxy_read_timeout` (≥ 600s)；检查客户端网络 |

---

## 5. 费用与限流

- **限流**：`litellm.yaml → rate_limit (rpm / tpm)`  
- **费用封顶**：`litellm.yaml → spend_cap.hard_cap_usd`  
- 超阈值时 LiteLLM 返回 `429` 并在日志打印：