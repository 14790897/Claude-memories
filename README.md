# Claude-memories

长期积累的 AI 编程助手使用经验。每条都来自一次真实的踩坑，保留「为什么」和「什么时候适用」，去掉项目相关的具体信息。

不是提示词合集，是**协作纪律**——哪些事该做、哪些不该做、做到什么程度停手。

---

## 排查与推理

- **[先搜 issue，别自己推演](debugging/search-the-issue-tracker-first.md)**
  加了「本该起作用的标准机制」却静默无效时，通常不是配置问题，是软件 bug。四个方向都改不通的那次，搜 issue 两分钟就命中了。

- **[小问题别起编排](debugging/dont-orchestrate-small-lookups.md)**
  查一句话出动用一次搜索，别开多智能体。编排的等待和噪音远大于收益。

## 工具与自动化

- **[抓内容先用 API](tooling/prefer-api-over-html-scraping.md)**
  HTML 抓取的成本大头不是抓，是清理——分页、导航、编码、PUA 乱码、误删风险。

- **[别把抓来的标题读进对话](tooling/keep-scraped-titles-out-of-context.md)**
  搜索结果的自由文本进入上下文，可能触发下游内容过滤，整个会话都发不出去。

- **[读图失败时别猜](tooling/when-image-reading-fails.md)**
  换一条视觉通道，或者明说读不了。基于文件名编一个描述是最坏的选择。

- **[发快捷键之前先确保焦点](tooling/focus-before-global-shortcuts.md)**
  `SendInput` 级按键只认前台窗口。一次 `Ctrl+W` 关掉的是 AI 自己的对话。

- **[只碰自己开的标签页](tooling/never-touch-user-browser-tabs.md)**
  用户正在用的界面和自己的临时工作环境是两个域。越界一次，信任就没了。

- **[带空格的 profile 名要内层加引号](tooling/quoting-chrome-profile-names.md)**
  参数被按空格拆开，Chrome 会把孤立的数字当网址，在已运行的窗口里开出错页。

- **[按笔记删视频要读正文](tooling/read-content-not-filenames.md)**
  文件名是人随手起的，不是主键。129 个笔记只匹配上 8 个，因为 23 篇根本没有编号。

- **[定时任务用现有调度器](tooling/use-existing-routines.md)**
  多一套调度器 = 行为来源不可见 + 日志分散。能改就不新建。

## 协作约定

- **[提交之后立刻推送](collaboration/push-after-commit.md)**
  不要分轮次等确认。同时记住 `.git/hooks/` 不随仓库分发。

- **[用完就清](collaboration/clean-up-temp-files.md)**
  临时目录累积到 26 GB 是真实发生过的。白名单 + 年龄，不要一刀切。

- **[私人消息别过度审校](collaboration/dont-over-hedge-personal-messages.md)**
  人际温度本身就包含一点小越界。把风险全磨掉，消息也就死了。

## 信息核实

- **[查费率先锁定产品线](research/check-the-right-product-line.md)**
  同一家公司不同产品线的费率天差地别。开发者文档答不了 App 内产品的问题。

- **[找 skill 必须联网搜](research/search-the-web-for-skills.md)**
  翻了一遍本地目录就答「没有」，是不成立的。本地没装 ≠ 不存在。

---

## 关于这份记录

写作原则：

1. **每条都对应一次具体的事故或打断**，不是抽象建议
2. **说明适用边界**——什么时候这条不适用，和什么时候适用一样重要
3. **给出可执行的判据**，而不是「要注意」「要小心」这类无法验证的提醒

内容源自个人使用 AI 编程助手的长期实践，已移除项目、账号、基础设施相关的具体信息。
