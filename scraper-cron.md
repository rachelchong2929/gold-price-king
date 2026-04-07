# 金價王 Cron Job — Daily Gold Price Scraper

**Cron ID:** `f4644422-23a2-45b5-b4f6-74ab851404b4`
**Schedule:** `0 9 * * *` Asia/Hong_Kong (daily 9am HKT)
**Timeout:** 300s

---

## Rules

- ONLY use official jeweller websites as PRIMARY source
- **NO FALLBACK** — do NOT use goldpricehk.com or any aggregator
- If official site fails or data cannot be extracted, mark as `null` (未有)
- ONLY extract SELL prices (賣出價) — NO buy prices
- VERIFY the date on the page matches TODAY before using the data
- Extract both 飾金賣出價 (`gold_sell`) and 金粒賣出價 (`gold_bar_sell`) per tael (両)

---

## Data Sources & XPath Selectors

### 六福 (Luk Fook)
- **URL:** https://www.lukfookeshop.com.hk/每日金價
- **飾金賣出價:** First (両) price in the 9999/999金 section (e.g. `HKD52,288`)
- **金粒賣出價:** In the `9999金粒` section, first (両) price is sell
- **金粒 XPath:** `//*[@id='lfg-container']/div/div[2]/div[6]/div[2]/div[2]/span[2]`

### 英皇 (Emperor)
- **URL:** https://www.emperorwatchjewellery.com/zh/gold-price/
- **飾金賣出價:** `足金飾品` → `賣出價 (HK$)` value
- **金粒賣出價:** `足金金粒` → `賣出價 (HK$)` value
- **金粒 XPath:** `//*[@id='bodyWrapper']/section[1]/div[3]/div[2]/div/div[1]/p[2]/text()`

### 周生生 (Chow Sang Sang)
- **URL:** https://www.chowsangsang.com/tc/gold-price
- **飾金賣出價:** JS-rendered — `web_fetch` shows `-` placeholders → mark `null`
- **金粒賣出價:** JS-rendered → mark `null`
- **金粒 XPath:** `//*[@id='hk-G_BAR_BUY']`

### 周大福 (Chow Tai Fook)
- **URL:** https://www.chowtaifook.com/zh-hk/eshop/realtime-gold-price.html
- **飾金賣出價:** JS-rendered — `web_fetch` shows template vars → mark `null`
- **金粒賣出價:** JS-rendered → mark `null`
- **金粒 XPath:** `//*[@id='maincontent']/div[1]/div/div/div/div[1]/div/div[1]/div[3]/div[1]/table/tbody/tr[5]/td[2]/div[2]`

---

## Parse Logic

**六福** markdown pattern:
```
(両) HKD52,288 HKD42,332
```
First number = sell, second = buy. Same for 金粒 section.

**英皇** markdown clearly labels 賣出價 and 買入價. Extract 賣出價 numbers.

**周生生/周大福**: If web_fetch returns `-` or `${variables}`, set both `gold_sell` and `gold_bar_sell` to `null`.

---

## Output

Update `/home/node/aibot/gold-price-king/deploy/data/prices.json`:
```json
{
  "last_updated": "YYYY-MM-DD",
  "update_time": "HH:MM",
  "shops": [
    {
      "name": "六福",
      "name_en": "Luk Fook",
      "logo": "🏮",
      "url": "https://www.lukfookeshop.com.hk/每日金價",
      "gold_sell": 52288,
      "gold_bar_sell": 47310
    },
    ...
  ]
}
```

Then `git add -A && git commit && git push origin main`.
