---
layout: post
title: ollama pull 慢
date: 2026-03-12
tags: ollama
---

ollama pull 常常越來越慢，網路上找的，有人寫的可以中斷，繼續下載
```
import subprocess
import time
import os
import sys
import signal

def run_command(model):
    while True:
        # 要执行的命令和参数
        command = ['ollama', 'pull', model]

        # 启动命令进程
        process = subprocess.Popen(command)

        try:
            # 等待 60 秒
            time.sleep(60)

            # 尝试在 60 秒后终止进程
            if process.poll() is None:  # 如果进程仍在运行
                os.kill(process.pid, signal.SIGTERM)
                print("Process terminated")
        except Exception as e:
            print(f"An error occurred: {e}")
        finally:
            process.wait()  # 确保进程关闭
        print("Looping again...")
if __name__ == '__main__':
    run_command(sys.argv[1])
```

```
