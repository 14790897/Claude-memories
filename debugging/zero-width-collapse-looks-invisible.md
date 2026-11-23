# 元素被压成 0 宽，会被判成「不可见」

**`toBeVisible()` 失败，但元素明明在 DOM 里——先量宽度。**

## 为什么

一个只在 CI 复现的失败：日志写着 `locator resolved to <span …>`，紧接着 `unexpected value "hidden"`。本地怎么跑都是绿的。

Playwright 把 **0 尺寸**算作 hidden。而 flex 行里如果「主文本可截断 + 次要列不可收缩」，整行内容超出容器时，主文本会被一路压到 0 宽。

本地 Windows 字体窄、CI 的 Linux 字体宽，于是同一段内容只在 CI 撑爆。一个弹层里「日期 + 金额」吃满整行，旁边的作业 ID 整条消失。

## 怎么用

**排查**：CI 挂掉时先下载 Playwright 产物，看 `test-failed-*.png`——肉眼就能看出塌陷。再看 `trace.zip` 里的 DOM 快照确认宽度。

**修复**：把争宽的元素分行。别让 `shrink-0` 的次要列和可截断的主文本待在同一行。

**守卫**：只断言 `toBeVisible()` 不够——被截到 20px 也算 visible。把容器压窄后断言 `boundingBox().width > 40`，或者直接断言具体尺寸。

## 一条推论

**「可见」在浏览器里是个几何判断，不是存在判断。** 只要涉及布局，断言就要落到数值上。
