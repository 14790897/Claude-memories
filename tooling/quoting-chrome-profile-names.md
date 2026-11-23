# 带空格的 Chrome profile 名要内层加引号

**用 PowerShell 启动指定 profile 的 Chrome 时，必须把 `Profile N` 本身用双引号包住。**

## 现象

```
--profile-directory=Profile 6
```

会被按空格拆成 `--profile-directory=Profile` 和 `6` 两个参数。Chrome 把孤立的 `6` 当成网址，在**已经运行的窗口里**开出一个 `0.0.0.6` 的错误页，并把当前标签跳走。

看起来像「启动失败」，实际是污染了用户正在用的窗口。

## 正确写法

```powershell
Start-Process "C:\Program Files\Google\Chrome\Application\chrome.exe" `
  -ArgumentList '--profile-directory="Profile 20"'
```

外层单引号保住整串，内层双引号保住 profile 名里的空格。

`Default` 没有空格，不受影响。

## 怎么确认真的起来了

不要只看 `Get-Process chrome | Select MainWindowTitle`——它每个进程只报一个标题，而且报的是主窗口，不是你新起的那个 profile。

要看：

- 自动化通道的连接列表里是不是多了一条
- `Get-CimInstance Win32_Process` 里是不是出现了带 `--profile-directory` 的主进程

## 推广

这是 Windows 命令行通用的坑：**参数里带空格，就要有引号，而且要在正确的层加**。PowerShell 的 `-ArgumentList` 再多一层解析，所以内层引号是必需的，不是可选的。
