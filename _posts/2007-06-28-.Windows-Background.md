---
layout: post
title: WINDOWS 背景執行方式
date: 2025-11-17
tags: WIN windows win
---
windows 原來是直接用 powershell 一行執行就可以背景執行 以前都傻傻的寫 vbs
'''
(New-Object -ComObject WScript.Shell).Run("cmd /c autogenstudio ui --host 0.0.0.0 --port 8081", 0, $False)
'''
