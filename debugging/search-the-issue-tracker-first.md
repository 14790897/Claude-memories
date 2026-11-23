# 先搜 issue，别自己推演

**遇到「某个软件的行为不符合预期」的故障时，先去搜它的 issue tracker。**

## 为什么

一次真实的排查：Obsidian LiveSync 报了一个假 CORS 错误——

```
The request was successful by API. But the native fetch API failed!
Please check CORS settings...
```

我读了插件源码，造了一整套 curl 预检矩阵，改了两处生产服务器配置（CouchDB 的 `[cors]`、Caddy 的 `keepalive`/`lb_retries`）。**四个方向全部落空**，而且那个 curl 矩阵虽然做得完整，却答错了题——真实客户端的请求头本来就在白名单里。

换成搜 issue 之后，两分钟命中 [caddy#6678](https://github.com/caddyserver/caddy/issues/6678)：客户端走 HTTP/3 时请求偶发中断，后端根本收不到。症状逐字吻合，真实根因与 CORS 毫无关系。

## 怎么用

**触发条件**——满足任意一条就停止推演，去搜：

- 为同一个症状已经改了 2 个以上配置项，而且每个都「合理但无效」
- 加了一个**本该起作用的标准机制**（重试、超时、白名单、缓存），却**静默无效**
- 症状只出现在某一种协议、或某个特定客户端上

**搜法**：

- 拿**报错原文的关键片段**去搜，不要转述成自己的话
- 重点搜**症状描述**，而不是产品名
- 换几轮关键词

用 GitHub 搜索可以走 API：

```bash
gh api "search/issues?q=repo:<org>/<repo>+%22<报错片段>%22&per_page=10" \
  --jq '.items[]|"#\(.number) \(.title)"'
```

## 两条推论

**「标准机制静默无效」本身就是最强的线索。** 它说明你面对的不是配置问题，是软件 bug。

**报错文案是写它的人当时的假设，不是诊断结论。** 上面那个 "CORS" 文案，实际只代表一个 `TypeError`。
