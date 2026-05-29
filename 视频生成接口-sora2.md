# 🎬 Sora 2 · 异步 API 接入文档（`/v1/videos` 统一版）
> **唯一路径**：`POST /v1/videos`（提交） + `GET /v1/videos/{id}`（轮询） —— NEW API 原生异步任务支持
sora2只支持一张参考图

## 1. 完整调用流程
### Step 1 · 提交任务
```bash
curl -X POST https://xxx.xxx.xxx/v1/videos \
  -H "Authorization: Bearer api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "sora2",
    "prompt": "...",
    "aspect_ratio": "16:9",
    "duration": 12,
    "image": "https://...",
  }'
```
立即返回：
```json
{
  "id": "v_8c7e4b...",
  "task_id": "v_8c7e4b...",
  "object": "video",
  "created_at": 1714200000,
  "status": "queued",
  "progress": 0,
  "video_url": null,
  "prompt": "...",
  "aspect_ratio": "16:9",
  "duration": 12,
  "model": "sora2"
}
```

### Step 2 · 轮询
```bash
curl https://xxx.xxx.xxx/v1/videos/v_8c7e4b... \
  -H "Authorization: api-key"
```
完成响应：

```json
{
  "id": "v_8c7e4b...",
  "object": "video",
  "status": "completed",
  "progress": 100,
  "video_url": "https://xxx.xxx.xxx/generated/v_8c7e4b....mp4",
  "completed_at": 1714200180,
  "aspect_ratio": "16:9",
  "duration": 12
}
| `status` | 含义 | 该做什么 |
|---|---|---|
| `queued` | 排队中 | 继续轮询 |
| `in_progress` | 生成中（progress 0–99） | 继续轮询 |
| `completed` | 完成 | 取 `video_url`（.mp4 直链） |
| `failed` | 失败 | 看 `error.message` |
```

## 3. 案例集

### 案例 1 · 文生短视频

```bash
curl -X POST https://xxx.xxx.xxx/v1/videos \
  -H "Authorization: Bearer api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "sora2",
    "prompt": "a golden retriever puppy running through a field of sunflowers, slow motion, sunny afternoon",
    "duration": 12,
    "aspect_ratio": "16:9"
  }'
```

### 案例 2 · 图生视频 
```bash
curl -X POST https://xxx.xxx.xxx/v1/videos \
  -H "Authorization: Bearer api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "sora2",
    "prompt": "the character in the photo turns slowly toward the camera and smiles, soft cinematic lighting",
    "duration": 12,
    "aspect_ratio": "9:16",
    "image": "https://your-cdn.com/portrait.jpg"
  }'
```


## 4. 完整字段表

### 4.1 提交字段

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `model` | string | ✅ | `sora2`|
| `prompt` | string | ✅\* | `prompt` 或 `image` 至少一项 |
| `aspect_ratio` | string | ❌ | `16:9` / `9:16`；默认 `16:9` |
| `duration` | int | ❌ | `12`
| `image` | string\|array | ❌ | 单图 URL（首帧）或多图数组 |
| `messages` | array | ❌ | OpenAI Chat 风格 |

### 4.2 响应字段

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` / `task_id` | string | 任务 ID（双字段兼容） |
| `object` | string | 固定 `"video"` |
| `created_at` | int | 提交时间戳 |
| `status` | string | `queued` / `in_progress` / `completed` / `failed` |
| `progress` | int | 0–100 |
| `video_url` | string\|null | 完成时是 mp4 直链 |
| `metadata.result_urls` | string[] | `[video_url]`（NEW API 兼容字段） |
| `prompt` / `aspect_ratio` / `duration` / `model` | – | 回显 |
| `completed_at` | int | 完成时间戳 |
| `error` | object | 失败时 `{message, code}` |