# 📺 直播功能支持指南

> **关于直播功能的常见问题解答**

## 🎯 当前状态

**KatelyaTV 目前主要支持点播（VOD）内容，暂不直接支持直播流。**

### 📋 现有功能
- ✅ **点播内容**：电影、电视剧、综艺等完整视频内容
- ✅ **M3U8 播放**：支持 HLS 流媒体格式播放
- ✅ **多源聚合**：整合多个视频资源站点的点播内容
- ✅ **播放记录**：自动记录观看进度，支持断点续播

### ❌ 当前限制
- ❌ **直播频道**：不支持电视直播频道
- ❌ **实时流媒体**：不支持持续的直播流
- ❌ **直播源管理**：无专门的直播源配置界面

## 🔧 技术架构说明

### 当前系统设计
KatelyaTV 基于 **苹果 CMS V10 API 格式**，专门处理点播内容：

```json
{
  "api": "https://example.com/api.php/provide/vod",
  "name": "视频源名称"
}
```

- API 路径使用 `/provide/vod`（Video on Demand）
- 返回的是完整的视频文件信息
- 包含剧集列表和播放链接

### 直播流的技术要求
直播功能需要不同的技术架构：

```json
{
  "api": "https://example.com/api.php/provide/live",
  "name": "直播源名称",
  "type": "live"
}
```

- 需要 `/provide/live` 类型的 API
- 返回实时流媒体链接
- 无剧集概念，只有频道

## 🚀 如何添加直播支持

如果您需要直播功能，可以考虑以下方案：

### 方案一：使用现有 M3U8 播放器
虽然系统不直接支持直播源管理，但播放器本身支持 M3U8 格式，您可以：

1. **手动添加直播链接**：
   - 将直播 M3U8 链接作为"单集内容"添加
   - 在视频源 API 中返回直播流链接

2. **修改 API 响应**：
   ```json
   {
     "list": [{
       "vod_id": "live_channel_1",
       "vod_name": "央视新闻",
       "vod_play_url": "$https://live.example.com/channel1.m3u8"
     }]
   }
   ```

### 方案二：扩展系统架构
如需完整的直播功能，需要进行以下技术改进：

#### 1. 配置格式扩展
```json
{
  "cache_time": 7200,
  "api_site": {
    "vod_source": {
      "api": "https://example.com/api.php/provide/vod",
      "name": "点播源",
      "type": "vod"
    },
    "live_source": {
      "api": "https://example.com/api.php/provide/live", 
      "name": "直播源",
      "type": "live"
    }
  }
}
```

#### 2. 数据类型扩展
需要修改 `src/lib/types.ts` 添加直播相关类型：

```typescript
export interface LiveChannel {
  id: string;
  name: string;
  logo: string;
  stream_url: string;
  category: string;
  source: string;
  source_name: string;
}

export interface SearchResult {
  // 现有字段...
  content_type: 'vod' | 'live';
  channels?: LiveChannel[];  // 用于直播内容
}
```

#### 3. API 处理逻辑
修改 `src/lib/downstream.ts` 支持直播 API：

```typescript
export async function searchFromLiveApi(
  apiSite: ApiSite,
  query: string
): Promise<LiveChannel[]> {
  // 处理直播 API 响应
  const apiUrl = apiBaseUrl + '/api.php/provide/live';
  // ... 处理直播频道数据
}
```

#### 4. 界面组件更新
- 添加直播频道列表组件
- 修改播放器支持直播流特性
- 更新搜索界面区分点播/直播

## 📝 实施建议

### 对于开发者
1. **评估需求**：确定是否真的需要完整的直播功能
2. **选择方案**：
   - 简单需求：使用方案一（手动配置）
   - 复杂需求：考虑方案二（架构扩展）
3. **测试播放**：确保直播流与现有播放器兼容

### 对于用户
1. **当前解决方案**：
   - 可以通过自定义视频源 API 返回直播链接
   - 将直播频道作为"单集内容"处理
2. **未来期待**：
   - 可以在 GitHub Issues 提交直播功能需求
   - 参与项目讨论，推动直播功能开发

## 🔗 相关资源

- [GitHub Issues](https://github.com/katelya77/KatelyaTV/issues) - 提交功能请求
- [苹果 CMS 文档](https://www.maccms.la/) - 了解 API 格式
- [HLS.js 文档](https://github.com/video-dev/hls.js/) - 直播流播放技术

## 💡 总结

**简短回答：** KatelyaTV 当前专注于点播内容，不直接支持直播源管理。但播放器技术上支持 M3U8 直播流，可以通过自定义 API 实现基础直播功能。

**完整直播支持需要架构扩展，建议根据实际需求选择合适的实施方案。**

---

*最后更新：2024年* | *如有疑问，请在 GitHub Issues 中讨论*