# 代理指向死端口时 git 会报错——别让管道把这一步藏起来

**代理变量存在 ≠ 代理在跑。但「失败没看见」通常不是 git 没报，是调用方式把错误吃掉了。**

## 实测：git 报得很清楚

`HTTPS_PROXY` 指向一个没人监听的本地端口：

```
$ HTTPS_PROXY=http://127.0.0.1:9 git ls-remote origin
fatal: unable to access 'https://github.com/...':
Failed to connect to github.com port 443 via 127.0.0.1 after 2101 ms:
Could not connect to server
```

代理地址、目标主机、耗时都写出来了。**不是静默失败。**

## 那「静默」是怎么来的

三个原因都能让一次失败的推送看起来像没发生：

**① 管道吞掉退出码。**

```bash
git push 2>&1 | tail -3
echo $?        # 0 —— 这是 tail 的退出码
```

不接管道时是 `128`。加 `set -o pipefail` 才能拿回真实状态。

**② `tail -n` 截掉了错误行。** 只留最后几行，而 `fatal` 那行不一定在末尾。

**③ 只看「仓库有没有更新」，没读输出。** 没更新就当成「本来就没有要推的」。

## 怎么用

排查「明明网络是通的，某个工具就是不工作」，先看环境：

```bash
env | grep -i proxy
```

**要清的不止 `HTTP(S)_PROXY`**——`ALL_PROXY` 同样会被 git 认（实测报一模一样的错）。

单次绕过比 `unset` 省事：

```bash
git -c http.proxy= push <remote> <branch>
```

`http.proxy` 设成空串确实生效——实测 `git -c http.proxy= ls-remote` 能正常连通。

## 一条推论

**环境变量是隐式配置。** 命令行里看不到它，但它改变每一个下游工具的行为。

配套纪律：**跑命令不要用 `| tail` 收尾之后再读 `$?`**——那个状态已经属于 `tail` 了，不再是你关心的那条命令的。
