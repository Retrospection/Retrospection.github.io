---
title: "解决 Windows 11 PowerShell 禁止运行脚本问题"
summary: 通过配置注册表，一劳永逸地解决 PowerShell
date: 2022-01-12T23:01:07+08:00
draft: false
---

最近入手了一台新的 windows 电脑作为工作外学习研究的主力机，在配置开发环境时，遇到了 PowerShell 禁止运行脚本问题。
在网上搜索一番，普遍给出的方案是通过**管理员模式**下执行 `Set Execution-Policy` 命令的方法解决问题，如下：

```powershell
PS C:\> Set Execution-Policy RemoteSigned
```

但是，由于 Execution Policy 的本意是为了保护用户安全，防止恶意脚本随意执行。因此，在 PowerShell 里用执行上述命令进行的修改只能针对本次会话生效，而这就给需要频繁使用脚本的开发者带来了很多不便。

于是尝试寻找能够一劳永逸解决问题的方法，最后，到微软官方文档里找到了一个修改注册表的解决方案。


![ExecutionPolicy](/images/windows11-execution-policy/1.jpg)

如图所示，在注册表中在路径 `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\PowerShell\1\ShellIds\Microsoft.PowerShell`
下找到 `ExecutionPolicy`，然后将值改成 `RemoteSigned` 即可解决问题。

