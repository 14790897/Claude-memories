# 发版跑着的时候不要推 main

**自动发版的工作流最后要把版本提交推回 main。这中间 main 一前进，推送就被拒。**

## 为什么

语义化发版（semantic-release 之类）的收尾动作是 `git push HEAD:main`，把改过版本号的提交推回去。

如果这中间有人往 main 推了东西：

```
! [rejected] HEAD -> main (fetch first)
```

而那时 **tag 可能已经建了**——留下一个「只有 tag、没有产物」的半成品。

发版全程十几分钟，很容易在推完之后去做别的事，然后忘了又推一次。

## 三个会骗人的检查

**① `gh run list --limit 1`。** 它按时间排序，你看到的很可能是同一批 push 触发的**另一个工作流**。必须按 workflow 过滤。

**② `gh run watch`。** 推完立刻取「最新那个 run」，可能取到的是**上一次已经跑完的**——新 run 还没注册进列表。watch 一个已完成的 run 会非零退出，于是「等它跑完」的脚本报告「这次没成功」，而实际发布是成功的。

**③ 本地 tag。** 没有 `git fetch --tags` 的话，`git describe` 拿到的是过期版本，于是下载并安装了**上一个版本**——整个过程看起来正常，只是版本号没变。

## 怎么用

**判据要用远端的：**

```bash
git fetch --tags origin
gh release list --limit 3
```

**闸门要写成 `exit 1`，不能只打印一行。** 「输出里明明写着 in_progress，然后照样推」这种事真的会发生在自己身上。

## 收尾

每次失败后核一遍有没有孤立的 tag：

```bash
git ls-remote --tags origin   # 对比 gh release list
```
