# IPTV m3u8 生成器设计

## 概述

从 GITV EPG API 拉取江苏省三大运营商（联通、移动、电信）的 IPTV 直播源，为每个运营商生成独立的 m3u8 播放列表文件。

## 文件结构

```
pyproject.toml          # uv 项目配置，仅依赖 requests
generate_iptv.py        # 单脚本，所有逻辑集中在一个文件中
js-cucc.m3u             # 输出：联通 (JS_CUCC)
js-cmcc.m3u             # 输出：移动 (JS_CMCC)
js-ctcc.m3u             # 输出：电信 (JS_CTCC)
```

## 依赖

- `requests` — HTTP 客户端
- 通过 `uv` 管理（pyproject.toml）

## API 端点

| 运营商 | API 地址 | 输出文件 |
|--------|---------|----------|
| 联通 | `http://live.epg.gitv.tv/tagNewestEpgList/JS_CUCC/1/100/0.json` | `js-cucc.m3u` |
| 移动 | `http://live.epg.gitv.tv/tagNewestEpgList/JS_CMCC/1/100/0.json` | `js-cmcc.m3u` |
| 电信 | `http://live.epg.gitv.tv/tagNewestEpgList/JS_CTCC/1/100/0.json` | `js-ctcc.m3u` |
| 联通 | `http://live.epg.gitv.tv/chnInfos/JS_CUCC/0.json` | — |
| 移动 | `http://live.epg.gitv.tv/chnInfos/JS_CMCC/0.json` | — |
| 电信 | `http://live.epg.gitv.tv/chnInfos/JS_CTCC/0.json` | — |

## 脚本流程

对每个运营商依次执行：

1. **拉取 EPG 数据**：GET 请求 tagNewestEpgList JSON 接口
2. **拉取台标数据**：GET 请求 chnInfos JSON 接口，构建 `chnCode -> logoUrl` 映射
3. **解析频道列表**：从 `response['data']` 中提取所有频道条目
4. **去重画质变体**：对每个唯一的 `chnunCode`，仅保留 `chnDefinition` 值最高的条目。保持 API 返回的原始顺序
5. **解析真实流地址**：并发请求每个频道的 `playUrl`，根据响应格式提取真实流地址
   - CUCC 格式：`resp.json().get("u")`
   - CTCC 格式：`resp.json()["data"][0].get("url")`
    - 解析失败时回退使用原始 playUrl
6. **频道分组**：根据 `chnName` 关键词匹配分配分组
7. **按 API 原始顺序生成 m3u8**
8. **写入文件**

## 频道分组

按 `chnName` 关键词匹配（按优先级顺序检查，首次命中即确定分组）：

| 优先级 | 分组 | 匹配关键词 |
|--------|------|-----------|
| 1 | CCTV | `CCTV`、`CGTN` |
| 2 | 南京 | `南京` |
| 3 | 江苏 | `江苏` |
| 4 | 卫视 | `卫视` |
| 5 | 教育 | `CETV`、`教育` |
| 6 | 其他 | 兜底 |

相比参考脚本移除的分组：少儿（少儿频道合并到"其他"分组）。

## m3u8 格式

```
#EXTM3U
#EXTINF:-1 group-title="CCTV" tvg-name="CCTV-1" tvg-id="G_CCTV-1-CQ" tvg-logo="http://live.pic.gitv.tv/...",CCTV-1
http://解析后的真实流地址/playlist.m3u8?...
```

- `group-title`：关键词匹配分配的分组名
- `tvg-name`：清洗后的频道名（去除 `高清`、`超清`、`-8M` 等画质后缀）
- `tvg-id`：频道唯一标识（来自 `chnCode` 字段）
- `tvg-logo`：频道台标 URL（取自 `chnIcon` 字段）
- 显示名：仅频道名
- 时长：`-1`（直播流）
- 流地址：从二级 playUrl 请求解析出的真实播放地址
- 频道顺序：与 GITV EPG API 返回顺序一致

## 并发处理

使用 `concurrent.futures.ThreadPoolExecutor`，15 个工作线程并发解析所有频道的真实流地址。通过索引收集结果，确保输出顺序与 API 一致。

## 画质去重

JS_CUCC 会返回同一频道不同画质的重复条目（SD/HD/4K）。对于每个唯一的 `chnunCode`，仅保留 `chnDefinition` 值最高的条目。

## 错误处理

- 网络错误：打印警告，回退使用原始 playUrl，继续处理
- 无效 JSON 响应：打印错误，跳过该运营商
- 单个频道解析失败不影响其他频道
- 缺少预期字段：使用空字符串 / `None` 回退
