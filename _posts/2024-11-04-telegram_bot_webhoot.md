---
layout: post
title: telegram bot webhoot 方式紀錄所加入的該群內所有訊息
date: 2024-11-04
tags: telegram bot
---

You: 

請用python 幫我寫一個使用 telegram bot webhoot 方式紀錄所加入的該群內所有訊息的程序

ChatGPT: 

要在 Telegram Bot 中使用 Webhook 方式來紀錄群組中的所有訊息，你可以使用 Python 搭配 `Flask` 來建立一個 Web 伺服器，並利用 Telegram 的 Webhook 功能來接收群組中的訊息。
以下是實作步驟：
 
1. **申請 Telegram Bot API Token** ：首先透過 [BotFather]()  申請一個新的 Bot 並取得 Token。
 
2. **設定 Webhook** ：將 Bot 的 Webhook 指向你的伺服器，這樣 Telegram 就會將所有訊息推送到該 Webhook URL。
 
3. **建立 Flask 應用程式** ：透過 Flask 來接收訊息，並將訊息記錄下來。

以下是完整的程式碼範例：

### 1. 安裝所需的套件 


```bash
pip install flask requests
```

### 2. 寫一個 Flask 應用程式來接收並記錄訊息 


```python
from flask import Flask, request
import requests
import json
import logging

# 設定 Telegram Bot Token 和 Webhook URL
TELEGRAM_BOT_TOKEN = 'your_bot_token_here'
WEBHOOK_URL = f'https://api.telegram.org/bot{TELEGRAM_BOT_TOKEN}/setWebhook'
WEBHOOK_HOST = 'https://your_server_domain_or_ip'
WEBHOOK_PATH = '/telegram_webhook'
WEBHOOK_FULL_URL = WEBHOOK_HOST + WEBHOOK_PATH

app = Flask(__name__)

# 設定 logging 來記錄訊息
logging.basicConfig(level=logging.INFO, filename='telegram_messages.log', 
                    format='%(asctime)s - %(message)s')

def set_webhook():
    """設定 Telegram Webhook"""
    response = requests.post(WEBHOOK_URL, json={'url': WEBHOOK_FULL_URL})
    if response.status_code == 200:
        print("Webhook 設定成功")
    else:
        print(f"設定 Webhook 失敗: {response.text}")

@app.route(WEBHOOK_PATH, methods=['POST'])
def webhook():
    """處理來自 Telegram 的 Webhook"""
    data = request.get_json()
    
    # 檢查訊息是否存在
    if 'message' in data:
        chat_id = data['message']['chat']['id']
        user = data['message']['from']['username']
        text = data['message'].get('text', '')

        # 記錄訊息
        logging.info(f"Chat ID: {chat_id} | User: {user} | Message: {text}")
        
    return 'OK'

if __name__ == '__main__':
    set_webhook()  # 設定 Webhook
    app.run(host='0.0.0.0', port=5000)
```

### 3. 說明程式碼 
 
- **Webhook 設定** ：透過 `set_webhook()` 函數將 Webhook 設定到你的伺服器上，讓 Telegram 能將訊息推送到 `WEBHOOK_PATH` 指定的 URL 上。
 
- **Webhook 處理** ：當群組中有新訊息時，Telegram 會將訊息以 JSON 格式傳送到該 Webhook URL。程式會擷取 `message` 物件中的 `chat_id`、`username` 和 `text`，並記錄在 `telegram_messages.log` 檔案中。

### 4. 運行 Flask 應用 

在伺服器上執行 Flask 應用：


```bash
python app.py
```

確保伺服器已啟動且在防火牆中開放對應的端口 (如 5000)，以便 Telegram 伺服器能夠訪問你的 Webhook。

### 注意事項 

- 確保你的伺服器支援 HTTPS，Telegram 只允許 HTTPS 的 Webhook URL。
 
- 如果需要讓程式在伺服器上長時間運行，建議使用 `gunicorn` 或將 Flask 應用放在 Docker 或其他 Web 伺服器中，以便更穩定地執行。

You: 

把這個程式碼改成記錄到資料庫中

ChatGPT: 

要將接收到的 Telegram 訊息記錄到資料庫中，可以使用 SQLite 或其他資料庫，例如 MySQL 或 PostgreSQL。以下的程式碼使用 SQLite，因為它簡單且不需要額外的伺服器設定。程式會在接收到訊息時，將訊息儲存到一個 SQLite 資料庫。

以下是具備資料庫功能的程式碼：

### 1. 安裝所需的套件 


```bash
pip install flask requests sqlite3
```
`sqlite3` 是 Python 標準庫的一部分，不需要另外安裝。
### 2. 修改程式碼以將訊息記錄到資料庫 


```python
from flask import Flask, request
import requests
import sqlite3
import json

# 設定 Telegram Bot Token 和 Webhook URL
TELEGRAM_BOT_TOKEN = 'your_bot_token_here'
WEBHOOK_URL = f'https://api.telegram.org/bot{TELEGRAM_BOT_TOKEN}/setWebhook'
WEBHOOK_HOST = 'https://your_server_domain_or_ip'
WEBHOOK_PATH = '/telegram_webhook'
WEBHOOK_FULL_URL = WEBHOOK_HOST + WEBHOOK_PATH

app = Flask(__name__)

# 初始化資料庫連接
def init_db():
    """初始化資料庫並創建 messages 表格"""
    conn = sqlite3.connect('messages.db')
    cursor = conn.cursor()
    cursor.execute('''CREATE TABLE IF NOT EXISTS messages (
                        id INTEGER PRIMARY KEY AUTOINCREMENT,
                        chat_id INTEGER NOT NULL,
                        user TEXT,
                        message TEXT,
                        timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
                    )''')
    conn.commit()
    conn.close()

# 設定 Webhook
def set_webhook():
    """設定 Telegram Webhook"""
    response = requests.post(WEBHOOK_URL, json={'url': WEBHOOK_FULL_URL})
    if response.status_code == 200:
        print("Webhook 設定成功")
    else:
        print(f"設定 Webhook 失敗: {response.text}")

@app.route(WEBHOOK_PATH, methods=['POST'])
def webhook():
    """處理來自 Telegram 的 Webhook"""
    data = request.get_json()
    
    # 檢查訊息是否存在
    if 'message' in data:
        chat_id = data['message']['chat']['id']
        user = data['message']['from'].get('username', 'unknown')  # 如果沒有使用者名稱，預設為 unknown
        text = data['message'].get('text', '')

        # 將訊息儲存到資料庫
        save_message(chat_id, user, text)
        
    return 'OK'

def save_message(chat_id, user, text):
    """儲存訊息到 SQLite 資料庫"""
    conn = sqlite3.connect('messages.db')
    cursor = conn.cursor()
    cursor.execute("INSERT INTO messages (chat_id, user, message) VALUES (?, ?, ?)",
                   (chat_id, user, text))
    conn.commit()
    conn.close()

if __name__ == '__main__':
    init_db()     # 初始化資料庫
    set_webhook() # 設定 Webhook
    app.run(host='0.0.0.0', port=5000)
```

### 3. 說明程式碼 
 
- **資料庫初始化** ：在 `init_db()` 函數中，創建了名為 `messages` 的表格，包含以下欄位： 
  - `id`: 自動遞增的主鍵。
 
  - `chat_id`: 傳送訊息的聊天 ID。
 
  - `user`: 發送訊息的使用者名稱。
 
  - `message`: 訊息的文字內容。
 
  - `timestamp`: 訊息的接收時間，預設為當前時間。
 
- **紀錄訊息** ：`save_message()` 函數會將訊息儲存到 `messages` 表格中。每當接收到 Telegram 的 Webhook 請求，`webhook()` 函數會解析出 `chat_id`、`user` 和 `text`，然後呼叫 `save_message()` 儲存這些資訊。

### 4. 運行應用 

在伺服器上運行 Flask 應用：


```bash
python app.py
```

### 注意事項 

- 如果將此程式部署在生產環境，考慮使用更成熟的資料庫，如 MySQL 或 PostgreSQL。
 
- 如果伺服器使用 HTTPS，請確保 `WEBHOOK_FULL_URL` 指向正確的 HTTPS 網址。
 
- 可以使用 SQL 管理工具或命令行查詢 `messages.db` 資料庫中的訊息記錄。

You: 

改程式為記錄到 mariadb 中

ChatGPT: 

要將接收到的 Telegram 訊息記錄到 MariaDB 中，需先安裝 MariaDB 的 Python 客戶端，並將資料庫初始化及儲存訊息的部分改成連接 MariaDB。

以下是使用 MariaDB 的完整程式碼：

### 1. 安裝所需的套件 


```bash
pip install flask requests mysql-connector-python
```

### 2. 建立資料庫和表格 

首先，進入 MariaDB，並建立資料庫和表格。


```sql
CREATE DATABASE telegram_bot;
USE telegram_bot;

CREATE TABLE messages (
    id INT AUTO_INCREMENT PRIMARY KEY,
    chat_id BIGINT NOT NULL,
    user VARCHAR(255),
    message TEXT,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 3. 修改程式碼以將訊息記錄到 MariaDB 


```python
from flask import Flask, request
import requests
import mysql.connector
import json

# 設定 Telegram Bot Token 和 Webhook URL
TELEGRAM_BOT_TOKEN = 'your_bot_token_here'
WEBHOOK_URL = f'https://api.telegram.org/bot{TELEGRAM_BOT_TOKEN}/setWebhook'
WEBHOOK_HOST = 'https://your_server_domain_or_ip'
WEBHOOK_PATH = '/telegram_webhook'
WEBHOOK_FULL_URL = WEBHOOK_HOST + WEBHOOK_PATH

app = Flask(__name__)

# 資料庫連線設定
DB_CONFIG = {
    'host': 'localhost',
    'user': 'your_db_user',
    'password': 'your_db_password',
    'database': 'telegram_bot'
}

# 設定 Webhook
def set_webhook():
    """設定 Telegram Webhook"""
    response = requests.post(WEBHOOK_URL, json={'url': WEBHOOK_FULL_URL})
    if response.status_code == 200:
        print("Webhook 設定成功")
    else:
        print(f"設定 Webhook 失敗: {response.text}")

@app.route(WEBHOOK_PATH, methods=['POST'])
def webhook():
    """處理來自 Telegram 的 Webhook"""
    data = request.get_json()
    
    # 檢查訊息是否存在
    if 'message' in data:
        chat_id = data['message']['chat']['id']
        user = data['message']['from'].get('username', 'unknown')  # 如果沒有使用者名稱，預設為 unknown
        text = data['message'].get('text', '')

        # 將訊息儲存到資料庫
        save_message(chat_id, user, text)
        
    return 'OK'

def save_message(chat_id, user, text):
    """儲存訊息到 MariaDB 資料庫"""
    conn = mysql.connector.connect(**DB_CONFIG)
    cursor = conn.cursor()
    try:
        cursor.execute("INSERT INTO messages (chat_id, user, message) VALUES (%s, %s, %s)",
                       (chat_id, user, text))
        conn.commit()
    except mysql.connector.Error as err:
        print(f"資料庫錯誤: {err}")
    finally:
        cursor.close()
        conn.close()

if __name__ == '__main__':
    set_webhook() # 設定 Webhook
    app.run(host='0.0.0.0', port=5000)
```

### 4. 說明程式碼 
 
- **資料庫連線設定** ：使用 `DB_CONFIG` 字典來儲存資料庫的連線資訊，包括主機、使用者名稱、密碼和資料庫名稱。
 
- **紀錄訊息** ：`save_message()` 函數會連接到 MariaDB，並將 `chat_id`、`user` 和 `message` 插入 `messages` 表格中。
 
- **例外處理** ：在 `save_message()` 中加入了例外處理，以防資料庫連線或 SQL 執行發生錯誤。

### 5. 運行應用 

在伺服器上運行 Flask 應用：


```bash
python app.py
```

### 注意事項 
 
- 確保你的 MariaDB 資料庫使用者具有對 `telegram_bot` 資料庫的讀寫權限。
 
- 如果程式長時間執行，考慮增加連線池（例如 `mysql.connector.pooling`）來提升效能。

- 建議在生產環境中使用 HTTPS，以確保 Webhook 的安全。

You: 

將flask 程式配合  WSGI 及 Nginx 服務

ChatGPT: 

要將 Flask 應用程式與 WSGI 和 Nginx 配合，通常會使用 `gunicorn` 作為 WSGI 伺服器，並使用 Nginx 作為反向代理。以下是詳細的部署步驟：
### 1. 安裝所需套件 

首先，確保伺服器已經安裝好 Flask 應用程式所需的套件。


```bash
pip install flask requests mysql-connector-python gunicorn
```

### 2. 建立 WSGI 入口檔案 
建立一個 WSGI 入口檔案，通常稱為 `wsgi.py`，來啟動 Flask 應用程式。在應用目錄中新增一個 `wsgi.py` 檔案：

```python
from app import app

if __name__ == "__main__":
    app.run()
```

> 注意：`app` 是我們在主程式中創建的 Flask 應用實例。如果應用實例名稱不同，請相應修改。
### 3. 使用 Gunicorn 啟動 Flask 應用 
使用 `gunicorn` 啟動 Flask 應用程式。可以先在終端測試以下命令，確認應用程式啟動正常：

```bash
gunicorn --bind 127.0.0.1:8000 wsgi:app
```
這將啟動一個綁定在 `127.0.0.1:8000` 的 `gunicorn` 伺服器。
### 4. 設定 Nginx 作為反向代理 
編輯 Nginx 的配置文件來處理外部請求並轉發給 `gunicorn`。在 `/etc/nginx/sites-available/` 中創建新的配置文件（例如 `telegram_bot`）：

```nginx
server {
    listen 80;
    server_name your_domain_or_ip;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # 對於 Telegram Webhook 使用的 URL
    location /telegram_webhook {
        proxy_pass http://127.0.0.1:8000/telegram_webhook;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

> **注意** ：請將 `your_domain_or_ip` 替換為你的伺服器域名或 IP。
#### 啟用 Nginx 配置並重新啟動 
將新的配置文件符號連接到 `/etc/nginx/sites-enabled/` 目錄：

```bash
sudo ln -s /etc/nginx/sites-available/telegram_bot /etc/nginx/sites-enabled
sudo nginx -t  # 檢查配置是否正確
sudo systemctl restart nginx  # 重新啟動 Nginx
```

### 5. 建立 Systemd 服務檔案（選用） 
為了讓 `gunicorn` 伺服器開機自動啟動，可以創建一個 Systemd 服務檔案。創建並編輯 `/etc/systemd/system/telegram_bot.service`：

```ini
[Unit]
Description=Gunicorn instance to serve telegram bot
After=network.target

[Service]
User=your_user
Group=www-data
WorkingDirectory=/path/to/your/app
ExecStart=/usr/local/bin/gunicorn --workers 3 --bind 127.0.0.1:8000 wsgi:app

[Install]
WantedBy=multi-user.target
```

> 將 `your_user` 替換為執行應用程式的伺服器使用者名稱。`WorkingDirectory` 應設為 Flask 應用程式的根目錄。`ExecStart` 應使用 Gunicorn 的完整路徑，可以用 `which gunicorn` 查找。
#### 啟用並啟動服務 


```bash
sudo systemctl start telegram_bot
sudo systemctl enable telegram_bot
```
這樣，當伺服器重啟時，`gunicorn` 伺服器會自動啟動並提供應用程式服務。
### 6. 測試 Webhook 

設定好 Nginx 和 Gunicorn 後，可以再次執行 Webhook 設定程式，並確保 Telegram 的 Webhook 能夠正確地將訊息傳遞給應用程式：


```python
# 在 app.py 中的 Webhook 設定
set_webhook()
```

### 總結 
此時，Nginx 會處理外部請求，並將流量轉發到 `gunicorn` 執行的 Flask 應用程式。這樣的設置可以提高穩定性與效率，使應用程式在生產環境中能穩定運行。
