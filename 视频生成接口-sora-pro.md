# 🎬 Seedance 2 · 视频异步 API 接入文档（`/v1/videos`）

> **提交任务**：`POST /v1/videos`  
> **查询进度**：`GET /v1/videos/{task_id}`  
> 支持：文生视频、首尾帧图片生成视频、多参考图生成视频（`omni_reference`）

图片地址请使用可公网访问的 `https://` URL。

## 1. 完整调用流程

### Step 1 · 提交任务

```bash
curl -X POST https://xxx.xxx.xxx/v1/videos \
  -H "Authorization: Bearer api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "sora-pro",
    "prompt": "a golden retriever puppy running through sunflowers, cinematic, slow motion",
    "duration": 15,
    "aspect_ratio": "16:9",
    "resolution": "720p"
  }'
```

立即返回：

```json
{
  "id": "v_8c7e4b...",
  "task_id": "v_8c7e4b...",
  "object": "video",
  "created_at": 1714200000000,
  "status": "queued",
  "progress": 0,
  "model": "sora-pro",
  "video_url": null,
  "metadata": { "result_urls": [] },
  "seconds": "15",
  "duration": 15,
  "aspect_ratio": "16:9",
  "prompt": "a golden retriever puppy running through sunflowers, cinematic, slow motion"
}
```

### Step 2 · 轮询任务

```bash
curl https://xxx.xxx.xxx/v1/videos/v_8c7e4b... \
  -H "Authorization: Bearer api-key"
```

完成响应：

```json
{
  "id": "v_8c7e4b...",
  "task_id": "v_8c7e4b...",
  "object": "video",
  "created_at": 1714200000000,
  "status": "completed",
  "progress": 100,
  "model": "sora-pro",
  "video_url": "https://xxx.xxx.xxx/generated/video.mp4",
  "url": "https://xxx.xxx.xxx/generated/video.mp4",
  "metadata": {
    "result_urls": ["https://xxx.xxx.xxx/generated/video.mp4"]
  },
  "completed_at": 1714200180000
}
```

| `status` | 含义 | 该做什么 |
|---|---|---|
| `queued` | 排队中 | 继续轮询 |
| `processing` | 生成中 | 继续轮询 |
| `completed` | 已完成 | 取 `video_url` 或 `url` |
| `failed` | 失败 | 查看 `error.message` |

建议每 3–5 秒轮询一次。

## 2. 调用示例

### 2.1 文生视频

不传图片即可。

```bash
curl -X POST https://xxx.xxx.xxx/v1/videos \
  -H "Authorization: Bearer api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "sora-pro",
    "prompt": "a cinematic drone shot over a misty mountain village at sunrise",
    "duration": 15,
    "aspect_ratio": "16:9",
    "resolution": "720p"
  }'
```

### 2.2 首尾帧图片生成视频

`images` 顺序为：`[首帧图, 尾帧图]`。只传 1 张时表示首帧图。

```bash
curl -X POST https://xxx.xxx.xxx/v1/videos \
  -H "Authorization: Bearer api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "sora-pro",
    "prompt": "animate naturally from the first frame to the final frame, smooth camera movement",
    "duration": 15,
    "aspect_ratio": "9:16",
    "resolution": "720p",
    "function_mode": "first_last_frames",
    "images": [
      "https://your-cdn.com/first.jpg",
      "https://your-cdn.com/last.jpg"
    ]
  }'
```

也可以写成：

```json
{
  "function_mode": "first_last_frames",
  "first_image_url": "https://your-cdn.com/first.jpg",
  "last_image_url": "https://your-cdn.com/last.jpg"
}
```

### 2.3 多参考图生成视频（`omni_reference`）

最多 9 张参考图。

```bash
curl -X POST https://xxx.xxx.xxx/v1/videos \
  -H "Authorization: Bearer api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "sora-pro",
    "prompt": "use the character, outfit and product references to make a fashion commercial video",
    "duration": 15,
    "aspect_ratio": "16:9",
    "resolution": "720p",
    "function_mode": "omni_reference",
    "images": [
      "https://your-cdn.com/character.jpg",
      "https://your-cdn.com/outfit.jpg",
      "https://your-cdn.com/product.jpg"
    ]
  }'
```

## 3. 请求字段

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `model` | string | ✅ | 固定传 `sora-pro` |
| `prompt` | string | ✅ | 生成提示词 |
| `duration` | int | ✅ | 固定传 `15` |
| `aspect_ratio` | string | ❌ | 默认 `16:9`；支持 `16:9` / `9:16` / `1:1` / `4:3` / `3:4` / `21:9` |
| `resolution` | string | ❌ | 默认 `720p`；支持 `480p` / `720p` / `1080p` |
| `function_mode` | string | ❌ | 首尾帧传 `first_last_frames`；多参考图传 `omni_reference`；文生视频可不传 |
| `images` | string[] | ❌ | 图片 URL 数组。首尾帧最多 2 张；`omni_reference` 最多 9 张 |
| `first_image_url` | string | ❌ | 首帧图 URL |
| `last_image_url` | string | ❌ | 尾帧图 URL |

## 4. 响应字段

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` / `task_id` | string | 任务 ID |
| `object` | string | 固定为 `video` |
| `created_at` | int | 创建时间戳，毫秒 |
| `status` | string | `queued` / `processing` / `completed` / `failed` |
| `progress` | int | 0–100 |
| `model` | string | 请求模型 |
| `video_url` | string\|null | 完成后的视频地址 |
| `url` | string\|null | 最终结果地址，通常等于 `video_url` |
| `metadata.result_urls` | string[] | 最终结果地址数组 |
| `completed_at` | int | 完成时间戳，毫秒 |
| `error.message` | string | 失败原因 |
