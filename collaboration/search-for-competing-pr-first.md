# 开工前先搜有没有人在做同一件事

**issue 是 OPEN，不代表没人动工。**

## 为什么

一次做完一个 issue，提交 PR 后才发现别人 4 分钟前已经提了功能更全的版本——而且对方修的是根因，我只是绕过了症状。我的白做，关了让位。

issue 列表上那个 issue 一直是 OPEN 的。

## 怎么用

**接「完成某个 issue」的任务时，第一条工具调用之前先搜**：

```bash
gh pr list --repo <owner/repo> --state all --search "<issue号> in:title"
gh search prs --repo <owner/repo> "<issue号>"
```

有 open PR 覆盖它，先向用户报告冲突，别继续实现。

## 一条推论

**在多人协作的仓库里，「没人做」和「没人认领」是两件事。** issue 的分配状态不是事实来源，PR 列表才是。
