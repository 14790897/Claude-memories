# 别用 adb 往设备输入框里打字

**中文输入法会把 `input text` 搅乱，而关掉输入法会让键盘彻底坏掉。**

## 症状

- `:` 和 `/` 被**直接丢掉**
- `.` 变成全角 `。`
- 字符会**乱序**：`https://dav.example.com/dav/` 落成 `: 、 h:/t/t.psda./v/examplecomdav`
- **切到英文模式也没用**——标点变 ASCII 了，乱序照旧
- 纯字母数字没问题，所以**看起来像随机故障**

更要命的是：**不要在两个输入框之间移动焦点**。输入法把没提交的内容当组词，焦点一换就整段倒进**新**的框、原来的框清空。本来想填地址，结果把地址「搬」进了账号框。

## 更严重的一个坑

为了绕开上面的问题，把设备上的输入法全 `disable` 掉——**没有 IME 时，很多框架（Flutter 等）一个字都收不到**，因为文本输入全程走 IME。

再 `enable` 回来之后，键盘**再也不弹了**，而且状态全是「正确」的：

```
mInputShown=true                        ← 系统以为键盘是显示的
mInputWindowVisibleHeight=-44           ← 但窗口可见高度是负的
mServedInputConnection=RemoteInputConnectionImpl{...}   ← 连接也在
```

**state 全对，屏幕上一个键都没有。** 退出应用、重启应用都没用。

修法是把输入法切走再切回，强制系统重建窗口：

```bash
adb shell am force-stop <ime 包名>
adb shell ime set <另一个 IME>     # 这里窗口被拆掉
adb shell ime set <原来的 IME>     # 这里建一个新的
adb shell dumpsys window | grep mInputMethodWindowVisibleHeight   # 要 > 0 才算好
```

## 结论

- **别用 adb 往设备输入框里打字。** 要凭据就让用户自己填
- 真要自动化，装一个专用输入法（走广播的那种），不要跟系统输入法搏斗
- **`ime disable` 不要用**——它造成的故障比它绕开的问题更难收场
- 判断键盘出没出，**不能看 `dumpsys input_method`**（会说 shown=true），要看 `dumpsys window` 里的 `mInputMethodWindowVisibleHeight`，或者直接截图
