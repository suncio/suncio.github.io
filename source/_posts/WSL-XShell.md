---
title: WSL + XShell
date: 2017-11-29 10:15:19
tags: 
- WSL
- OS
---
因为虚拟机炸了<i>（血的教训，告诉我们不要相信50G空间够你日摆一个操作系统）</i>，加上CentOS的yum源的更新程度感天动地，就把工作环境迁移到WSL了。WSL使用体验整体良好，尽管有一些尚可接受的延迟。但是！配色瞎眼睛！这个配色开发者你们眼睛不会瞎吗？！
遂决定更改配色方案，由于之前连远程机器时习惯XShell的配色。就干脆开一个sshd，用本地回路在XShell上工作了，主要参考“[Windows Subsystem for Linux(WSL)的配置](https://memeda.github.io/%E6%8A%80%E6%9C%AF/2016/08/03/WslBashOnWindowsConfig.html)”这篇文章。
最后得到的结果似乎比直接开bash.exe还要快，而且配色也改善了很多。