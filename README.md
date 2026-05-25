# 江苏 IPTV 直播源

从 GITV EPG API 拉取江苏联通/移动/电信的 IPTV 直播源，生成 m3u8 播放列表。

## 使用

```bash
uv sync
uv run python generate_iptv.py
```

生成的文件：

| 文件 | 运营商 |
|------|--------|
| `js-cucc.m3u` | 联通 |
| `js-cmcc.m3u` | 移动 |
| `js-ctcc.m3u` | 电信 |

## 频道分组

CCTV / 江苏 / 南京 / 卫视 / 教育 / 其他

## 参考

- [lesca/jiangsu-iptv](https://github.com/lesca/jiangsu-iptv) — 原始参考脚本（仅联通）
