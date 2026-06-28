<div align="center">

# 🏠 通勤找房助手

**说一句话，30 秒拿到一份找房报告**

[![Amap Skill](https://img.shields.io/badge/高德开放平台-Skill-blue)](https://lbs.amap.com/)
[![Version](https://img.shields.io/badge/version-1.0.0-green)]()
[![Platform](https://img.shields.io/badge/platform-AI%20Agent%20%7C%20Claude%20Code%20%7C%20OpenClaw%20%7C%20QoderWork-blueviolet)]()

[📖 场景](#-场景) · [📸 效果演示](#-效果演示) · [✨ 核心功能](#-核心功能) · [📊 示例](#-示例) · [🚀 安装](#-安装) · [💬 使用](#-使用) · [❓ 常见问题](#-常见问题) · [📂 项目结构](#-项目结构) · [English](#english)

</div>

---

## 📖 场景

新到一个城市实习 / 入职 / 换工作，最头疼的不是找工作，是**找房子**——不知道住哪、通勤多久、租金多少、地铁挤不挤、周边有没有吃的。

把这个 Skill 丢给 AI，你只需要说：

> *"我在望京上班，预算 5000，帮我找房"*

**30 秒后你会拿到：**
- 🥇 **Top 3 推荐区域**（附评分 + 通勤方式 + 每月总花费）
- 🏘️ **具体房源**（小区名 / 户型 / 面积 / 月租）
- 🔗 **一键跳转找房**（贝壳 / 链家 / 自如链接直接搜）
- 🗺️ **交互地图**（地铁线 + 区域标记 + 点击看详情，手机电脑都能用）

不用刷 10 个帖子、不用在地图上一个个量距离、不用算来算去——**一句话全搞定。**

## 📸 效果演示

**电脑端 · 全图展示**：路线切换（公交/驾车）+ 区域标记 + 推荐标签

<p align="center">
  <img src="docs/screenshot-map.png" alt="电脑端全图展示" width="720">
</p>

**电脑端 · 细节展示**：通勤信息 + 每月总支出 + 房源列表 + 找房链接

<p align="center">
  <img src="docs/screenshot-detail.png" alt="电脑端细节展示" width="720">
</p>

**手机端 · 全图展示**

<p align="center">
  <img src="docs/mobile-map-link.jpg" alt="手机端全图展示" width="360">
</p>

**手机端 · 细节展示**

<p align="center">
  <img src="docs/mobile-map-detail.jpg" alt="手机端细节展示" width="360">
</p>

**手机端 · 房源链接**

<p align="center">
  <img src="docs/mobile-map-detail-house-link-2.jpg" alt="手机端房源链接" width="360">
</p>

## ✨ 核心功能

| 你只需要说 | 助手帮你做 |
|:---|:---|
| "我在望京上班，预算 5000" | 自动沿地铁发现候选区域 + 通勤计算 + 配套评估 + 租金参考 + Top 3 排名 |
| "我在国贸上班，对比回龙观和天通苑" | 跳过自动发现，直接评估指定区域 |
| "我在西二旗上班，预算 4000，生成地图" | 带预算推荐 + 交互式 HTML 地图 |
| "太远了，有没有近一点的" | 调整搜索圈层，重新推荐 |

### 智能发现流程

1. **解析上班地点** → 高德地理编码
2. **三维度发现候选区域**：步行/骑行可达（≤3km）+ 地铁沿线（3-20km）+ 公交可达
3. **评估每个区域**：通勤时间（公交+驾车双算）+ 周边配套（6 类 POI）+ 租金参考
4. **评分排名**：通勤 50% + 配套 30% + 性价比 20%
5. **推荐房源**：按用户身份（实习生/应届/换工作/情侣）智能推荐整租或合租
6. **输出**：文字报告 + CSV + 交互地图

### 对话流追问

智能追问 8 个关键维度：上班地点 → 几个人住 → 预算 → 身份 → 通勤容忍度 → 整租/合租 → 特别需求。能推断的自动推断，不多问。

## 📊 示例

两个真实场景，基于高德 API 实时数据生成：

| 场景 | 公司 | 预算 | 人群 | 推荐结果 | 文件 |
|------|------|------|------|------|------|
| 🏢 北京望京 | 高德/阿里 | ¥3000/月 | 一个人·应届 | 花家地 20min·合租单间 | [`examples/beijing-amap/`](examples/beijing-amap/) |
| 📱 南京小米 | 小米科技园 | ¥4000/月 | 情侣·整租一居 | 油坊桥 32min·整租一居 | [`examples/nanjing-xiaomi/`](examples/nanjing-xiaomi/) |

每个示例：`report.md`（报告+原始API数据） + `rental_areas.csv` + `rental_listings.csv` + `map.html`（交互地图）

<details>
<summary>📄 北京望京 · 一个人 · ¥3000 · ≤30min（点击展开完整报告）</summary>

```
📋 一句话总结
  推荐住花家地：14号线直达 20 分钟，合租单间 ¥2200-3200，配套 666 个。

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🏢 上班地点：望京 | 💰 预算：3000元/月 | 👤 一个人 | ⏱️ ≤30min

🥇 花家地(14号线) 🚌20min·直达 🏘️成熟社区
  综合评分 78分 | 参考租金 合租单间 2200-3200 | 每月总成本约 2700元
  配套：超市 23 · 餐饮 412 · 医院 118 · 健身 107 · 地铁 6
  💡 通勤最短 20 分钟直达，老社区生活气息浓

🥈 酒仙桥(14号线) 🚌23min·直达 💪运动友好
  综合评分 76分 | 参考租金 合租单间 2400-3800 | 每月总成本约 2850元
  💡 14号线往北 1 站到望京，早高峰有座位

🥉 将台(14号线) 🚌26min·直达 🚇交通枢纽
  综合评分 75分 | 参考租金 合租单间 2200-3600 | 每月总成本约 2750元
  💡 配套最多 699 个

🎯 如果只选一个：花家地合租单间
  通勤最短 20min，合租 ¥2200-3200 在预算内，每月总成本约 ¥2700

🔗 贝壳找房 / 链家租房 / 安居客 / 自如 一键直达

📁 已生成 CSV：rental_areas.csv（7 区域） + rental_listings.csv（15 套房源）
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
</details>

## 🚀 安装

```bash
openclaw skills install @dongdongyue/amap-settle-guide
```

### 环境变量

| 变量名 | 必填 | 获取方式 |
|--------|------|----------|
| `AMAP_API_KEY` | ✅ | [高德开放平台](https://lbs.amap.com/) → 控制台 → 添加 Key → 类型选「Web 服务」 |
| `AMAP_JSAPI_KEY` | ❌ | 同上，类型选「Web端(JS API)」，用于交互地图 |
| `AMAP_SECURITY_JS_CODE` | ❌ | JS API 安全密钥 |

> 免费额度：5000 次/天，找房够用。

## 💬 使用

```
"我在望京上班，帮我推荐住哪"              → 自动发现候选区域 + 推荐
"我在国贸上班，对比回龙观和天通苑"         → 评估指定区域
"我在西二旗上班，预算 4000"               → 带预算的推荐
"帮我生成一个地图"                       → 输出交互式 HTML 地图
"太远了，有没有近一点的"                  → 缩小范围重新推荐
```

## ❓ 常见问题

**Q: 推荐的区域我不满意怎么办？**
直接说"还有别的推荐吗"或"太远了，有没有近一点的"，助手会调整搜索范围重新推荐。

**Q: 支持哪些城市？**
全国有地铁的城市都支持。租金参考数据覆盖北上广深杭宁蓉汉 8 城，其他城市无租金对比但通勤和配套分析正常。

**Q: 找房链接打不开？**
链接跳转到贝壳/链家/自如的搜索页，按城市自动匹配。如果跳转不对，手动在租房 App 搜索区域名即可。

**Q: 需要装 Python 吗？**
不需要。本 Skill 是纯提示词驱动，AI 助手直接调用高德 HTTP API。

## 📂 项目结构

```
amap-settle-guide/
├── README.md
├── SKILL.md                       # Skill 定义（核心）
├── skill.json                     # 元数据
├── LICENSE                        # MIT 协议
├── .env.example                   # 环境变量模板
├── .gitignore
├── docs/                          # 效果截图
│   ├── screenshot-map.png
│   ├── screenshot-detail.png
│   ├── mobile-map-link.jpg
│   ├── mobile-map-detail.jpg
│   └── mobile-map-detail-house-link-2.jpg
├── references/                    # 运行时模板
│   ├── html-template-commute.md   # 交互地图模板
│   ├── text-card-commute.md       # CSS 文字卡片模板
│   ├── rental-reference.md        # 8 城租金参考表
│   ├── anjuke-scraper.md          # 安居客抓取方案
│   └── social-rental-scraper.md   # 58同城/链家抓取方案
└── examples/                      # 输出示例
    ├── beijing-amap/              # 北京望京
    └── nanjing-xiaomi/            # 南京小米
```

### 调用的高德 API

| API | 用途 |
|-----|------|
| `geocode/geo` | 地名 → 坐标 |
| `place/around` | 周边 POI 搜索 |
| `place/text` | 地铁站搜索 |
| `direction/driving` | 驾车路线 |
| `direction/transit/integrated` | 公交路线 |

---

## English

### The Problem

You just got an internship / new job in a city you don't know. You need to find an apartment — but you have no idea which neighborhoods are livable, how long the commute is, or what rent actually costs.

### The Fix

Drop this Skill into your AI assistant and say:

> *"I work at Wangjing, budget ¥5000/mo, help me find a place"*

**30 seconds later you get:**
- 🥇 **Top 3 recommended areas** with scores, commute method, and monthly total cost
- 🏘️ **Specific listings** — community name, layout, size, rent
- 🔗 **One-click rental links** — Beike / Lianjia / Ziroom search pages
- 🗺️ **Interactive map** — metro lines, area markers, click-for-details, works on mobile

### Demo

<p align="center">
  <img src="docs/screenshot-map.png" alt="Desktop Full Map" width="720">
  <img src="docs/screenshot-detail.png" alt="Desktop Detail View" width="720">
  <img src="docs/mobile-map-link.jpg" alt="Mobile Full Map" width="360">
</p>

### Install

```bash
openclaw skills install @dongdongyue/amap-settle-guide
```

> Requires a free Amap API Key.

### Environment Variables

| Variable | Required | How to get it |
|----------|----------|---------------|
| `AMAP_API_KEY` | ✅ | [Amap Open Platform](https://lbs.amap.com/) → Console → Add Key → type: 「Web Service」 |
| `AMAP_JSAPI_KEY` | ❌ | Same, type: 「Web(JS API)」 — for interactive maps |
| `AMAP_SECURITY_JS_CODE` | ❌ | JS API security code |

### Usage

```
"I work at Wangjing, recommend where to live"           → auto-discover + recommend
"I work at Guomao, compare Huilongguan vs Tiantongyuan" → evaluate specific areas
"I work at Xierqi, budget 4000"                         → budget-aware
"Generate a map for me"                                 → interactive HTML map
```

### FAQ

**Q: What if I don't like the recommended areas?**
Just say "anything closer?" — the assistant will adjust the search range.

**Q: Which cities are supported?**
All cities with metro systems. Rental data covers 8 major cities.

**Q: Do I need Python?**
No. Pure prompt-driven, AI calls Amap HTTP API directly.

---

## 📝 License

MIT
