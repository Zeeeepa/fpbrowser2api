# 🎥 VEO 3.1 / VEO Omni Flash · 视频异步 API 接入文档（`/v1/videos`）

> **提交任务**：`POST /v1/videos`  
> **查询进度**：`GET /v1/videos/{task_id}`  
> 支持：首帧/首尾帧图片生成视频、多参考图生成视频

## 1. 模型与时长

| 模型 | `duration` | 说明 |
|---|---:|---|
| `veo-3-1` | `8` | VEO 3.1 视频 |
| `veo-omni-flash` | `10` | VEO Omni Flash 视频 |

> `duration` 必须按上表传固定值。

## 2. 完整调用流程

### Step 1 · 提交任务

```bash
curl -X POST https://xxx.xxx.xxx/v1/videos \
  -H "Authorization: Bearer api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "veo-3-1",
    "prompt": "animate smoothly from the first frame to the last frame, cinematic motion",
    "duration": 8,
    "aspect_ratio": "16:9",
    "images": [
      "https://your-cdn.com/first.jpg",
      "https://your-cdn.com/last.jpg"
    ]
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
  "model": "veo-3-1",
  "video_url": null,
  "metadata": { "result_urls": [] },
  "seconds": "8",
  "duration": 8,
  "aspect_ratio": "16:9",
  "prompt": "animate smoothly from the first frame to the last frame, cinematic motion"
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
  "model": "veo-3-1",
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

## 3. 调用示例

### 3.1 VEO 3.1 · 首帧/首尾帧生成视频

`images` 顺序为：`[首帧图, 尾帧图]`。只传 1 张时表示首帧图。

```bash
curl -X POST https://xxx.xxx.xxx/v1/videos \
  -H "Authorization: Bearer api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "veo-3-1",
    "prompt": "the subject walks forward and the camera slowly pushes in",
    "duration": 8,
    "aspect_ratio": "9:16",
    "images": [
      "https://your-cdn.com/first.jpg",
      "https://your-cdn.com/last.jpg"
    ]
  }'
```

也可以写成：

```json
{
  "first_image_url": "https://your-cdn.com/first.jpg",
  "last_image_url": "https://your-cdn.com/last.jpg"
}
```

### 3.2 VEO 3.1 · 多参考图生成视频

多参考图字段使用 `Ingredients_images`，最多 3 张。

```bash
curl -X POST https://xxx.xxx.xxx/v1/videos \
  -H "Authorization: Bearer api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "veo-3-1",
    "prompt": "combine these references into a cinematic product launch video",
    "duration": 8,
    "aspect_ratio": "16:9",
    "Ingredients_images": [
      "https://your-cdn.com/ref-1.jpg",
      "https://your-cdn.com/ref-2.jpg",
      "https://your-cdn.com/ref-3.jpg"
    ]
  }'
```

### 3.3 VEO Omni Flash · 首帧/首尾帧生成视频

```bash
curl -X POST https://xxx.xxx.xxx/v1/videos \
  -H "Authorization: Bearer api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "veo-omni-flash",
    "prompt": "animate the scene with dynamic camera movement and natural lighting",
    "duration": 10,
    "aspect_ratio": "16:9",
    "images": [
      "https://your-cdn.com/first.jpg",
      "https://your-cdn.com/last.jpg"
    ]
  }'
```

### 3.4 VEO Omni Flash · 多参考图生成视频

```bash
curl -X POST https://xxx.xxx.xxx/v1/videos \
  -H "Authorization: Bearer api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "veo-omni-flash",
    "prompt": "create a 10 second cinematic video using all references",
    "duration": 10,
    "aspect_ratio": "9:16",
    "Ingredients_images": [
      "https://your-cdn.com/character.jpg",
      "https://your-cdn.com/product.jpg"
    ]
  }'
```

## 4. 请求字段

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `model` | string | ✅ | `veo-3-1` 或 `veo-omni-flash` |
| `prompt` | string | ✅ | 生成提示词 |
| `duration` | int | ✅ | `veo-3-1` 固定 `8`；`veo-omni-flash` 固定 `10` |
| `aspect_ratio` | string | ❌ | 默认 `16:9`；支持 `16:9` / `9:16` |
| `images` | string[] | ❌ | 首帧/首尾帧图片数组，最多 2 张，顺序为 `[首帧, 尾帧]` |
| `first_image_url` / `image_url` | string | ❌ | 首帧图 URL |
| `last_image_url` / `end_image_url` | string | ❌ | 尾帧图 URL；不能只传尾帧 |
| `Ingredients_images` | string[] | ❌ | 多参考图生成视频，最多 3 张；也可写 `ingredients_images` |

## 5. 响应字段

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
