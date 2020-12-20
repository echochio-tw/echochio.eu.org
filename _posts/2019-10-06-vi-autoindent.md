---
layout: post
title: vi 縮排關閉
date: 2019-10-06
tags: vi
---


設定自動縮排：即每行的縮排值與上一行相等:

```
:set autoindent
:set shiftwidth=4
```

取消設定
```
:set noautoindent
```

進入paste模式以後，可以在插入模式下粘貼內容，不會有任何變形。
```
:set paste
```

Vim中滑鼠右鍵不能貼上

Copy lines in visual mode in vim
```
echo 'set mouse-=a' > ~/.vimrc
```

or
```
:set mouse=
```


