---
layout: post
title: GCP 爬蟲抓費用
date: 2026-01-14
tags: GCP
---
GCP 爬蟲抓費用，其中 D:\chrome_temp 有google 登入紀錄 (可以加 time.sleep)

(google 登入的紀錄。要改成自己去生成)
```
import sys
import time
import re
import requests  # 新增：用於發送 API 請求
from playwright.sync_api import Playwright, sync_playwright

# ================= 設定區 =================
USER_DATA_DIR = r"C:\chrome_temp"
TARGET_URL = "https://console.cloud.google.com/billing/overview?project=atomi"
PROJECT_ID = "atomic-elixir-435115-h0"

# --- Telegram Bot 設定 ---
TELEGRAM_BOT_TOKEN = "610418723:XXXXXXXXXXXXXRDwM8x7s"
TELEGRAM_CHAT_ID = "-358XXXX"  # 房間 ID
# =========================================
def log_step(msg):
    with open(r"C:\python\debug_log.txt", "a", encoding="utf-8") as f:
        f.write(f"{time.strftime('%Y-%m-%d %H:%M:%S')} - {msg}\n")

def send_telegram_photo(photo_path, caption=""):
    """使用 Bot API 發送照片到 Telegram"""
    url = f"https://api.telegram.org/bot{TELEGRAM_BOT_TOKEN}/sendPhoto"
    try:
        with open(photo_path, "rb") as photo:
            payload = {
                "chat_id": TELEGRAM_CHAT_ID,
                "caption": caption
            }
            files = {
                "photo": photo
            }
            response = requests.post(url, data=payload, files=files)
            if response.status_code == 200:
                log_step("✅ 截圖已成功發送到 Telegram")
            else:
                log_step(f"❌ 發送失敗，錯誤碼：{response.status_code}, 內容：{response.text}")
    except Exception as e:
        log_step(f"⚠️ 發送過程中發生異常: {e}")

def run(playwright: Playwright) -> None:
    log_step("🚀 嘗試啟動 Chrome (Persistent Context)...")
    try:
        context = playwright.chromium.launch_persistent_context(
            user_data_dir=USER_DATA_DIR,
            headless=True, # 排程環境建議headless
            # headless=False,　# 如果要認證google 可以加手動輸入
            handle_sigint=False, # 排程環境建議關閉信號處理
            handle_sigterm=False,
            handle_sighup=False,
            args=[
                "--disable-blink-features=AutomationControlled",
                "--no-sandbox",          # 必須：Session 0 安全沙箱常導致掛起
                "--disable-gpu",          # 必須：後台環境無顯卡加速
                "--disable-dev-shm-usage", # 必須：防止資源限制導致崩潰
                "--single-process"       # 可選：在受限環境中有時更穩定
            ],
            ignore_default_args=["--enable-automation"],
            viewport={'width': 1920, 'height': 1080},
            #no_viewport=True
        )
        log_step("✅ Chrome 啟動成功")
　　　　# time.sleep(600) # 如果要認證google 可以加手動輸入　
    except Exception as e:
        log_step(f"❌ Chrome 啟動失敗: {e}")
        return
    
    page = context.new_page()
    screenshot_path = r"C:\python\billing_exact_check.png"
    
    try:
        log_step(f"🔗 前往頁面: {TARGET_URL}")
        page.goto(TARGET_URL)
        log_step("🖱️ 正在導航至總覽頁面...")
# 嘗試多種方式點擊，並加入顯式等待
        try:
            # 1. 優先嘗試直接點擊文字按鈕 (加上等待)
            page.wait_for_selector('text="前往「總覽」頁面"', timeout=5000)
            page.click('text="前往「總覽」頁面"')
            log_step("   ✅ 直接點擊成功")
        except:
            try:
                # 2. 針對截圖中看到的藍色連結樣式進行定位
                # 使用 XPath 尋找包含特定文字的 <a> 標籤
                target = page.locator('a:has-text("前往「總覽」頁面")')
                if target.count() > 0:
                    target.first.click()
                    log_step("   ✅ 透過連結定位點擊成功")
                else:
                    # 3. 如果在 iframe 內，強制對所有 frame 進行搜索
                    clicked = False
                    for frame in page.frames:
                        btn = frame.locator('text="前往「總覽」頁面"')
                        if btn.count() > 0:
                            btn.first.click()
                            log_step(f"   ✅ 在 iframe [{frame.name}] 中點擊成功")
                            clicked = True
                            break
                    if not clicked:
                        log_step("   ⚠️ 找不到按鈕，將嘗試直接截圖目前畫面")
            except Exception as e:
                log_step(f"   ❌ 點擊發生錯誤: {e}")
        log_step("⏳ 等待數據載入 (20秒)...")
        time.sleep(20)

        try:
            remind_btn = page.locator('button:has-text("之後提醒我")')
            if remind_btn.is_visible():
                remind_btn.click()
        except:
            pass

        # 截圖
        page.screenshot(path=screenshot_path, full_page=True)
        log_step(f"📸 截圖已儲存: {screenshot_path}")

        # --- 新增：發送到 Telegram ---
        send_telegram_photo(screenshot_path, caption=f"📊 GCP 帳單截圖\nProject: {PROJECT_ID}")

    finally:
        log_step("關閉瀏覽器...")
        context.close()

if __name__ == "__main__":
    try:
        with sync_playwright() as playwright:
            run(playwright)
        log_step("🏁 腳本完全結束")
        sys.exit(0)  # 確保回傳結束信號給工作排程器
    except Exception as e:
        log_step(f"❌ 主程式異常: {e}")
        sys.exit(1)
```
