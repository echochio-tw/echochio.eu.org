---
layout: post
title: playwright Pyppeteer (selenium) 測試
date: 2024-03-19
tags: playwright Pyppeteer selenium
---
在不同的瀏覽器上進行快速、可靠的測試，並且看重最新的功能，選擇 Playwright。

總結來說，對於專註於Chrome的網頁抓取和快速自動化，請選擇Puppeteer(Pyppeteer)。 

瀏覽器兼容性和多語言支持是首要任務，Selenium是最好的選擇

_posts/2007-11-05-me.md

這邊看看 Playwright
```
python -m playwright install

playwright install

python -m playwright codegen --target python -o 'my.py' -b chromium https://www.google.com
```

關閉用
```

    page.wait_for_timeout(10000)
    page.close()
    # ---------------------
    context.close()
    browser.close()
```


pyppeteer
```
pip install pyppeteer
```

網路上的範例 看都是用 asyncio
```
import asyncio
from pyppeteer import launch
from bs4 import BeautifulSoup

async def main():
    # 開啟瀏覽器程序
    browser = await launch()
    # 用瀏覽器連到 https://pythonclock.org/ 網站
    page = await browser.newPage()
    await page.goto('https://pythonclock.org/')    

    html_doc = await page.content()
    soup = BeautifulSoup(html_doc, 'lxml')

    await browser.close()

    print(soup.find('div', class_='python-27-clock'))

asyncio.set_event_loop(asyncio.new_event_loop())
asyncio.get_event_loop().run_until_complete(main())
```


```
import asyncio
from pyppeteer import launch
from pyquery import PyQuery as pq
 
async def main():
    browser = await launch()
    page = await browser.newPage()
    await page.goto('http://quotes.toscrape.com/js/')
    doc = pq(await page.content())
    print('Quotes:', doc('.quote').length)
    await browser.close()
 
asyncio.get_event_loop().run_until_complete(main())
```

