# CRLF 会让远程脚本直接语法错误

**往 Linux 上传 `.sh` 之前，先把本地文件转成 LF。**

## 为什么

仓库里存的是 LF，但 git 在 Windows 工作区会检出成 CRLF。`scp` / `pscp` 原样传输，远程 bash 不剥 `\r`——于是反斜杠续行后面跟着一个 `\r`，就不再是续行：

```
unexpected end of file
```

一个每 30 分钟跑一次的生产脚本，传上去就坏了，下一个 tick 整轮失败。

## 最坑的地方

**本地 `bash -n` 是通过的。** Git Bash 容忍 CRLF，所以本机自检给的是假的安全感。

## 怎么用

```bash
# 1. 转 LF
python -c "d=open(p,'rb').read().replace(b'\r\n',b'\n'); open(p,'wb').write(d)"

# 2. 本地语法检查
bash -n <file>

# 3. 上传后，在远程再跑一次 bash -n —— 这一步才是权威
```

`.py` 不受影响，Python 接受 CRLF。

## 判据

**「本地通过」和「目标环境通过」是两个断言。** 只要传输会改变字节（换行符、编码、压缩），就必须在目标端再验一次。
