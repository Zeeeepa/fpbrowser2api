# 🎨 GPT Image · 异步 API 接入文档（`/v1/videos` 统一版）

> **唯一路径**：`POST /v1/videos`（提交） + `GET /v1/videos/{id}`（轮询） —— NEW API 原生异步任务支持，**0 配置即用**，自带进度条 +

---

##文生图

```bash
# ① 提交（走 /v1/videos，NEW API 自动当异步任务跟踪 → 进度条 + 失败退款 + 任务历史）
curl -X POST https://xxx.xxx.xxx/v1/videos \
  -H "Authorization: Bearer apikey" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2-2K",
    "prompt": "retro neon poster, cyberpunk girl",
    "aspect_ratio": "9:16"
  }'
---

##多图生图

```bash
curl -X POST https://xxx.xxx.xxx/v1/videos \
  -H "Authorization: Bearer apikey" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2-2K",
    "prompt": "the character from the reference photo, in a futuristic city",
    "aspect_ratio": "16:9",
    "image_urls": [
      "https://your-cdn.com/character.jpg",
      "https://your-cdn.com/scene.jpg"
    ]
  }'

---

## 1. 完整调用流程

### Step 1 · 提交任务

```bash
curl -X POST https://xxx.xxx.xxx/v1/videos \
  -H "Authorization: Bearer apikey" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2-2K",
    "prompt": "retro neon poster",
    "image_urls": ["https://your-cdn.com/ref.jpg"]
  }'
```

立即返回（< 1 秒）：

```json
{
  "id": "8c7e4b...",
  "task_id": "8c7e4b...",
  "object": "video",
  "created_at": 1714200000,
  "status": "queued",
  "progress": 0,
  "video_url": null,
  "prompt": "retro neon poster",
  "aspect_ratio": "16:9",
  "model": "gpt-image-2-2K-16x9"
}
```

### Step 2 · 轮询任务
**轮询策略**：
### Step 3 · 状态判断

| `status` | 含义 | 该做什么 |
|---|---|---|
| `queued` | 排队中 | 继续轮询 |
| `in_progress` | 生成中（progress 0–99） | 继续轮询 |
| `completed` | 完成 | 取 `video_url`（图片直链 .png） |
| `failed` | 失败 | 看 `error.message` |

完成响应：

```json
{
  "id": "8c7e4b...",
  "object": "video",
  "status": "completed",
  "progress": 100,
  "video_url": "https://YOUR-HOST/generated/8c7e4b....png",
  "completed_at": 1714200068,
  "aspect_ratio": "9:16"
}
```
