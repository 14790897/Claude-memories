# worktree 里别重装依赖，用 junction 借

**在 git worktree 里跑测试，缺依赖时优先链到主检出，而不是重新安装。**

## 为什么

Windows 上 `npm ci` 重装要几分钟还占空间。`node_modules` 是 gitignored 的，用目录 junction 指过去零拷贝、不影响 git 状态：

```bash
cmd //c "mklink /J <worktree>\node_modules <main>\node_modules"
```

Python 的 `.venv` 同理。

## 但有一条硬边界

**不要 junction 构建产物目录（`out/`、`dist/`、`build/`）。**

有个项目的主进程用 `join(__dirname, '../../..')` 推仓库根。`__dirname` 经过 junction 会解析回**主检出**的路径——于是 worktree 里跑起来的程序，加载的是主检出的代码和主检出的 Python 环境。worktree 里的改动对测试完全不可见，查了两个小时才发现。

**规则**：依赖可以共享，产物必须各自生成。worktree 里跑集成测试前，老老实实本地构建一次。

## 几个附带的坑

- **共享目录里可能缺包**：主检出的 `package.json` 声明了某依赖但实际没装，在 worktree 里就报 `Cannot find package`。补装时用 `--no-save`，别动 lock 文件
- **浏览器二进制不共享**：测试框架要找的无头浏览器可能没装全，整套用例会挂在 `Executable doesn't exist` 上——那不是代码回归，别误判
- **管道会吃掉退出码**：`npm test | tail` 的退出码是 tail 的。要判成败就读测试报告 JSON，或者用 `${PIPESTATUS[0]}`

## 一条推论

**「省时间」和「环境隔离」是矛盾的。** 共享依赖省时间，共享产物会骗你。分界线画在「可重新生成的」和「不可重新生成的」之间。
