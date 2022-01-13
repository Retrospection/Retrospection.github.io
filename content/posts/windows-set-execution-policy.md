---
title: "解决 Windows 11 PowerShell 禁止运行脚本问题"
summary: 通过配置注册表，一劳永逸地解决 PowerShell
date: 2022-01-12T23:01:07+08:00
draft: false
---



最近入手了一台新的 windows 电脑作为工作外学习研究的主力机，在配置开发环境时，遇到了 PowerShell 禁止运行脚本问题。
在网上搜索一番，普遍给出的方案是通过管理员模式下执行 `Set Execution-Policy` 命令的方法解决问题，如下：

```powershell
PS C:\> Set Execution-Policy RemoteSigned
```

但是，这个方案在机器重启之后就失效了，因此想找到一个能够永久变更配置的方案。

最后，到微软官方文档里找到了一个修改注册表的解决方案。

可以到 `HKEY_CURRENT_USER\SOFTWARE\Microsoft\PowerShell\1\ShellIds\Microsoft.PowerShell`
下找到 `ExecutionPolicy`，然后将值改成 `RemoteSigned` 即可解决问题。
