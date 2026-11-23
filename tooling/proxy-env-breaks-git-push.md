# 环境里配了代理，git push 会静默失败

**代理变量存在 ≠ 代理在跑。**

## 为什么

本机 `HTTP_PROXY` / `HTTPS_PROXY` 指向一个本地端口，而这个代理平时不开。

`git push` 不报「没网」，也不报「连不上」，就是失败——因为 git 老老实实按环境变量走代理，代理没在监听。

## 怎么用

推之前先清掉：

```bash
unset HTTP_PROXY HTTPS_PROXY http_proxy https_proxy && git push <remote> <branch>
```

或者对这个仓库永久禁用：

```bash
git config --local http.proxy ""
```

## 一条推论

**环境变量是隐式配置。** 你在命令行里看不到它，但它会改变每一个下游工具的行为。遇到「明明网络是通的，某个工具就是不工作」，先 `env | grep -i proxy`。
