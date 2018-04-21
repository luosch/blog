title: mac 替换 rm 命令
date: 2015-08-15 00:13:07
tags: [技术, 实用技巧]
---
从 windows 迁移到 mac 之后, 命令行操作的时候 rm 永远是一个痛点

人总难免操作出一些失误，上次就听过一个信科的同学在命令行下用 rm -rf / 的悲剧，然后用移动硬盘恢复了半天233

### 所以我现在采取了下列做法
1. 用一个移动硬盘做 time machine 备份
2. 每天下班后，把代码 push 到 vps 搭建的一个私有 git repository 上
3. 将 rm 命令替换为移动到废纸篓

mac 的垃圾箱目录为 ~/.Trash 

（mac 被吐槽废纸篓不能恢复文件, 就是因为 mac 只是简单地将文件移动到这个命令, 而没有像别的操作系统那样追踪文件行为）

因为我是 zsh 用户，所以我通过编辑 ~/.zshrc 下做一个 alias 来替换 rm
（bash 用户编辑 ~/.bashrc）

在 # Example aliases 下面添加
``` bash
# Example aliases
alias rm=trash
trash() {
    mv $@ ~/.Trash/
}
```
最后在命令行敲下
``` bash
source ~/.zshrc
```

因为 rm 替换成了 mv，所以现在 rm 一个目录也是可以的 😊
