# 通勤找房雷达 - HTML 文字卡片模板

当用户未配置 JS API Key 时，生成此文字卡片作为替代。纯 CSS + 数据，无外部依赖，手机友好。

## 完整模板

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>通勤找房雷达 - {上班地点}</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    font-family: -apple-system, 'PingFang SC', 'Microsoft YaHei', sans-serif;
    background: linear-gradient(135deg, #1a73e8 0%, #0d47a1 100%);
    min-height: 100vh; padding: 16px; color: #333;
  }

  .card {
    background: #fff; border-radius: 16px; padding: 24px;
    margin-bottom: 12px; box-shadow: 0 4px 20px rgba(0,0,0,.1);
  }

  /* 头部 */
  .header h1 { font-size: 22px; margin-bottom: 4px; }
  .header .subtitle { font-size: 13px; color: #999; }

  /* 排名卡片 */
  .rank-card {
    display: flex; align-items: center; padding: 16px;
    border-radius: 12px; margin-bottom: 8px;
    transition: transform .2s;
  }
  .rank-card:active { transform: scale(.98); }
  .rank-medal { font-size: 28px; margin-right: 12px; }
  .rank-info { flex: 1; }
  .rank-name { font-size: 16px; font-weight: 700; }
  .rank-desc { font-size: 12px; color: rgba(255,255,255,.8); margin-top: 2px; }
  .rank-score {
    font-size: 32px; font-weight: 800; color: rgba(255,255,255,.9);
    line-height: 1;
  }
  .rank-score-unit { font-size: 12px; font-weight: 400; opacity: .7; }
  .rank-1 { background: linear-gradient(135deg, #52c41a, #389e0d); color: #fff; }
  .rank-2 { background: linear-gradient(135deg, #1890ff, #096dd9); color: #fff; }
  .rank-3 { background: linear-gradient(135deg, #faad14, #d48806); color: #fff; }

  /* 对比表格 */
  .compare-section { padding: 0; }
  .compare-row {
    display: flex; align-items: center; padding: 14px 0;
    border-bottom: 1px solid #f5f5f5;
  }
  .compare-row:last-child { border-bottom: none; }
  .compare-label { flex: 1; font-size: 13px; color: #666; }
  .compare-values { display: flex; gap: 16px; }
  .compare-val {
    text-align: center; min-width: 72px; padding: 6px 8px;
    border-radius: 8px; background: #f8f9fa;
  }
  .compare-val .area-name { font-size: 10px; color: #999; margin-bottom: 2px; }
  .compare-val .val { font-size: 16px; font-weight: 700; }
  .compare-val.best .val { color: #52c41a; }

  /* 评分条 */
  .score-bar-section { margin-top: 12px; }
  .score-bar-item { margin-bottom: 12px; }
  .score-bar-header {
    display: flex; justify-content: space-between; align-items: center;
    margin-bottom: 4px;
  }
  .score-bar-name { font-size: 13px; font-weight: 600; }
  .score-bar-total { font-size: 13px; font-weight: 700; }
  .score-bar-track {
    height: 8px; background: #f0f0f0; border-radius: 4px;
    overflow: hidden; display: flex;
  }
  .score-bar-commute { background: #1890ff; height: 100%; transition: width .5s; }
  .score-bar-amenity { background: #52c41a; height: 100%; transition: width .5s; }
  .score-bar-legend {
    display: flex; gap: 16px; margin-top: 8px; font-size: 11px; color: #999;
  }
  .score-bar-legend span::before {
    content: ''; display: inline-block; width: 8px; height: 8px;
    border-radius: 2px; margin-right: 4px; vertical-align: middle;
  }
  .score-bar-legend .lbl-commute::before { background: #1890ff; }
  .score-bar-legend .lbl-amenity::before { background: #52c41a; }

  /* 配套明细 */
  .amenity-grid {
    display: grid; grid-template-columns: repeat(5, 1fr); gap: 6px;
    margin-top: 8px;
  }
  .amenity-cell {
    text-align: center; padding: 8px 4px;
    background: #f8f9fa; border-radius: 8px;
  }
  .amenity-cell .num { font-size: 18px; font-weight: 700; color: #333; }
  .amenity-cell .lbl { font-size: 10px; color: #999; margin-top: 2px; }

  .area-title {
    font-size: 14px; font-weight: 700; margin-bottom: 8px;
    display: flex; align-items: center; gap: 8px;
  }
  .area-dot { width: 10px; height: 10px; border-radius: 50%; }

  /* 导航按钮 */
  .nav-btn {
    display: inline-block; margin-top: 8px; padding: 6px 14px;
    background: #e6f7ff; color: #1890ff; border-radius: 14px;
    font-size: 12px; text-decoration: none; font-weight: 500;
  }

  /* 底部 */
  .footer {
    text-align: center; padding: 16px; font-size: 11px; color: rgba(255,255,255,.6);
  }
</style>
</head>
<body>

<!-- 头部 -->
<div class="card header">
  <h1>📊 通勤找房雷达</h1>
  <p class="subtitle">🏢 上班地点：{上班地点名称}</p>
</div>

<!-- 排名卡片 -->
<!-- AI 为每个区域按评分从高到低生成，第一个 rank-1，第二个 rank-2，依次类推 -->
<div class="rank-card rank-1">
  <span class="rank-medal">🥇</span>
  <div class="rank-info">
    <div class="rank-name">{区域名}</div>
    <div class="rank-desc">{推荐语} · {画像标签如：🍜美食丰富 🏘️成熟社区}</div>
  </div>
  <div class="rank-score">{评分}<span class="rank-score-unit">分</span></div>
</div>
<!-- 更多排名... -->

<!-- 对比表格 -->
<div class="card compare-section">
  <div style="font-size:14px;font-weight:600;margin-bottom:8px;">📋 详细对比</div>

  <div class="compare-row">
    <div class="compare-label">🚗 驾车(平峰~高峰)</div>
    <div class="compare-values">
      <!-- AI 为每个区域生成 compare-val，最短时间的加 class="best" -->
      <div class="compare-val best">
        <div class="area-name">{区域名}</div>
        <div class="val">{时间}min</div>
      </div>
      <!-- 更多... -->
    </div>
  </div>

  <div class="compare-row">
    <div class="compare-label">🚌 公交</div>
    <div class="compare-values">
      <!-- 同上 -->
    </div>
  </div>

  <div class="compare-row">
    <div class="compare-label">📏 驾车距离</div>
    <div class="compare-values">
      <!-- 同上 -->
    </div>
  </div>

  <div class="compare-row">
    <div class="compare-label">🏪 配套数量</div>
    <div class="compare-values">
      <!-- 同上，最多的加 best -->
    </div>
  </div>
</div>

<!-- 评分可视化 -->
<div class="card">
  <div style="font-size:14px;font-weight:600;margin-bottom:12px;">⭐ 评分拆解</div>

  <!-- AI 为每个区域生成评分条 -->
  <div class="score-bar-item">
    <div class="score-bar-header">
      <span class="score-bar-name" style="color:{区域颜色}">{区域名}</span>
      <span class="score-bar-total">{综合分}分</span>
    </div>
    <div class="score-bar-track">
      <!-- 通勤分占 60%，配套分占 40%，按实际值比例显示 -->
      <div class="score-bar-commute" style="width:{通勤分占比}%"></div>
      <div class="score-bar-amenity" style="width:{配套分占比}%"></div>
    </div>
  </div>
  <!-- 更多区域... -->

  <div class="score-bar-legend">
    <span class="lbl-commute">通勤时间（60%）</span>
    <span class="lbl-amenity">生活配套（40%）</span>
  </div>
</div>

<!-- 各区域配套明细 -->
<!-- AI 为每个区域生成明细卡片 -->
<div class="card">
  <div class="area-title">
    <div class="area-dot" style="background:{区域颜色}"></div>
    {区域名}
  </div>
  <div class="amenity-grid">
    <div class="amenity-cell"><div class="num">{超市菜场数}</div><div class="lbl">超市菜场</div></div>
    <div class="amenity-cell"><div class="num">{医院诊所数}</div><div class="lbl">医院诊所</div></div>
    <div class="amenity-cell"><div class="num">{健身运动数}</div><div class="lbl">健身运动</div></div>
    <div class="amenity-cell"><div class="num">{地铁站数}</div><div class="lbl">地铁站</div></div>
    <div class="amenity-cell"><div class="num">{餐饮美食数}</div><div class="lbl">餐饮美食</div></div>
    <div class="amenity-cell"><div class="num">{咖啡奶茶数}</div><div class="lbl">咖啡奶茶</div></div>
  </div>
  <a class="nav-btn" href="https://uri.amap.com/navigation?to={lng},{lat},{name}&mode=car&coordinate=gaode" target="_blank">🧭 查看路线</a>
  <div style="display:flex;gap:6px;margin-top:8px;">
    <a style="flex:1;text-align:center;padding:8px 0;background:#f6ffed;color:#52c41a;border-radius:8px;font-size:11px;text-decoration:none;font-weight:600;" href="https://{城市}.ke.com/zufang/rs{区域名}/" target="_blank">贝壳找房</a>
    <a style="flex:1;text-align:center;padding:8px 0;background:#f0f5ff;color:#1890ff;border-radius:8px;font-size:11px;text-decoration:none;font-weight:600;" href="https://{城市}.lianjia.com/zufang/rs{区域名}/" target="_blank">链家租房</a>
    <a style="flex:1;text-align:center;padding:8px 0;background:#fff7e6;color:#fa8c16;border-radius:8px;font-size:11px;text-decoration:none;font-weight:600;" href="https://www.ziroom.com/z/s{区域名}/" target="_blank">自如租房</a>
  </div>
</div>
<!-- 更多区域... -->

<div class="footer">
  由 <a href="https://lbs.amap.com" style="color:rgba(255,255,255,.8)">高德地图</a> 提供数据支持 · 通勤找房雷达 Skill
</div>

</body>
</html>
```

## 数据注入说明

AI 生成时需替换：

1. **头部** — `{上班地点名称}`
2. **排名卡片** — 按综合分从高到低排列，使用 `rank-1`/`rank-2`/`rank-3` 样式。推荐语附带画像标签
3. **对比表格** — 驾车显示"平峰~高峰"格式(如 `28~42min`)，其余指标同前
4. **评分条** — 通勤分占比 = `通勤分/100 × 60`，配套分占比 = `配套分/100 × 40`
5. **配套明细** — 每个区域一张明细卡 + 导航链接 + 找房链接（贝壳/链家/自如）
6. **导航链接**：`https://uri.amap.com/navigation?to={lng},{lat},{name}&mode=car&coordinate=gaode`
7. **找房链接**：`https://{城市简写}.ke.com/zufang/rs{区域名}/` 等，城市简写如 bj/sh/gz

## 设计特点

- 无需任何外部依赖
- 渐变背景 + 白色卡片
- 排名用渐变色卡片（绿 → 蓝 → 橙），视觉冲击力强
- 评分条用双色进度条直观展示通勤/配套权重
- 配套明细用网格布局，紧凑清晰
- 导航按钮可唤起高德地图 App
