# 安居客租房数据抓取

当需要获取某个区域的实时租金数据时，使用 DrissionPage 抓取安居客。

## 依赖

```bash
pip install DrissionPage
```

需要 Chrome 浏览器已安装。

## 抓取脚本

```python
import re
from DrissionPage import ChromiumPage, ChromiumOptions

def scrape_anjuke_rent(city_code, area_name):
    """抓取安居客某个区域的租房数据
    
    Args:
        city_code: 城市代码 (bj/sh/gz/sz/hz/nj/cd/wh)
        area_name: 区域名（如"天通苑"、"新街口"）
    
    Returns:
        dict: {count, avg_price, median_price, by_layout: {layout: avg_price}}
    """
    co = ChromiumOptions()
    co.headless(True)
    co.set_argument('--disable-blink-features=AutomationControlled')
    
    page = ChromiumPage(co)
    
    try:
        import urllib.parse
        kw = urllib.parse.quote(area_name)
        url = f'https://{city_code}.zu.anjuke.com/fangyuan/?kw={kw}'
        page.get(url)
        
        import time
        time.sleep(3)
        
        items = page.eles('css:.zu-itemmod', timeout=5)
        if not items:
            items = page.eles('css:.list-item', timeout=5)
        
        listings = []
        for item in items:
            text = item.text
            price_match = re.search(r'(\d{3,5})\s*元/月', text)
            layout_match = re.search(r'(\d室\d厅)', text)
            area_match = re.search(r'(\d+\.?\d*)\s*平米', text)
            
            if price_match:
                listings.append({
                    'price': int(price_match.group(1)),
                    'layout': layout_match.group(1) if layout_match else '',
                    'area': float(area_match.group(1)) if area_match else 0,
                })
        
        if not listings:
            return None
        
        prices = [l['price'] for l in listings]
        
        # 按户型统计
        by_layout = {}
        for l in listings:
            key = l['layout'] or '未知'
            if key not in by_layout:
                by_layout[key] = []
            by_layout[key].append(l['price'])
        
        return {
            'count': len(listings),
            'avg_price': sum(prices) // len(prices),
            'median_price': sorted(prices)[len(prices) // 2],
            'min_price': min(prices),
            'max_price': max(prices),
            'by_layout': {k: sum(v)//len(v) for k, v in by_layout.items()},
        }
    finally:
        page.quit()

# 使用示例
# result = scrape_anjuke_rent('nj', '新街口')
# print(f'样本: {result["count"]}条, 均价: {result["avg_price"]}元/月, 中位数: {result["median_price"]}元/月')
```

## 城市代码映射

| 城市 | 代码 | 安居客域名 |
|:---|:---|:---|
| 北京 | bj | bj.zu.anjuke.com |
| 上海 | sh | sh.zu.anjuke.com |
| 广州 | gz | gz.zu.anjuke.com |
| 深圳 | sz | sz.zu.anjuke.com |
| 杭州 | hz | hz.zu.anjuke.com |
| 南京 | nj | nj.zu.anjuke.com |
| 成都 | cd | cd.zu.anjuke.com |
| 武汉 | wh | wh.zu.anjuke.com |

## 注意事项

- 安居客页面结构可能变化，选择器需定期检查
- 首次运行需等待 3 秒让页面加载
- 抓取频率不宜过高，建议每次间隔 ≥ 5 秒
- 若安居客更新反爬策略，可能需要调整 UA 或增加等待时间
- 抓取失败时降级使用 references/rental-reference.md 静态数据
