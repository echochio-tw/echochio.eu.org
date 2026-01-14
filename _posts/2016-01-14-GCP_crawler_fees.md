---
layout: post
title: GCP 爬蟲抓費用
date: 2026-01-14
tags: GCP
---
google 登入的紀錄。要改成自己去生成
```
cd D:
mkdir chrome_temp 
"C:\Program Files\Google\Chrome\Application\chrome.exe" "https://console.cloud.google.com/"  --user-data-dir="D:\chrome_temp"
```

GCP 爬蟲抓費用，其中 D:\chrome_temp 有google 登入紀錄
```
import time
import re
from playwright.sync_api import Playwright, sync_playwright

# ================= 設定區 =================
USER_DATA_DIR = r"D:\chrome_temp"
TARGET_URL = "https://console.cloud.google.com/billing/overview?project=ez1022"
# =========================================

def run(playwright: Playwright) -> None:
    print("🚀 啟動 Chrome...")
    # 這裡加入 args 避開一些偵測與加速
    context = playwright.chromium.launch_persistent_context(
        user_data_dir=USER_DATA_DIR,
        channel="chrome",
        headless=False,
        no_viewport=True # 使用視窗原始大小
    )
    
    page = context.new_page()
    
    try:
        print(f"🔗 前往頁面: {TARGET_URL}")
        page.goto(TARGET_URL)

        # 1. 執行你錄製的第一個動作：點擊主頁面的「前往總覽」
        print("🖱️ 動作 1: 點擊主頁面『前往總覽』...")
        try:
            page.get_by_role("link", name="前往「總覽」頁面").wait_for(state="visible", timeout=10000)
            page.get_by_role("link", name="前往「總覽」頁面").click()
        except Exception as e:
            print(f"   跳過動作 1 (可能不存在): {e}")

        # 2. 執行你錄製的第二個動作：點擊 iframe 內的「前往總覽」
        print("🖱️ 動作 2: 點擊 iframe 內的『前往總覽』...")
        try:
            # 這是你錄製到的關鍵定位：#google-feedback-rif
            target_frame = page.locator("#google-feedback-rif").content_frame
            target_frame.get_by_role("link", name="前往「總覽」頁面").wait_for(state="visible", timeout=5000)
            target_frame.get_by_role("link", name="前往「總覽」頁面").click()
            print("   ✅ iframe 點擊成功")
        except Exception as e:
            print(f"   跳過動作 2 (可能不存在): {e}")

        # 3. 關鍵等待：等待頁面網址變動或特定內容出現
        print("⏳ 等待帳單正式數據載入 (20秒)...")
        time.sleep(20) 

        # 4. 處理彈窗：點擊「之後提醒我」
        # 4. 處理彈窗：針對 Angular Mat-Button 的精確點擊
        print("🔍 正在嘗試精確定位『之後提醒我』按鈕...")
        try:
            # 策略 A: 使用 Playwright 的文字定位 (針對 mdc-button__label)
            # 這種寫法會自動掃描所有層級，包括 shadow DOM
            remind_btn = page.get_by_role("button", name="之後提醒我")
            
            # 策略 B: 如果 A 不夠強，使用屬性選擇器 (針對你提供的 jslog 或 class)
            if not remind_btn.is_visible():
                remind_btn = page.locator('button.cm-button:has-text("之後提醒我")')

            # 策略 C: 遍歷所有 Frame (防禦 iframe 隔離)
            if not remind_btn.is_visible():
                for frame in page.frames:
                    target = frame.locator('button:has-text("之後提醒我")')
                    if target.is_visible():
                        print(f"   🎯 在 Frame [{frame.name}] 中尋獲按鈕")
                        remind_btn = target
                        break

            # 執行點擊
            if remind_btn.is_visible(timeout=5000):
                remind_btn.click(force=True) # force=True 確保即使被遮擋也強制觸發
                print("   ✅ 已成功點擊『之後提醒我』")
            else:
                print("   💡 未發現彈窗，可能本次未出現。")

        except Exception as e:
            print(f"   ⚠️ 點擊失敗，嘗試最後手段 (Esc): {e}")
            page.keyboard.press("Escape")

        # 5. 抓取費用數據
# 📊 提取費用數據
        print("📊 正在提取費用數據...")
        try:
            # 1. 增加一點緩衝，確保數字跑完
            page.wait_for_timeout(3000)

            # 2. 直接找包含 $ 且旁邊有「費用」字眼的區塊
            # 我們改用 XPath，這在 GCP 這種混亂的頁面通常比 CSS 穩定
            # 意思是：尋找包含「費用」的元素，並抓取它後面的同級或子級文字
            cost_locator = page.locator('//div[contains(text(), "費用")]/following-sibling::div').first
            
            # 3. 備案：如果上面抓不到，直接抓頁面上所有「看起來像錢」的文字
            if not cost_locator.is_visible():
                # 使用正則運算式尋找 $ 開頭的數字 (例如 $593.00)
                # 使用 r"" (raw string) 解決你看到的 SyntaxWarning
                cost_locator = page.get_by_text(re.compile(r"\$\d+[\d,.]*")).first

            cost_text = cost_locator.inner_text().strip()
            
            if cost_text and "$" in cost_text:
                print("\n" + "★"*30)
                print(f"💰 抓取成功！目前費用為: {cost_text}")
                print("★"*30 + "\n")
            else:
                # 終極手段：抓取整個卡片的內容再用正則過濾
                summary_card = page.locator(".p6n-billing-summary-card-content").inner_text()
                # 抓取包含 $ 的那一行
                money_match = re.search(r"\$[\d,.]+", summary_card)
                if money_match:
                    print(f"💰 終極手段抓取成功: {money_match.group()}")
                else:
                    print(f"❌ 抓到的內容不對 ({cost_text})，請手動確認畫面。")

        except Exception as e:
            print(f"❌ 提取發生錯誤: {e}")

    except Exception as e:
        print(f"❌ 程式執行出錯: {e}")
    
    finally:
        time.sleep(60)
        print("關閉瀏覽器...")
        context.close()

if __name__ == "__main__":
    with sync_playwright() as playwright:
        run(playwright)
```
