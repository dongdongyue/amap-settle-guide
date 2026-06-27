# 多平台租房数据抓取

通过 58同城和链家获取租房数据，补充安居客的房源信息。两个平台各有优势：58同城数据量大、含个人房源；链家数据规范、户型信息完整。

## 依赖

```bash
pip install DrissionPage
```

需要 Chrome 浏览器已安装。

## 58同城抓取

```python
import re, time, urllib.parse
from DrissionPage import ChromiumPage, ChromiumOptions


def scrape_58_rent(city_code, area_name):
    """58同城租房数据

    Args:
        city_code: 城市代码 (nj/bj/sh/gz/sz/hz/cd/wh)
        area_name: 区域名（如"奥体"、"天通苑"）

    Returns:
        list[dict]: [{price, layout, area, community, floor, decoration}]
    """
    co = ChromiumOptions()
    co.headless(True)
    co.set_argument('--disable-blink-features=AutomationControlled')
    co.set_user_agent('Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36')

    page = ChromiumPage(co)
    results = []

    try:
        kw = urllib.parse.quote(area_name)
        url = f'https://{city_code}.58.com/zufang/?keyword={kw}'
        page.get(url)
        time.sleep(4)

        items = page.eles('css:.house-cell', timeout=5)

        for item in items[:20]:
            try:
                text = item.text.strip()
                if len(text) < 10:
                    continue

                price = ''
                m = re.search(r'(\d{3,5})\s*元/月', text)
                if m: price = int(m.group(1))

                layout = ''
                m = re.search(r'(\d室\d厅)', text)
                if m: layout = m.group(1)

                area = ''
                m = re.search(r'(\d+\.?\d*)\s*[㎡平米]', text)
                if m: area = m.group(1) + '㎡'

                community = ''
                lines = text.split('\n')
                for line in lines[:3]:
                    line = line.strip()
                    if 2 < len(line) < 20 and not re.match(r'^\d', line):
                        community = line
                        break

                floor = ''
                m = re.search(r'(低层|中层|高层)\(?\d*层?\)?', text)
                if m: floor = m.group(0)

                decoration = ''
                if '精装' in text: decoration = '精装'
                elif '简装' in text: decoration = '简装'

                if price:
                    results.append({
                        'price': price, 'layout': layout, 'area': area,
                        'community': community, 'floor': floor, 'decoration': decoration,
                        'source': '58同城',
                    })
            except Exception:
                continue
    finally:
        page.quit()

    return results
```

## 链家抓取

```python
def scrape_lianjia_rent(city_code, area_name=''):
    """链家租房数据

    注意：链家搜索特定区域需要登录，不带关键词的通用列表可访问。
    建议先尝试带关键词的URL，失败则用通用列表。

    Args:
        city_code: 城市代码 (nj/bj/sh/gz/sz/hz/cd/wh)
        area_name: 区域名（可选）

    Returns:
        list[dict]: [{price, layout, area, source}]
    """
    co = ChromiumOptions()
    co.headless(True)
    co.set_argument('--disable-blink-features=AutomationControlled')

    page = ChromiumPage(co)
    results = []

    try:
        kw = urllib.parse.quote(area_name) if area_name else ''
        urls = []
        if kw:
            urls.append(f'https://{city_code}.lianjia.com/zufang/rs{kw}/')
        urls.append(f'https://{city_code}.lianjia.com/zufang/')

        for url in urls:
            page.get(url)
            time.sleep(3)

            if '登录' in page.title or '验证' in page.title:
                continue

            items = page.eles('css:.content__list--item', timeout=3)
            if not items:
                items = page.eles('css:.list_item', timeout=3)

            if items:
                for item in items[:20]:
                    try:
                        text = item.text.strip()
                        m = re.search(r'(\d{3,5})\s*元/月', text)
                        price = int(m.group(1)) if m else None

                        m = re.search(r'(\d室\d厅)', text)
                        layout = m.group(1) if m else ''

                        m = re.search(r'(\d+\.?\d*)\s*[㎡平米]', text)
                        area = m.group(1) + '㎡' if m else ''

                        if price:
                            results.append({
                                'price': price, 'layout': layout, 'area': area,
                                'source': '链家',
                            })
                    except Exception:
                        continue
                break

    finally:
        page.quit()

    return results
```

## 平台对比

| 平台 | 数据量 | 价格 | 户型 | 面积 | 小区名 | 楼层 | 装修 | 特点 |
|:---|:---|:---|:---|:---|:---|:---|:---|:---|
| 安居客 | ⭐⭐⭐ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | 主力数据源 |
| 58同城 | ⭐⭐⭐ | ✅ | ⚠️少 | ✅ | ✅ | ✅ | ✅ | 个人房源多，合租信息丰富 |
| 链家 | ⭐⭐ | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | 数据规范，但搜区域需登录 |

## 使用策略

```
1. 安居客作为主力数据源（已有 anjuke-scraper.md）
2. 58同城作为补充（个人房源、合租信息）
3. 链家作为校验（数据规范，可验证价格合理性）
4. 三个平台数据合并去重，取交集作为高可信度数据
```

## 注意事项

- 所有平台都可能更新页面结构，选择器需定期检查
- 搜索频率：每次间隔 ≥ 5 秒
- 链家搜索特定区域需要登录，降级用通用列表
- 抓取失败时降级使用 references/rental-reference.md 静态数据
