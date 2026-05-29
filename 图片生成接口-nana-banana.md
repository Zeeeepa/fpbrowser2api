# 🖼️ Nana Banana 2 / Nana Banana Pro · 图片异步 API 接入文档（`/v1/videos`）

> **提交任务**：`POST /v1/videos`  
> **查询进度**：`GET /v1/videos/{task_id}`  
> 支持：文生图、多参考图生成图片

虽然入口路径是 `/v1/videos`，但这两个模型的最终结果是图片；完成后读取 `image_url` 或 `url`。

## 1. 支持模型

| 模型 | 输出 | 说明 |
|---|---|---|
| `nana-banana-2` | 图片 | 标准图片生成 |
| `nana-banana-pro` | 图片 | Pro 图片生成 |

## 2. 完整调用流程

### Step 1 · 提交任务

```bash
curl -X POST https://xxx.xxx.xxx/v1/videos \
  -H "Authorization: Bearer api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "nana-banana-2",
    "prompt": "a cute banana mascot wearing sunglasses, studio lighting, high detail",
    "aspect_ratio": "1:1",
    "resolution": "1k"
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
  "model": "nana-banana-2",
  "video_url": null,
  "metadata": { "result_urls": [] },
  "aspect_ratio": "1:1",
  "prompt": "a cute banana mascot wearing sunglasses, studio lighting, high detail"
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
  "model": "nana-banana-2",
  "video_url": null,
  "image_url": "https://xxx.xxx.xxx/generated/image.jpg",
  "url": "https://xxx.xxx.xxx/generated/image.jpg",
  "metadata": {
    "result_urls": ["https://xxx.xxx.xxx/generated/image.jpg"]
  },
  "completed_at": 1714200180000
}
```

| `status` | 含义 | 该做什么 |
|---|---|---|
| `queued` | 排队中 | 继续轮询 |
| `processing` | 生成中 | 继续轮询 |
| `completed` | 已完成 | 取 `image_url` 或 `url` |
| `failed` | 失败 | 查看 `error.message` |

建议每 3–5 秒轮询一次。

## 3. 调用示例

### 3.1 Nana Banana 2 · 文生图

```bash
curl -X POST https://xxx.xxx.xxx/v1/videos \
  -H "Authorization: Bearer api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "nana-banana-2",
    "prompt": "a playful banana mascot in a modern 3D icon style",
    "aspect_ratio": "1:1",
    "resolution": "1k"
  }'
```

### 3.2 Nana Banana 2 · 多参考图生成图片

最多 10 张参考图。

```bash
curl -X POST https://xxx.xxx.xxx/v1/videos \
  -H "Authorization: Bearer api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "nana-banana-2",
    "prompt": "make a poster using the character style and product reference",
    "aspect_ratio": "4:3",
    "resolution": "1k",
    "images": [
      "https://your-cdn.com/character.jpg",
      "https://your-cdn.com/product.jpg"
    ]
  }'
```

### 3.3 Nana Banana Pro · 文生图

```bash
curl -X POST https://xxx.xxx.xxx/v1/videos \
  -H "Authorization: Bearer api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "nana-banana-pro",
    "prompt": "premium editorial product photo, luxury magazine style, soft studio light",
    "aspect_ratio": "16:9",
    "resolution": "2k"
  }'
```

### 3.4 Nana Banana Pro · 多参考图生成图片

```bash
curl -X POST https://xxx.xxx.xxx/v1/videos \
  -H "Authorization: Bearer api-key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "nana-banana-pro",
    "prompt": "create a premium campaign image using all references",
    "aspect_ratio": "9:16",
    "resolution": "2k",
    "images": [
      "https://your-cdn.com/product.jpg",
      "https://your-cdn.com/background-style.jpg",
      "https://your-cdn.com/brand-color.jpg"
    ]
  }'
```

单参考图也可以使用：

```json
{
  "image_url": "https://your-cdn.com/reference.jpg"
}
```

## 4. 请求字段

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `model` | string | ✅ | `nana-banana-2` 或 `nana-banana-pro` |
| `prompt` | string | ✅ | 生成提示词 |
| `aspect_ratio` | string | ❌ | 默认 `16:9`；支持 `1:1` / `4:3` / `3:4` / `16:9` / `9:16` |
| `resolution` | string | ❌ | 默认 `1k`；支持 `1k` / `2k` |
| `images` | string[] | ❌ | 多参考图 URL 数组，最多 10 张 |
| `image_url` / `first_image_url` | string | ❌ | 单参考图 URL |
| `duration` | int | ❌ | 图片生成不需要传 |

## 5. 响应字段

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` / `task_id` | string | 任务 ID |
| `object` | string | 固定为 `video`（接口兼容字段） |
| `created_at` | int | 创建时间戳，毫秒 |
| `status` | string | `queued` / `processing` / `completed` / `failed` |
| `progress` | int | 0–100 |
| `model` | string | 请求模型 |
| `video_url` | null | 图片模型通常为 `null` |
| `image_url` | string\|null | 完成后的图片地址 |
| `url` | string\|null | 最终结果地址，通常等于 `image_url` |
| `metadata.result_urls` | string[] | 最终结果地址数组 |
| `completed_at` | int | 完成时间戳，毫秒 |
| `error.message` | string | 失败原因 |
