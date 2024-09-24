---
layout: post
title: Python 錄製並重複 Android 手機動作
date: 2024-09-24
tags: Android
---
用 Python 錄製並重複 Android 手機動作

在 Android 手機上使用 Python 來錄製並重複動作，可以透過以下方法實現：
1. 使用 `adb` 命令`adb`（Android Debug Bridge）是一個工具，允許你與 Android 設備進行互動，包括模擬觸控、鍵盤事件、錄製螢幕等操作。
### 2. 自動化工具 

你可以使用一些基於 Python 的自動化工具來錄製和執行動作，例如：
 
- `uiautomator`：專門用於控制 Android 設備 UI 的工具。
 
- `Appium`：一個支持多平台（iOS、Android）的自動化測試框架。
 
- `MonkeyRunner`：Android SDK 自帶的工具，用來自動化 UI 操作。

### 示例步驟： 
1. 安裝 `adb` 和 `uiautomator`：首先確保已安裝 Android SDK 並能使用 `adb` 命令，然後可以安裝 `uiautomator` 的 Python 包：

```bash
pip install uiautomator2
```

#### 2. 連接 Android 設備： 
確保你的設備開啟了開發者模式，並允許 USB 調試。你可以使用 `adb devices` 來確認設備是否連接。
#### 3. 錄製動作： 
可以通過 `adb` 命令錄製設備的螢幕操作，然後使用 Python 腳本來重複這些操作。
錄製螢幕的指令：


```bash
adb shell screenrecord /sdcard/screen.mp4
```

#### 4. 使用 Python 控制設備： 
這裡是一個使用 `uiautomator2` 的簡單範例，來自動化點擊和滑動操作：

```python
import uiautomator2 as u2

# 連接設備
d = u2.connect('設備IP或USB')

# 點擊螢幕上特定坐標
d.click(300, 500)

# 滑動
d.swipe(300, 500, 300, 1000)

# 輸入文字
d.send_keys("Hello World")

# 打開應用
d.app_start("com.example.app")

# 等待特定元素出現並點擊
d(text="Click Me").click()

# 錄製的動作可以根據需求腳本化重複
```

#### 5. 重複操作： 

可以將這些動作腳本化，然後根據需求調用不同的動作序列來實現重複操作。

### 進一步擴展： 

如果需要更複雜的錄製與重複功能，你可以考慮使用 Appium 來進行完整的測試自動化，這樣能夠記錄更加複雜的應用內行為。

You: 

adb shell getevent 錄製並重複 Android 手機動作

------------------------------------------------------------------------------------------------------------------

使用 `adb shell getevent` 來錄製並重複 Android 手機的操作是一種更底層的方式，這會捕獲設備上的所有輸入事件，例如觸摸、按鍵等。這裡是具體的步驟：
### 1. 確認設備連接 
首先，確認設備是否已連接並可被 `adb` 檢測到：

```bash
adb devices
```

確保你已啟用開發者選項並允許 USB 調試。
2. 使用 `getevent` 錄製事件在 Android 設備上使用 `getevent` 來錄製操作事件。打開一個終端，輸入以下命令開始錄製：

```bash
adb shell getevent -t > /sdcard/events.txt
```
這會記錄你在設備上的所有操作，並將它們保存到 `/sdcard/events.txt`。`-t` 參數會為每個事件添加時間戳，方便稍後重放。
#### 停止錄製 
當你完成錄製時，按下 `Ctrl + C` 來結束錄製。
你可以通過以下命令將錄製的文件複製到電腦上：


```bash
adb pull /sdcard/events.txt
```

### 3. 重複操作 

錄製完成後，現在可以重放這些操作。

#### 方法一：直接重放事件 

你可以使用以下命令將錄製的事件文件重放到設備上：


```bash
adb shell cat /sdcard/events.txt | adb shell sendevent
```

這會按照錄製時的順序將所有事件重放回設備。

#### 方法二：使用 Python 來進行更靈活的重放 
你也可以使用 Python 來解析 `getevent` 的輸出，並重複這些事件，這樣可以實現更加靈活的控制。 
1. 首先，查看 `events.txt` 文件的格式。每行會記錄一個事件，格式如下：


```yaml
/dev/input/event2: 0003 0039 00000001
/dev/input/event2: 0003 0035 000003d3
/dev/input/event2: 0003 0036 00000780
/dev/input/event2: 0000 0000 00000000
```

這些數字代表了事件類型、代碼和值。

1. 使用 Python 腳本來解析這些事件並重放。

示例代碼（假設你對事件做了一些解析）：


```python
import subprocess
import time

def replay_event(event_file):
    with open(event_file, 'r') as f:
        for line in f:
            if line.strip():  # 確保不是空行
                # 解析事件行，提取設備、類型、代碼和數值
                parts = line.split()
                device = parts[0].strip(':')
                type_code = parts[1]
                code = parts[2]
                value = parts[3]

                # 使用 adb sendevent 重放事件
                sendevent_cmd = f'adb shell sendevent {device} {type_code} {code} {value}'
                subprocess.run(sendevent_cmd, shell=True)

                # 根據錄製的時間戳添加適當延遲（可選）
                time.sleep(0.01)  # 調整為適合的延遲

# 重放錄製的事件
replay_event('events.txt')
```

### 注意事項 
 
1. **不同設備有不同的事件輸入設備** ：設備上可能會有多個輸入設備（如觸摸屏、按鍵等），你需要確認正確的輸入設備文件（如 `/dev/input/event0`）。
 
2. **精確控制** ：`getevent` 會記錄大量低層次的事件，可能會包含噪音或無關的事件（如鍵盤按鍵等）。你可以對事件進行過濾，只保留與觸摸相關的事件來進行重放。
 
3. **兼容性問題** ：某些 Android 設備可能會對這種重放方式做出防範措施，或者不同的 Android 版本處理事件的方式不同。
這樣，你就可以使用 `adb shell getevent` 來錄製和重複操作了。如果需要更進一步的自動化控制，可能需要借助其他專門的自動化工具。


在 Android 上錄製并重放动作，通常需要使用一些工具，例如 ADB（Android Debug Bridge）和 Python 来自动化执行任务。这里是一种方法，可以使用 ADB 工具配合 Python 脚本录制并重放设备的触摸操作。

### 步骤： 

#### 1. 安装 ADB 

首先，你需要安装 ADB（Android Debug Bridge）工具。你可以通过以下方式安装：
 
- **Windows：**  你可以从 Android SDK 下载 ADB。
 
- **Mac/Linux：**  使用 Homebrew 或 Apt 安装： 
  - Mac: `brew install android-platform-tools`
 
  - Linux: `sudo apt-get install android-tools-adb`

#### 2. 通过 ADB 连接 Android 设备 

确保 USB 调试已在 Android 设备上启用。通过 USB 将设备连接到计算机，然后运行以下命令以验证是否连接成功：


```bash
adb devices
```

#### 3. 使用 ADB 录制屏幕上的操作 

你可以通过以下命令记录设备上的触摸操作：


```bash
adb shell screenrecord /sdcard/screenrecord.mp4
```
这将录制设备屏幕到 `/sdcard/screenrecord.mp4` 文件中。你可以指定文件路径和名称。
#### 4. 使用 ADB 获取触摸事件 

你还可以通过以下命令监控并捕捉设备的触摸事件：


```bash
adb shell getevent -lt
```

这会输出所有触摸事件的原始数据，包括屏幕触摸、按钮按下等操作。

#### 5. 用 Python 录制和重放触摸动作 

下面是一个用 Python 调用 ADB 命令来实现录制和重放触摸动作的示例：


```python
import os
import time

# 录制触摸动作
def record_events(output_file):
    print("Recording events...")
    os.system(f"adb shell getevent -lt > {output_file}")
    print("Recording completed!")

# 重放触摸动作
def replay_events(input_file):
    print("Replaying events...")
    with open(input_file, 'r') as f:
        for line in f:
            if "EV_SYN" not in line:  # 排除同步事件
                os.system(f"adb shell sendevent {line.strip()}")
            time.sleep(0.01)  # 控制每个事件之间的时间间隔
    print("Replay completed!")

# 示例用法
record_file = "touch_events.txt"

# 录制动作
record_events(record_file)

# 重放动作
replay_events(record_file)
```

### 解释： 
 
1. **录制动作：**  `record_events` 函数会使用 ADB 的 `getevent` 命令捕获设备的触摸事件，并将其保存到文件中。
 
2. **重放动作：**  `replay_events` 函数从文件中读取触摸事件，并使用 ADB 的 `sendevent` 命令将这些事件重放到设备上。

### 注意： 

- 使用 ADB 捕获和重放触摸事件可能因不同设备而有所不同。

- 重放的触摸事件必须精确，包括设备分辨率等因素。

- Android 版本的差异可能会影响事件的捕获和回放。

你可以通过这些步骤实现基本的动作录制和重放。如果需要更高级的自动化操作，也可以考虑使用 Appium 等自动化工具。

----------------------------------------------------------------------------------------------------------------

用 MonkeyRecorder 录制Android 动作, 重放Android 动作

**MonkeyRecorder**  是一个工具，可以帮助录制和重放 Android 设备上的用户操作。它是 **UI/Application Exerciser Monkey** （简称 Monkey）的一个增强版，Monkey 通常用于自动化测试应用，但 MonkeyRecorder 提供了图形化界面，可以更直观地录制并重放操作。以下是如何使用 **MonkeyRecorder**  录制和重放 Android 操作的步骤：
### 前置条件： 
 
1. **安装 ADB（Android Debug Bridge）** ： 
  - **Windows** : 从 Android SDK 安装 ADB。
 
  - **Mac/Linux** : 通过 Homebrew 或 apt 安装： 
    - Mac: `brew install android-platform-tools`
 
    - Linux: `sudo apt-get install android-tools-adb`
 
2. **启用 Android 设备的开发者模式和 USB 调试** ：
  - 在 Android 设备上启用 USB 调试（在“开发者选项”中）。
 
3. **下载并安装 MonkeyRecorder** ：
  - MonkeyRecorder 通常是与 Android SDK 一起提供的工具之一，但你也可以通过搜索获取更具体的安装和使用说明。

### 步骤 1：启动 ADB 并连接设备 

确保你的 Android 设备已连接并可以通过 ADB 识别。在终端或命令提示符下运行以下命令，以确认设备连接成功：


```bash
adb devices
```

设备连接成功后，将显示设备的 ID。

### 步骤 2：下载并使用 MonkeyRecorder 
 
1. **下载 MonkeyRecorder** ： 
  - 你可以从 [MonkeyRunner 和 MonkeyRecorder](https://developer.android.com/studio/test/monkeyrunner)  页面下载。
 
2. **启动 MonkeyRecorder** ：
  - 启动 MonkeyRecorder 应用，它通常提供一个 GUI 界面，方便你录制动作。
 
3. **录制操作** ：
  - 点击 "Start Record" 开始录制 Android 设备上的操作。你在设备上进行的每个操作（触摸、滚动、滑动等）都会被记录下来。
 
  - 完成后，点击 "Stop Record" 停止录制。录制的内容会保存为一个 MonkeyRunner 脚本，通常是 `.py` 格式。

### 步骤 3：回放录制的操作 
 
1. **MonkeyRunner 回放脚本** ：
  - 你可以通过 MonkeyRunner 工具来运行生成的 Python 脚本。
 
2. **运行回放** ：
使用以下命令运行回放脚本：

```bash
monkeyrunner recorded_script.py
```

这将模拟你录制的所有操作，并在设备上重新执行。

### 代码示例： 

当你使用 MonkeyRecorder 录制操作后，它会生成类似于以下的 Python 脚本：


```python
# MonkeyRunner Script Example
from com.android.monkeyrunner import MonkeyRunner, MonkeyDevice

# Connect to the device
device = MonkeyRunner.waitForConnection()

# Perform some actions (example)
device.touch(100, 200, 'DOWN_AND_UP')
MonkeyRunner.sleep(1)
device.drag((100, 200), (300, 400), 1, 10)

# More actions can be added based on your recording
```

### 扩展：使用 Python 结合 MonkeyRunner 进行自动化 

MonkeyRunner 还允许你编写自定义的 Python 脚本来自动化 Android 设备上的各种操作，如点击、滑动、输入文本等。你可以通过 Python 脚本模拟用户操作，并集成到更大的测试框架中。

### 总结： 

通过 MonkeyRecorder，用户可以轻松录制 Android 设备上的交互操作并使用 MonkeyRunner 工具回放这些操作。这种方式不仅适用于测试，还可以用于自动化操作。

如果需要进一步详细的配置或遇到问题，随时可以继续讨论！

--------------------------------------------------------------------------------------------------

使用 **Appium**  进行 Android 操作录制和重放可以通过 Python 语言来实现。Appium 是一个开源的自动化工具，支持多种平台（如 Android 和 iOS）的应用程序自动化测试。你可以通过 Appium 客户端在 Python 中编写脚本来录制并重放 Android 设备上的操作。
### 环境准备 
 
1. **安装 Appium** ： 
  - 使用 Node.js 包管理器（npm）安装 Appium：

```bash
npm install -g appium
```
 
2. **安装 Appium Python Client** ： 
  - 使用 `pip` 安装 Appium 的 Python 客户端：

```bash
pip install Appium-Python-Client
```
 
3. **安装 Appium Desktop（可选，用于录制动作）** ： 
  - Appium Desktop 提供了一个图形化用户界面，可以用于连接设备并手动录制操作，生成 Appium 脚本。你可以从 [Appium 官网]()  下载 Appium Desktop。
 
4. **Android 环境设置** ： 
  - **启用 USB 调试** ：在 Android 设备上启用开发者模式和 USB 调试。
 
  - **安装 ADB** ：确保 ADB 工具已安装，并能识别 Android 设备。

### 步骤 1：使用 Appium Desktop 录制动作 
 
1. **启动 Appium Server** ： 
  - 运行 Appium Desktop 或使用命令行启动 Appium：

```bash
appium
```
 
2. **启动 Appium Inspector** ：
  - 打开 Appium Desktop 并点击 "Start Server" 来启动 Appium Server。
 
  - 然后，点击 "Start Session"，并填写 Android 设备的 Desired Capabilities（这些是用于定义 Appium 与 Android 设备进行交互的配置）。一个示例的 `Desired Capabilities` 配置如下：

```json
{
  "platformName": "Android",
  "platformVersion": "11.0",
  "deviceName": "Android Emulator",
  "appPackage": "com.android.calculator2",
  "appActivity": ".Calculator"
}
```
 
3. **录制脚本** ：
  - 在 Appium Inspector 中点击 "Record" 按钮，开始在 Android 设备上执行操作。Appium 将自动生成相应的 Python 代码。
 
4. **导出脚本** ：
  - 录制完成后，可以将生成的脚本导出为 Python 格式，用于后续的自动化回放。

### 步骤 2：使用 Python 编写 Appium 脚本 

如果你不使用 Appium Desktop 的录制功能，可以直接编写 Appium 脚本来控制 Android 设备的操作。以下是一个简单的 Appium Python 脚本示例，用于自动化 Android 设备上的基本操作。

#### Python 脚本示例： 


```python
from appium import webdriver
from time import sleep

# 设置 Desired Capabilities
desired_caps = {
    "platformName": "Android",
    "platformVersion": "11.0",  # 根据你的设备版本
    "deviceName": "Android Emulator",  # 设备名称，可以是实际设备或模拟器
    "appPackage": "com.android.calculator2",  # 你想测试的应用程序包名
    "appActivity": ".Calculator",  # 应用的主 Activity
    "automationName": "UiAutomator2"  # 使用的自动化引擎
}

# 连接 Appium Server 并启动 Session
driver = webdriver.Remote("http://localhost:4723/wd/hub", desired_caps)

# 等待应用启动
sleep(5)

# 示例操作：点击数字 1
digit_one = driver.find_element_by_id("com.android.calculator2:id/digit_1")
digit_one.click()

# 示例操作：点击加号
plus_sign = driver.find_element_by_accessibility_id("plus")
plus_sign.click()

# 示例操作：点击数字 2
digit_two = driver.find_element_by_id("com.android.calculator2:id/digit_2")
digit_two.click()

# 示例操作：点击等号
equal_sign = driver.find_element_by_accessibility_id("equals")
equal_sign.click()

# 等待几秒以查看结果
sleep(2)

# 关闭会话
driver.quit()
```

#### 代码说明： 
 
1. **Desired Capabilities** ：这是用于告诉 Appium 如何连接 Android 设备以及要测试的应用程序。 
  - `platformName`: 平台类型（如 Android）。
 
  - `platformVersion`: Android 系统的版本号。
 
  - `deviceName`: 设备的名称或模拟器的名称。
 
  - `appPackage` 和 `appActivity`: 要启动的应用程序包名和主 Activity 名称。
 
  - `automationName`: Appium 使用的自动化引擎，通常是 `UiAutomator2`。
 
2. **操作元素** ：通过 `driver.find_element_by_id` 或 `driver.find_element_by_accessibility_id` 来找到元素并进行点击等操作。

### 步骤 3：重放录制的操作 

录制完成后，你可以通过运行生成的 Python 脚本来重放操作。确保 Appium Server 正在运行，然后运行脚本：


```bash
python recorded_script.py
```

这将会在连接的 Android 设备上自动重现录制的操作。

### 总结： 
 
- **录制** ：通过 Appium Desktop 的录制功能，生成 Python 脚本，帮助你快速录制 Android 应用中的操作。
 
- **重放** ：使用生成的 Python 脚本，通过 Appium 自动化执行操作，完成重放。
 
- **扩展** ：你可以在脚本中添加更多的自动化测试内容，如断言（Assertions）来验证应用的状态。

通过以上步骤，你可以利用 Appium 实现 Android 操作的自动录制和回放。如果有任何问题或进一步的需求，欢迎继续讨论！
