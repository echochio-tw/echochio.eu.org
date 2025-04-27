---
layout: post
title: pyenv 多版本 python
date: 2025-04-26
tags: python pyenv pyenv-virtualenv
---

Linux 安裝
安裝依賴套件

```
sudo apt-get update
sudo apt-get install -y build-essential libssl-dev zlib1g-dev libbz2-dev \
libreadline-dev libsqlite3-dev wget curl llvm gettext libncurses5-dev \
tk-dev tcl-dev blt-dev libgdbm-dev git python-dev python3-dev aria2
```
下載 pyenv
```
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
```
設定 shell 環境變數（以 bash 為例）
```
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n eval "$(pyenv init -)"\nfi' >> ~/.bashrc
```
pyenv-virtualenv 安裝
```
git clone https://github.com/pyenv/pyenv-virtualenv.git $(pyenv root)/plugins/pyenv-virtualenv
```
設定 shell 環境變數（以 bash 為例）
```
eval "$(pyenv init --path)"  >> ~/.bashrc
eval "$(pyenv init -)"  >> ~/.bashrc
eval "$(pyenv virtualenv-init -)"  >> ~/.bashrc
```
列出可安裝的 Python 版本：
```
pyenv install --list
```
安裝指定版本：
```
pyenv install 3.11.12
```
建立名為 env-3.11.12 的虛擬環境
```
pyenv virtualenv 3.11.12 env-3.11
這會以 Python 3.10.12 為基礎建立一個名為 env-3.10 的虛擬環境。
```
啟用虛擬環境
```
cd ~/FramePack
pyenv activate env-3.11
pyenv local env-3.11
```
如 cd ~/FramePack 就會
```
(env-3.11) root@user:~/FramePack#
python -V
Python 3.11.12
```
如 cd ~/ 會
```
root@user:~#
python -V
Python 3.12.3
```
如果想看「所有可以安裝的版本」（超多超多）：
```
pyenv install --list
```




