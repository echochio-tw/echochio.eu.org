---
layout: post
title: python Playwright 来debug
date: 2025-04-20
tags: python Playwright
---

Playwright 中可用來監控/控制網路活動的方式（不寫 HAR）可以问AI

page.on("request") / page.on("response") — 即時監控網路請求與回應
```
def log_request(request):
    print(f">> Request: {request.method} {request.url}")

def log_response(response):
    print(f"<< Response: {response.status} {response.url}")

page.on("request", log_request)
page.on("response", log_response)
```
route — 攔截、修改、模擬 API 回應
```
def handle_route(route, request):
    if "api/data" in request.url:
        route.fulfill(
            status=200,
            content_type="application/json",
            body='{"fake": "data"}'
        )
    else:
        route.continue_()

context.route("**/*", handle_route)
```
request.post_data() / response.body() — 直接抓內容
```
page.on("request", lambda req: print(req.post_data()))
page.on("response", lambda res: print(res.status, res.body()))

```
使用 DevTools Protocol（高階技巧）
```
cdp_session = context.new_cdp_session(page)
await cdp_session.send("Network.enable")
await cdp_session.on("Network.requestWillBeSent", lambda e: print(e))
```
#在 page.goto(url) 跳轉新頁面後，等待所有資源和動態數據加載完成再進行後續操作
```
page.wait_for_load_state("networkidle")
```
#获取 Playwright 的 Cookies
```
playwright_cookies = context.cookies()
```
#监听所有请求
```
from playwright.sync_api import sync_playwright
with sync_playwright() as p:
    browser = p.chromium.launch(headless=False)
    ctx = browser.new_context()
    page = ctx.new_page()
```

# 监听所有请求，打印出 method、url 和 body
```
def log_every_request(request):
    body = request.post_data or "<空>"
    print(f"{request.method} {request.url}\n▶ Body: {body}\n")
page.on("request", log_every_request)
# 然后你的正常流程
page.goto("https://158.test.vip/login.aspx")
```

#當程式執行到 page.pause() 這一行時，會自動彈出 Playwright Inspector 工具視窗，並暫停腳本執行
#你可以在 Inspector 介面中點擊「繼續」或「單步執行」來恢復或逐步執行後續腳本


用 record_trace
```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    context = browser.new_context(record_video_dir="videos/", record_har_path="har.json", record_trace_dir="traces/")  # trace儲存
    page = context.new_page()
    page.goto("https://example.com")
    page.click("text=More information")  # 根據實際內容調整
    context.close()
    browser.close()
```

	這樣會在 `traces/` 目錄產生 trace 資料。
	
```bash
python -m playwright show-trace traces/trace.zip
```

- 如果你手動收集 trace，可以用 `context.tracing.start()` 和 `context.tracing.stop(path="trace.zip")` 來控制。
