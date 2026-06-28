# 通勤找房雷达 - HTML 地图模板

本文件定义了移动端优先的交互式对比地图 HTML 模板。生成时将 `{变量}` 替换为实际数据。

## 设计原则

- **移动端优先**：底部滑动卡片，每个区域一张对比卡
- **地图全屏**：地图占满屏幕，面板浮动在底部
- **底部 Tab 切换**：驾车/公交路线用底部 tab 切换，手指友好
- **底部弹窗详情**：点击 Marker 弹出底部详情卡
- **折叠图例**：默认隐藏，按钮展开

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
  html, body { height: 100%; overflow: hidden; }
  body { font-family: -apple-system, 'PingFang SC', 'Microsoft YaHei', sans-serif; }
  #container { width: 100%; height: 100%; }

  /* 顶部信息栏 */
  .top-bar {
    position: fixed; top: 0; left: 0; right: 0; z-index: 1000;
    background: rgba(255,255,255,.95); backdrop-filter: blur(10px);
    padding: 12px 16px; display: flex; align-items: center; gap: 12px;
    box-shadow: 0 1px 6px rgba(0,0,0,.08);
  }
  .top-bar h2 { font-size: 16px; color: #333; flex: 1; }

  /* 路线切换 Tab */
  .route-tabs {
    position: fixed; top: 52px; left: 50%; transform: translateX(-50%);
    z-index: 999; display: flex; background: rgba(255,255,255,.95);
    backdrop-filter: blur(8px); border-radius: 20px; overflow: hidden;
    box-shadow: 0 2px 8px rgba(0,0,0,.1);
  }
  .route-tab {
    padding: 8px 20px; font-size: 13px; cursor: pointer;
    border: none; background: transparent; color: #666;
    min-height: 36px; transition: all .2s; white-space: nowrap;
  }
  .route-tab.active { background: #1890ff; color: #fff; border-radius: 20px; }

  /* 底部抽屉面板 */
  .bottom-sheet {
    position: fixed; left: 0; right: 0; bottom: 0; z-index: 1000;
    background: #fff; border-radius: 16px 16px 0 0;
    box-shadow: 0 -4px 20px rgba(0,0,0,.1);
    transition: height .3s ease;
    display: flex; flex-direction: column;
  }
  .bottom-sheet.collapsed { height: 40vh; }
  .bottom-sheet.expanded { height: 75vh; }

  .sheet-handle {
    width: 40px; height: 4px; border-radius: 2px;
    background: #ddd; margin: 10px auto 6px; cursor: grab; flex-shrink: 0;
  }

  /* 滑动卡片区 */
  .card-slider {
    display: flex; overflow-x: auto; gap: 12px; padding: 0 16px 12px;
    -webkit-overflow-scrolling: touch; scrollbar-width: none;
    scroll-snap-type: x mandatory; flex-shrink: 0;
  }
  .card-slider::-webkit-scrollbar { display: none; }

  .area-card {
    min-width: 260px; flex-shrink: 0; scroll-snap-align: start;
    background: #f8f9fa; border-radius: 12px; padding: 16px;
    border: 2px solid transparent; cursor: pointer; transition: all .2s;
  }
  .area-card.selected { border-color: #1890ff; background: #e6f7ff; }
  .area-card .card-header {
    display: flex; align-items: center; justify-content: space-between; margin-bottom: 12px;
  }
  .area-card .card-name { font-size: 16px; font-weight: 700; }
  .area-card .card-score {
    font-size: 24px; font-weight: 800;
    padding: 2px 10px; border-radius: 12px; color: #fff;
  }

  .area-card .card-commute {
    display: flex; gap: 12px; margin-bottom: 12px;
  }
  .area-card .commute-item {
    background: #fff; padding: 8px 12px; border-radius: 8px;
    text-align: center; flex: 1;
  }
  .area-card .commute-num { font-size: 18px; font-weight: 700; color: #333; }
  .area-card .commute-lbl { font-size: 10px; color: #999; }

  .area-card .card-amenity {
    display: flex; gap: 6px; flex-wrap: wrap;
  }
  .area-card .amenity-tag {
    padding: 3px 8px; background: #fff; border-radius: 10px;
    font-size: 11px; color: #666;
  }
  .area-card .amenity-tag b { color: #333; }

  /* 排名区域 */
  .rank-section {
    padding: 0 16px; flex: 1; overflow-y: auto;
    -webkit-overflow-scrolling: touch;
  }
  .rank-title {
    font-size: 14px; font-weight: 600; color: #333;
    padding: 8px 0; border-bottom: 1px solid #f0f0f0;
  }
  .rank-item {
    display: flex; align-items: center; padding: 12px 0;
    border-bottom: 1px solid #fafafa;
  }
  .rank-medal { font-size: 20px; margin-right: 10px; }
  .rank-name { flex: 1; font-size: 14px; font-weight: 600; }
  .rank-score-text { font-size: 14px; font-weight: 700; color: #1890ff; }
  .rank-desc { font-size: 11px; color: #999; }

  /* 底部详情弹窗 */
  .detail-popup {
    position: fixed; left: 0; right: 0; bottom: 0; z-index: 2000;
    background: #fff; border-radius: 16px 16px 0 0;
    box-shadow: 0 -4px 24px rgba(0,0,0,.15);
    padding: 20px; transform: translateY(100%);
    transition: transform .3s ease;
  }
  .detail-popup.show { transform: translateY(0); }
  .detail-close {
    position: absolute; top: 12px; right: 16px;
    width: 32px; height: 32px; border-radius: 50%;
    background: #f5f5f5; border: none; font-size: 16px;
    cursor: pointer; display: flex; align-items: center;
    justify-content: center; color: #999;
  }
  .detail-name { font-size: 18px; font-weight: 700; margin-bottom: 12px; padding-right: 40px; }
  .detail-stats { display: flex; gap: 12px; margin-bottom: 16px; }
  .detail-stat {
    background: #f5f5f5; padding: 10px 16px; border-radius: 10px;
    text-align: center; flex: 1;
  }
  .detail-stat .num { font-size: 22px; font-weight: 700; }
  .detail-stat .lbl { font-size: 10px; color: #999; margin-top: 2px; }
  .detail-amenity {
    display: grid; grid-template-columns: repeat(5, 1fr); gap: 6px;
    margin-bottom: 16px;
  }
  .detail-amenity-cell {
    text-align: center; padding: 8px; background: #f8f9fa; border-radius: 8px;
  }
  .detail-amenity-cell .num { font-size: 16px; font-weight: 700; }
  .detail-amenity-cell .lbl { font-size: 10px; color: #999; }

  /* 图例按钮 + 弹出 */
  .legend-btn {
    position: fixed; bottom: calc(40vh + 16px); right: 12px; z-index: 999;
    width: 40px; height: 40px; border-radius: 50%;
    background: rgba(255,255,255,.95); border: none;
    box-shadow: 0 2px 8px rgba(0,0,0,.15); cursor: pointer;
    font-size: 18px; display: flex; align-items: center; justify-content: center;
  }
  .legend-popup {
    position: fixed; bottom: calc(40vh + 60px); right: 12px; z-index: 999;
    background: rgba(255,255,255,.95); backdrop-filter: blur(10px);
    padding: 12px 16px; border-radius: 12px;
    box-shadow: 0 2px 12px rgba(0,0,0,.12);
    font-size: 12px; display: none;
  }
  .legend-popup.show { display: block; }
  .legend-row { display: flex; align-items: center; margin-bottom: 4px; }
  .legend-dot { width: 10px; height: 10px; border-radius: 50%; margin-right: 6px; }
  .legend-line { width: 16px; height: 3px; margin-right: 6px; border-radius: 1px; }

  /* 桌面端 */
  @media (min-width: 768px) {
    .bottom-sheet { left: auto; right: 16px; width: 400px;
      bottom: 16px; border-radius: 16px; max-height: 70vh; height: 70vh; }
    .bottom-sheet.expanded { height: 85vh; }
    .legend-btn { bottom: 16px; right: 432px; }
    .legend-popup { bottom: 60px; right: 432px; }
    .top-bar { right: auto; width: 420px; border-radius: 0 0 12px 0; }
  }
</style>
</head>
<body>

<div id="container"></div>

<!-- 顶部 -->
<div class="top-bar">
  <h2>📊 {上班地点名称}</h2>
</div>

<!-- 路线 Tab -->
<div class="route-tabs">
  <button class="route-tab active" onclick="switchRoute('driving', this)">🚗 驾车</button>
  <button class="route-tab" onclick="switchRoute('transit', this)">🚌 公交</button>
</div>

<!-- 底部抽屉 -->
<div class="bottom-sheet collapsed" id="sheet">
  <div class="sheet-handle" id="handle"></div>

  <!-- 雷达图对比 -->
  <div id="radar-wrap" style="padding:12px 12px 4px;display:none;">
    <div style="font-size:14px;font-weight:700;color:#333;margin-bottom:6px;">📊 多维对比</div>
    <canvas id="radarChart" style="max-height:220px;"></canvas>
  </div>

  <!-- 滑动卡片 -->
  <div class="card-slider" id="card-slider"></div>

  <!-- 排名 -->
  <div class="rank-section">
    <div class="rank-title">🏆 推荐排名</div>
    <div id="rank-list"></div>
  </div>
</div>

<!-- 详情弹窗 -->
<div class="detail-popup" id="detail-popup">
  <button class="detail-close" onclick="closeDetail()">✕</button>
  <div id="detail-content"></div>
</div>

<!-- 图例 -->
<button class="legend-btn" onclick="toggleLegend()">📋</button>
<div class="legend-popup" id="legend-popup"></div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.min.js"></script>
<script src="https://webapi.amap.com/loader.js"></script>
<script>
// ===== 数据注入区 =====
const WORKPLACE = { name: '{上班地点名称}', location: [{上班地点lng}, {上班地点lat}], city: '{城市简写如bj/sh/gz}' };

const AREAS = [
  {
    name: '区域名', location: [lng, lat], color: '#1890ff',
    driving: { duration: 35, distance: 18.2 },
    transit: { duration: 52, distance: 20.1 },
    amenities: { market: 12, hospital: 5, gym: 8, metro: 3, food: 14, cafe: 6 },
    score: 72,
    drivingPath: [[lng,lat], ...],
    transitPath: [[lng,lat], ...],
    listings: [
      { community: '小区名', layout: '2室1厅', area: 74, price: 3700, pricePerSqm: 50, floor: '高层(6层)', decoration: '', monthlyTotal: 3950 },
      { community: '小区名', layout: '1室1厅', area: 55, price: 3900, pricePerSqm: 71, floor: '高层(12层)', decoration: '精装', monthlyTotal: 4150 },
    ],
  },
];

const AREA_COLORS = ['#1890ff', '#52c41a', '#fa8c16', '#722ed1', '#eb2f96', '#13c2c2'];
const AMENITY_LABELS = { market: '超市菜场', hospital: '医院', gym: '健身', metro: '地铁', food: '餐饮', cafe: '咖啡奶茶' };

// ===== 底部抽屉 =====
const sheet = document.getElementById('sheet');
const handle = document.getElementById('handle');
let startY, startH;

handle.addEventListener('touchstart', (e) => {
  startY = e.touches[0].clientY; startH = sheet.offsetHeight;
}, { passive: true });
handle.addEventListener('touchmove', (e) => {
  const diff = startY - e.touches[0].clientY;
  const newH = Math.max(120, Math.min(window.innerHeight * 0.8, startH + diff));
  sheet.style.height = newH + 'px';
  sheet.classList.remove('collapsed', 'expanded');
}, { passive: true });
handle.addEventListener('touchend', () => {
  sheet.style.height = '';
  sheet.classList.toggle('expanded', sheet.offsetHeight > window.innerHeight * 0.5);
  sheet.classList.toggle('collapsed', !sheet.classList.contains('expanded'));
});
handle.addEventListener('click', () => {
  sheet.classList.toggle('collapsed');
  sheet.classList.toggle('expanded');
});

// ===== 地图 =====
window._AMapSecurityConfig = { securityJsCode: '{AMAP_SECURITY_JS_CODE}' };

AMapLoader.load({
  key: '{AMAP_JSAPI_KEY}', version: '2.0',
  plugins: ['AMap.Scale', 'AMap.ToolBar']
}).then((AMap) => {
  AMap.getConfig().appname = 'amap-commute-radar';

  const map = new AMap.Map('container', {
    viewMode: '3D', zoom: 11,
    center: WORKPLACE.location, mapStyle: 'amap://styles/light',
  });
  map.addControl(new AMap.Scale());
  map.addControl(new AMap.ToolBar());

  // 上班地点 Marker
  new AMap.Marker({
    position: WORKPLACE.location,
    content: '<div style="background:#ff4d4f;color:#fff;padding:5px 10px;border-radius:14px;font-size:12px;font-weight:600;white-space:nowrap;box-shadow:0 2px 6px rgba(0,0,0,.2)">🏢 '+WORKPLACE.name+'</div>',
    offset: new AMap.Pixel(-45, -25), zIndex: 200,
  });

  // 图层
  let drivingLayers = [], transitLayers = [];
  let currentRoute = 'driving';

  AREAS.forEach((area, i) => {
    area.color = AREA_COLORS[i % AREA_COLORS.length];

    // 区域 Marker（大点击区域）
    const marker = new AMap.Marker({
      position: area.location,
      content: '<div style="background:'+area.color+';color:#fff;width:40px;height:40px;border-radius:50%;display:flex;align-items:center;justify-content:center;font-weight:700;font-size:16px;box-shadow:0 2px 8px rgba(0,0,0,.25)">'+(i+1)+'</div>',
      offset: new AMap.Pixel(-20, -20), zIndex: 150,
    });
    marker.on('click', () => showDetail(area));
    map.add(marker);

    // 驾车路线
    if (area.drivingPath && area.drivingPath.length > 1) {
      const line = new AMap.Polyline({
        path: area.drivingPath, strokeColor: '#1890ff',
        strokeWeight: 5, strokeOpacity: 0.8, lineJoin: 'round',
      });
      drivingLayers.push(line);
    }

    // 公交路线
    if (area.transitPath && area.transitPath.length > 1) {
      const line = new AMap.Polyline({
        path: area.transitPath, strokeColor: '#52c41a',
        strokeWeight: 4, strokeOpacity: 0.8,
        strokeStyle: 'dashed', strokeDasharray: [10, 5],
      });
      transitLayers.push(line);
    }
  });

  // 默认显示驾车路线
  drivingLayers.forEach(l => map.add(l));
  map.setFitView(null, false, [60, 60, 60, 60]);

  // 路线切换
  window.switchRoute = function(type, btn) {
    document.querySelectorAll('.route-tab').forEach(b => b.classList.remove('active'));
    btn.classList.add('active');
    if (type === 'driving') {
      transitLayers.forEach(l => map.remove(l));
      drivingLayers.forEach(l => map.add(l));
    } else {
      drivingLayers.forEach(l => map.remove(l));
      transitLayers.forEach(l => map.add(l));
    }
    currentRoute = type;
    // 更新卡片高亮
    document.querySelectorAll('.area-card').forEach(card => {
      const area = AREAS[parseInt(card.dataset.idx)];
      updateCardCommute(card, area, type);
    });
  };

  // 详情弹窗
  window.showDetail = function(area) {
    const totalAmen = Object.values(area.amenities).reduce((a,b) => a+b, 0);
    const commute = currentRoute === 'driving' ? area.driving : area.transit;
    const commuteLabel = currentRoute === 'driving' ? '🚗 驾车' : '🚌 公交';
    const navUrl = 'https://uri.amap.com/navigation?to='+area.location[0]+','+area.location[1]+','+encodeURIComponent(area.name)+'&mode=car&coordinate=gaode';

    let amenityHtml = '';
    Object.entries(area.amenities).forEach(([key, val]) => {
      amenityHtml += '<div class="detail-amenity-cell"><div class="num">'+val+'</div><div class="lbl">'+(AMENITY_LABELS[key]||key)+'</div></div>';
    });

    const cityName = WORKPLACE.city || 'bj';
    const encName = encodeURIComponent(area.name);
    // 推荐房源
    let listingsHtml = '';
    if (area.listings && area.listings.length > 0) {
      listingsHtml = '<div style="margin-bottom:12px;"><div style="font-size:13px;color:#666;margin-bottom:8px;">🏘️ 推荐房源</div>';
      area.listings.slice(0, 3).forEach((l, i) => {
        listingsHtml += `<div style="background:#f8f9fa;border-radius:10px;padding:10px 12px;margin-bottom:6px;">
          <div style="display:flex;justify-content:space-between;align-items:center;">
            <span style="font-size:14px;font-weight:600;color:#333;">${i+1}. ${l.layout} ${l.area}㎡</span>
            <span style="font-size:15px;font-weight:700;color:#ff4d4f;">${l.price}元/月</span>
          </div>
          <div style="font-size:11px;color:#999;margin-top:4px;">${l.community} · ${l.floor}${l.decoration?' · '+l.decoration:''} · 单价${l.pricePerSqm}元/㎡ · 每月总成本${l.monthlyTotal}元</div>
        </div>`;
      });
      listingsHtml += '</div>';
    }

    document.getElementById('detail-content').innerHTML = `
      <div class="detail-name" style="color:${area.color}">📍 ${area.name}</div>
      <div class="detail-stats">
        <div class="detail-stat"><div class="num" style="color:#333">${area.driving.duration}<span style="font-size:11px">~${Math.round(area.driving.duration*1.3)}min</span></div><div class="lbl">🚗 驾车(平峰~高峰)</div></div>
        <div class="detail-stat"><div class="num" style="color:#333">${area.transit.duration}<span style="font-size:11px">min</span></div><div class="lbl">🚌 公交</div></div>
        <div class="detail-stat"><div class="num" style="color:${area.score>=60?'#52c41a':'#ff4d4f'}">${area.score}</div><div class="lbl">⭐ 综合分</div></div>
      </div>
      <div style="font-size:13px;color:#666;margin-bottom:8px;">🏪 周边配套 <b>${totalAmen}</b> 个</div>
      <div class="detail-amenity">${amenityHtml}</div>
      ${listingsHtml}
      <a style="display:block;width:100%;padding:14px;background:#1890ff;color:#fff;border-radius:12px;font-size:15px;font-weight:600;text-align:center;text-decoration:none;margin-bottom:8px;" href="${navUrl}" target="_blank">🧭 导航到 ${area.name}</a>
      <div style="display:flex;gap:6px;">
        <a style="flex:1;text-align:center;padding:10px 0;background:#fff7e6;color:#fa8c16;border-radius:10px;font-size:12px;text-decoration:none;font-weight:600;" href="https://${cityName}.zu.anjuke.com/fangyuan/?kw=${encName}%E6%95%B4%E7%A7%9F" target="_blank">安居客</a>
        <a style="flex:1;text-align:center;padding:10px 0;background:#f6ffed;color:#52c41a;border-radius:10px;font-size:12px;text-decoration:none;font-weight:600;" href="https://${cityName}.ke.com/zufang/rs${encName}/" target="_blank">贝壳找房</a>
        <a style="flex:1;text-align:center;padding:10px 0;background:#f0f5ff;color:#1890ff;border-radius:10px;font-size:12px;text-decoration:none;font-weight:600;" href="https://${cityName}.lianjia.com/zufang/rs${encName}/" target="_blank">链家租房</a>
      </div>
    `;
    document.getElementById('detail-popup').classList.add('show');
    map.setCenter(area.location);
    map.setZoom(13);
  };

  window.closeDetail = function() {
    document.getElementById('detail-popup').classList.remove('show');
  };

  // 填充面板
  fillPanel();
});

function updateCardCommute(card, area, type) {
  const el = card.querySelector('.commute-highlight');
  if (el) {
    const c = type === 'driving' ? area.driving : area.transit;
    el.textContent = c.duration + 'min';
  }
}

function fillPanel() {
  const sorted = [...AREAS].sort((a,b) => b.score - a.score);
  const medals = ['🥇','🥈','🥉','4️⃣','5️⃣','6️⃣'];

  // 滑动卡片
  const slider = document.getElementById('card-slider');
  AREAS.forEach((area, i) => {
    const totalAmen = Object.values(area.amenities).reduce((a,b) => a+b, 0);
    const scoreColor = area.score >= 75 ? '#52c41a' : area.score >= 60 ? '#faad14' : '#ff4d4f';
    const peakDrive = Math.round(area.driving.duration * 1.3);

    // 区域画像标签
    const profileTags = [];
    const am = area.amenities;
    if ((am.food||0) > 30) profileTags.push('🍜美食丰富');
    if ((am.cafe||0) > 10) profileTags.push('☕文艺氛围');
    if ((am.metro||0) > 5) profileTags.push('🚇交通枢纽');
    if ((am.gym||0) > 8) profileTags.push('💪运动友好');
    if ((am.market||0) > 30) profileTags.push('🛒采购便利');
    if ((am.hospital||0) > 5) profileTags.push('🏥就医方便');
    if ((am.food||0) > 30 && (am.cafe||0) > 10) profileTags.push('🌃夜生活丰富');
    if ((am.metro||0) > 3 && (am.market||0) > 20) profileTags.push('🏘️成熟社区');
    const tagHtml = profileTags.map(t => '<span style="display:inline-block;font-size:10px;background:#f0f5ff;color:#1890ff;padding:2px 6px;border-radius:8px;margin:2px 2px 0 0;">'+t+'</span>').join('');

    // 找房链接
    const cityName = WORKPLACE.city || 'bj';
    const encName = encodeURIComponent(area.name);
    const rentalHtml = `
      <div style="margin-top:8px;padding-top:6px;border-top:1px solid #f0f0f0;display:flex;gap:6px;">
        <a style="flex:1;text-align:center;padding:6px 0;background:#f6ffed;color:#52c41a;border-radius:8px;font-size:11px;text-decoration:none;font-weight:600;" href="https://${cityName}.ke.com/zufang/rs${encName}/" target="_blank">贝壳找房</a>
        <a style="flex:1;text-align:center;padding:6px 0;background:#f0f5ff;color:#1890ff;border-radius:8px;font-size:11px;text-decoration:none;font-weight:600;" href="https://${cityName}.lianjia.com/zufang/rs${encName}/" target="_blank">链家租房</a>
        <a style="flex:1;text-align:center;padding:6px 0;background:#fff7e6;color:#fa8c16;border-radius:8px;font-size:11px;text-decoration:none;font-weight:600;" href="https://www.ziroom.com/z/s${encName}/" target="_blank">自如租房</a>
      </div>`;

    let amenityTags = '';
    Object.entries(area.amenities).forEach(([key, val]) => {
      amenityTags += '<span class="amenity-tag">'+(AMENITY_LABELS[key]||key)+' <b>'+val+'</b></span>';
    });

    slider.innerHTML += `
      <div class="area-card${i===0?' selected':''}" data-idx="${i}" onclick="selectArea(${i})">
        <div class="card-header">
          <div class="card-name" style="color:${area.color}">${i+1}. ${area.name}</div>
          <div class="card-score" style="background:${scoreColor}">${area.score}</div>
        </div>
        <div class="card-commute">
          <div class="commute-item">
            <div class="commute-num commute-highlight">${area.driving.duration}<span style="font-size:11px;color:#999">~${peakDrive}min</span></div>
            <div class="commute-lbl">🚗 驾车(平峰~高峰) / ${area.driving.distance}km</div>
          </div>
          <div class="commute-item">
            <div class="commute-num">${area.transit.duration}min</div>
            <div class="commute-lbl">🚌 公交</div>
          </div>
        </div>
        ${tagHtml ? '<div style="margin-top:4px;">'+tagHtml+'</div>' : ''}
        <div class="card-amenity">${amenityTags}</div>
        ${rentalHtml}
      </div>
    `;
  });

  // 排名
  const rankList = document.getElementById('rank-list');
  sorted.forEach((area, i) => {
    const origIdx = AREAS.indexOf(area);
    let desc = '';
    if (area.driving.duration <= 20) desc = '通勤极近';
    else if (area.driving.duration <= 35) desc = '通勤适中';
    else desc = '通勤较远';
    const totalAmen = Object.values(area.amenities).reduce((a,b) => a+b, 0);
    if (totalAmen > 100) desc += '，配套极丰富';
    else if (totalAmen > 50) desc += '，配套不错';

    rankList.innerHTML += `
      <div class="rank-item" onclick="selectArea(${origIdx})">
        <span class="rank-medal">${medals[i]||''}</span>
        <div style="flex:1"><div class="rank-name" style="color:${area.color}">${area.name}</div><div class="rank-desc">${desc}</div></div>
        <span class="rank-score-text">${area.score}分</span>
      </div>
    `;
  });

  // 图例
  const legend = document.getElementById('legend-popup');
  legend.innerHTML = '<div class="legend-row"><div class="legend-dot" style="background:#ff4d4f"></div>🏢 上班地点</div>';
  AREAS.forEach((a, i) => {
    legend.innerHTML += '<div class="legend-row"><div class="legend-dot" style="background:'+a.color+'"></div> '+a.name+'</div>';
  });
  legend.innerHTML += '<div class="legend-row"><div class="legend-line" style="background:#1890ff"></div> 驾车路线</div>';
  legend.innerHTML += '<div class="legend-row"><div class="legend-line" style="background:#52c41a;border-top:2px dashed #52c41a;height:0"></div> 公交路线</div>';
}

window.selectArea = function(idx) {
  document.querySelectorAll('.area-card').forEach(c => c.classList.remove('selected'));
  const card = document.querySelector('.area-card[data-idx="'+idx+'"]');
  if (card) { card.classList.add('selected'); card.scrollIntoView({ behavior: 'smooth', inline: 'start' }); }
  if (typeof showDetail === 'function') showDetail(AREAS[idx]);
};

window.toggleLegend = function() {
  document.getElementById('legend-popup').classList.toggle('show');
};

// ===== 雷达图初始化 =====
(function() {
  if (typeof Chart === 'undefined' || AREAS.length < 2) return;
  document.getElementById('radar-wrap').style.display = 'block';
  const colors = ['#1890ff', '#52c41a', '#fa8c16', '#722ed1', '#eb2f96', '#13c2c2'];
  const datasets = AREAS.map((area, i) => {
    const cMin = (area.transit?.duration || area.driving?.duration || 60);
    const transfers = area.transit?.transfers ?? 1;
    const amenCount = Object.values(area.amenities || {}).reduce((a,b) => a+b, 0);
    return {
      label: area.name,
      data: [
        Math.max(0, (60 - cMin) / 60 * 100),
        transfers === 0 ? 100 : transfers === 1 ? 70 : 30,
        Math.min(100, Math.sqrt(amenCount) * 8),
        area.valueScore ?? 60,
      ],
      borderColor: colors[i % colors.length],
      backgroundColor: colors[i % colors.length] + '22',
      borderWidth: 2, pointRadius: 3,
    };
  });
  new Chart(document.getElementById('radarChart'), {
    type: 'radar',
    data: { labels: ['通勤速度', '换乘便利', '配套丰富', '性价比'], datasets },
    options: {
      responsive: true, maintainAspectRatio: true,
      scales: { r: { min: 0, max: 100, ticks: { stepSize: 25, font: { size: 10 }, backdropColor: 'transparent' }, pointLabels: { font: { size: 12, weight: '600' } }, grid: { color: '#e8e8e8' }, angleLines: { color: '#e8e8e8' } } },
      plugins: { legend: { position: 'bottom', labels: { font: { size: 11 }, boxWidth: 12, padding: 8 } } }
    }
  });
})();
</script>
</body>
</html>
```

## 数据注入说明

1. **WORKPLACE** — 替换 `{上班地点名称}`、`{上班地点lng}`、`{上班地点lat}`
2. **AREAS 数组** — 每个候选区域：
   ```javascript
   {
     name: '区域名', location: [lng, lat], color: '#1890ff',
     driving: { duration: 35, distance: 18.2 },
     transit: { duration: 52, distance: 20.1 },
     amenities: { market: 12, hospital: 5, gym: 8, metro: 3, food: 14, cafe: 6 },
     score: 72,
     drivingPath: [[lng,lat], ...],
     transitPath: [[lng,lat], ...],
   }
   ```
3. **AMAP_JSAPI_KEY** / **AMAP_SECURITY_JS_CODE**

## 交互功能

- **底部滑动卡片** — 每个区域一张卡，左右滑动对比
- **路线 Tab** — 顶部切换驾车/公交路线图层
- **点击区域 Marker** — 弹出底部详情（通勤时间、配套明细、导航按钮）
- **底部抽屉** — 触摸滑动展开/收起
- **折叠图例** — 右下角按钮展开
- **桌面端** — 768px 以上时抽屉变右侧面板

## 移动端适配标准

生成 HTML 时必须包含以下移动端优化：

```html
<!-- viewport 必须含 viewport-fit=cover -->
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">

<!-- CSS 必须包含 -->
<style>
*{-webkit-tap-highlight-color:transparent}  /* 去掉点击灰色高亮 */
.top-bar{padding:env(safe-area-inset-top,12px) 16px 12px}  /* 避开刘海 */
.detail-popup{padding:0 20px max(20px,env(safe-area-inset-bottom,20px))}  /* 避开 Home Indicator */
</style>

<!-- JS Key 注入：支持 ?key=&security= URL 参数，硬编码 fallback -->
<script>
var p=new URLSearchParams(location.search);
var JS_KEY=p.get('key')||'{AMAP_JSAPI_KEY}';
var SEC_CODE=p.get('security')||'{AMAP_SECURITY_JS_CODE}';
window._AMapSecurityConfig={securityJsCode:SEC_CODE};
</script>
```

Agent 生成时：`{AMAP_JSAPI_KEY}` 和 `{AMAP_SECURITY_JS_CODE}` 替换为环境变量值。未设置则保留占位符，用户可通过 `?key=xxx&security=xxx` URL 参数注入。
