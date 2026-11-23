# Claude-memories

长期使用 AI 编程助手积累的经验。每条都来自一次真实的踩坑，保留「为什么」和「什么时候适用」，去掉项目相关的具体信息。

不是提示词合集，是**协作纪律**——哪些事该做、哪些不该做、做到什么程度停手。

| 目录 | 主题 | 条目 |
|---|---|---|
| [debugging/](debugging/) | 排查与推理的纪律 | 15 |
| [verification/](verification/) | 交付前的验证 | 9 |
| [tooling/](tooling/) | 工具与自动化的边界 | 23 |
| [collaboration/](collaboration/) | 与人的协作约定 | 17 |
| [research/](research/) | 信息核实的方法 | 2 |

---

## 排查与推理

- **[先搜 issue，别自己推演](debugging/search-the-issue-tracker-first.md)** — 加了「本该起作用的标准机制」却静默无效，通常不是配置问题，是软件 bug
- **[小问题别起编排](debugging/dont-orchestrate-small-lookups.md)** — 查一句话出处出动一次搜索，别开多智能体
- **[heredoc 打补丁会吃掉反斜杠](debugging/heredoc-corrupts-escapes.md)** — `\b` 变成了退格字符 0x08，正则静默失效，而 `cat` 看起来完全正常
- **[别覆盖正在运行的 shell 脚本](debugging/dont-overwrite-running-scripts.md)** — bash 按字节偏移增量读脚本，覆盖会导致执行串位
- **[CRLF 会让远程脚本直接语法错误](debugging/crlf-breaks-remote-scripts.md)** — 本地 `bash -n` 会骗你，要在目标端再验一次
- **[epoch 不一定是 UTC](debugging/epoch-is-not-utc.md)** — 按 UTC 分组会整体偏几小时，半天比整天看着像数据源掉线
- **[别把全量数据读进内存](debugging/dont-load-everything-into-memory.md)** — 一个分析脚本可以把整台生产机拖到 SSH 都连不上
- **[OOM 看起来像「什么都没发生」](debugging/oom-looks-like-nothing.md)** — `rc=137` 只留一行日志，「单独跑能过」不等于「在调度器里能过」
- **[分清「水平」和「变化」](debugging/change-vs-level.md)** — 「从 A 到 B」写成「低于 B」，结果会完全反过来
- **[分不清 TCP 和 UDP，就别急着怪带宽](debugging/compare-transports-before-blaming-bandwidth.md)** — 同一线路 TCP 流畅、UDP 丢 30%，带宽测出来还富余。吞吐和丢包是两个指标
- **[读 PR 别只看正文](debugging/read-pr-comments-not-just-body.md)** — 正文写着「未联调」，评论里早就贴了联调通过的证据。开口前先读完记录
- **[CI 红了先看分支落后多少](debugging/check-branch-freshness-before-blaming-flake.md)** — workflow 来自基分支、代码来自 PR head。主干上早修好的测试，在落后的分支上必然失败
- **[元素被压成 0 宽，会被判成「不可见」](debugging/zero-width-collapse-looks-invisible.md)** — 只在 CI 塌陷：本地字体窄、CI 字体宽。可见在浏览器里是个几何判断，不是存在判断
- **[断言别命中你自己发的文本](debugging/assertions-that-match-your-own-input.md)** — 额度早就用完了，探针还是绿的——标记字符串同时存在于用户气泡和会话标题里
- **[提交信息里的 skip-ci 标记会吞掉整条 workflow](debugging/skip-ci-suppresses-workflow-runs.md)** — 不是跳过几步，是这个提交根本不创建 run。发版工具默认加的那个后缀，能让流水线静默停摆

## 交付前验证

- **[交付前先看数据的「形」](verification/check-the-shape-of-the-data.md)** — 「取最新一条」最易串维度；单位可能已被换算过；样本里可能混进了不该有的东西
- **[负结论先跑反向控制组](verification/negative-results-need-a-control-group.md)** — 反向也亏，说明问题在框架不在信号
- **[总量比不能当数据质量信号](verification/aggregate-ratios-prove-nothing.md)** — 销毁地址、锁仓、协议哨兵都会让它误报
- **[引用要核到底](verification/verify-citations.md)** — DOI 存在但指向另一篇论文，肉眼看不出来
- **[一致性不能证明正确性](verification/consistency-is-not-correctness.md)** — 两个实现结果一致，可能只是同一个坏模型
- **[没见它红过的测试等于没测](verification/make-the-test-fail-first.md)** — 写完测试，把修复改回去看它失败一次
- **[碰撞检测测不出「该挖的洞没挖」](verification/assert-the-feature-not-just-collisions.md)** — 结构件要逐特征断言，不是跑一遍干涉检查就算验过
- **[mock 全绿，不等于修好了](verification/mocks-passing-is-not-a-fix.md)** — 用例里 mock 立即返回，竞态被完全掩盖。用户否决你的判断时，先假设用户是对的
- **[碰真实账号的功能，必须真跑一次](verification/live-test-real-accounts.md)** — mock 只证明契约对，不证明真能拿到凭据。真实凭据只经环境变量注入

## 工具与自动化

- **[抓内容先用 API](tooling/prefer-api-over-html-scraping.md)** — HTML 抓取的成本大头不是抓，是清理
- **[别把抓来的标题读进对话](tooling/keep-scraped-titles-out-of-context.md)** — 自由文本进入上下文可能触发下游内容过滤，整个会话都发不出去
- **[读图失败时别猜](tooling/when-image-reading-fails.md)** — 换通道，或者明说读不了
- **[发快捷键之前先确保焦点](tooling/focus-before-global-shortcuts.md)** — `SendInput` 级按键只认前台窗口
- **[只碰自己开的标签页](tooling/never-touch-user-browser-tabs.md)** — 用户正在用的界面和自己的临时环境是两个域
- **[带空格的 profile 名要内层加引号](tooling/quoting-chrome-profile-names.md)** — 参数被拆开，孤立的数字会被当成网址
- **[按笔记删视频要读正文](tooling/read-content-not-filenames.md)** — 文件名是人随手起的，不是主键
- **[定时任务用现有调度器](tooling/use-existing-routines.md)** — 多一套调度器 = 行为来源不可见 + 日志分散
- **[别用 adb 往设备输入框里打字](tooling/dont-type-into-device-inputs.md)** — 中文输入法会吃掉标点、打乱顺序；关掉 IME 会让键盘彻底坏掉
- **[给推理模型设 max_tokens 会让它「拒绝回答」](tooling/cap-tokens-starves-reasoning.md)** — 思考和正文共享预算，正文会被挤成空的
- **[R8 混淆会让 JNI 库在 release 版崩](tooling/r8-strips-jni-names.md)** — debug 一切正常，release 第一次调用就 SIGABRT
- **[文档的内容可能在图片里](tooling/documents-hide-content-in-images.md)** — 只读到文字不代表读全了
- **[能写成 if 就别写提示词](tooling/prefer-rules-over-model-calls.md)** — 在自己算出来的特征上再调一次模型，是花钱买黑箱
- **[验的是「用户看到的」](tooling/look-at-what-the-user-sees.md)** — API 返回 200、字段齐全，和界面显示了，是完全不同的断言
- **[浏览器改动自己跑一遍再交付](tooling/verify-in-browser-before-delivery.md)** — 交代码让用户测，被回「你自己打开浏览器测」。实测当场抓到两个静态审阅发现不了的问题
- **[自动化打开的窗口移出屏幕](tooling/move-automation-windows-offscreen.md)** — 用户不需要提防一个看不见的窗口。只动自己开的，移不动就说明
- **[不要弹出任何窗口](tooling/dont-open-windows-at-all.md)** — 终端、浏览器、后台标签页都算。真正的后台是不创建窗口
- **[CI 的 shell 逻辑，本地就能跑](tooling/run-ci-shell-locally.md)** — 把 run 块从 yml 里抽出来跑。抄脚本会失真，Windows 版 jq 还会给 stdout 加 CRLF
- **[worktree 里别重装依赖，用 junction 借](tooling/share-node-modules-across-worktrees.md)** — 依赖可以共享，产物必须各自生成，否则程序会加载主检出的代码
- **[上传到 Release 的文件名会被改写](tooling/release-asset-names-get-sanitized.md)** — 空格变成点号，自动更新按原名找就是 404。文件名一旦有别的系统依赖，它就是接口
- **[环境里配了代理，git push 会静默失败](tooling/proxy-env-breaks-git-push.md)** — 代理变量指向的端口根本没在监听。网络是通的、工具却不工作时，先 `env | grep -i proxy`
- **[describe 层的 skip 会让单个用例的 timeout 失效](tooling/describe-skip-disables-per-test-timeout.md)** — 长用例在 10 分钟整被杀，看起来像应用崩溃。用 `test.setTimeout()` 兜底
- **[beforeAll 是每个 worker 一次，不是每个文件一次](tooling/beforeall-runs-per-worker.md)** — 失败重启后，剩余用例拿到全新环境。需要数据的用例自己幂等补齐

## 协作约定

- **[提交之后立刻推送](collaboration/push-after-commit.md)** — 不要分轮次等确认；`.git/hooks/` 不随仓库分发
- **[长任务跑到完，别中途问](collaboration/work-to-completion.md)** — 只有真正的阻塞和不可逆操作才停下来
- **[用完就清](collaboration/clean-up-temp-files.md)** — 临时目录累积到几十 GB 是真实发生过的
- **[私人消息别过度审校](collaboration/dont-over-hedge-personal-messages.md)** — 人际温度本身就包含一点小越界
- **[评价作品要直说，不要恭维](collaboration/critique-with-evidence.md)** — 恭维的成本是转移的
- **[仓库是成品，不是工作目录快照](collaboration/repos-are-not-snapshots.md)** — 历史备份、中间文件、日志都不该进去
- **[推荐要带证据](collaboration/recommend-with-evidence.md)** — star、fork、最近提交、license，缺一个就没法判断
- **[说错了改原文，不要发更正](collaboration/edit-in-place-dont-correct.md)** — 更正跟帖会让人读到过时信息
- **[别补回「丢失」的内容](collaboration/dont-restore-what-may-be-deleted.md)** — 文件变小可能是对方有意删的
- **[别往同步目录里擅自写入](collaboration/dont-write-into-synced-folders.md)** — 有权限不等于有授权
- **[发版跑着的时候不要推 main](collaboration/dont-push-while-release-runs.md)** — 会留下只有 tag 没有产物的半成品；三个检查都会骗你
- **[先给能用的答案，再决定要不要深挖](collaboration/answer-first-then-investigate.md)** — 多轮取证容易被「算了」打断。第一轮就给结论，该实测的仍要实测
- **[报告控制在两页](collaboration/keep-reports-to-two-pages.md)** — 四页被要求砍到两页。篇幅会被当成质量标准来读
- **[本地绿了再推](collaboration/local-green-before-push.md)** — CI 是最终确认，不是调试器。反馈回路越短，改得越准
- **[开工前先搜有没有人在做同一件事](collaboration/search-for-competing-pr-first.md)** — 别人 4 分钟前提的 PR 比你更全，issue 却还挂在 OPEN
- **[说人话](collaboration/plain-language-communication.md)** — 先一句话给结论。术语是给自己省力的，说人话是给对方省力的
- **[GitHub 上非必要不加评论](collaboration/fewer-github-comments.md)** — 每条评论都是一次打断。证据进 PR 描述或 commit message

## 信息核实

- **[查费率先锁定产品线](research/check-the-right-product-line.md)** — 同一家公司不同产品线的费率天差地别
- **[找 skill 必须联网搜](research/search-the-web-for-skills.md)** — 本地没装 ≠ 不存在

---

## 关于这份记录

写作原则：

1. **每条都对应一次具体的事故或打断**，不是抽象建议
2. **说明适用边界**——什么时候这条不适用，和什么时候适用一样重要
3. **给出可执行的判据**，而不是「要注意」「要小心」这类无法验证的提醒

内容源自个人使用 AI 编程助手的长期实践，已移除项目、账号、基础设施相关的具体信息。

---

## 许可

[MIT](LICENSE) © 2026 14790897
