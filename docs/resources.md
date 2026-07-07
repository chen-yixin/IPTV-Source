# IPTV 资源收集记录

## 1. IPTV 直播源

### CHINA-IPTV

- **地址**: <https://github.com/xisohi/CHINA-IPTV>
- **说明**: 国内 IPTV 直播源汇总仓库，包含各运营商（电信、联通、移动）的 M3U 源。

## 2. EPG / 台标等资源

### fanmingming/live

- **地址**: <https://github.com/fanmingming/live>
- **说明**: 提供 EPG（电子节目单）、台标、频道分类等 IPTV 周边资源。

## 3. 老张 EPG

- **EPG 接口**: <http://epg.51zmt.top:8000/e.xml>
- **网站**: <https://epg.51zmt.top:8001/>
- **说明**: 老张维护的 EPG 服务，提供 XML 格式的电子节目单数据。

## 4. 网络收集的 IPTV 源

> 预留路径，用于存放从网络收集到的各类 IPTV 直播源（M3U/M3U8 播放列表）。

📁 存放目录: [`sources/collected/`](../sources/collected/)

## 5. 历史存档 IPTV 源

> 预留路径，用于归档历史版本的 IPTV 源。

| 存档名称 | 说明 | 路径 |
|---|---|---|
| 江苏联通 GITV 源 | 江苏联通 GITV 历史源 | [`sources/archive/jiangsu-unicom-gitv/`](../sources/archive/jiangsu-unicom-gitv/) |
| 江苏移动 MOBAI 源 | 江苏移动 MOBAI 历史源 | [`sources/archive/jiangsu-mobile-mobai/`](../sources/archive/jiangsu-mobile-mobai/) |

## 6. 自用 IPTV 源生成计划

后续流程：

1. 从上述收集到的 IPTV 源中选取可用频道
2. 结合 [fanmingming/live](https://github.com/fanmingming/live) 提供的台标、EPG 等资源进行修正补充
3. 生成整理后的自用 M3U 播放列表
