# describe 层的 skip 会让单个用例的 timeout 失效

**跳过条件写在 describe 里，组内每个 test 的 `{ timeout }` 就不生效了。**

## 为什么

一个跑 8–10 分钟的长用例，写了 `{ timeout: 900_000 }`，仍然在 10 分钟整超时。查 `test.info().timeout`，报的还是项目默认值。

原因是在 `describe` 回调里调用了 `test.skip(条件, 原因)`（用于在 CI 或缺依赖的环境整组跳过）。这之后，组内 test 的第二个参数里的 timeout 被忽略。

更坑的是症状：超时把 worker 连同被测应用一起杀掉，日志里是「进程退出」「target closed」。看起来像应用崩了，其实是测试框架到点收工。

## 怎么用

需要超长的用例：

```js
test.describe('…', () => {
  test.skip(!process.env.LONG_RUN, '仅本地跑');

  test('…', async () => {
    test.setTimeout(900_000);   // 首行，兜底
    // …
  });
});
```

保留 describe 层的 skip（省掉环境里的启动开销），再在用例正文里显式设超时。

**排查「应用莫名关闭」时，先 grep 日志里的 `Test timeout of … exceeded`**，别急着查崩溃日志。

## 一条推论

**测试框架的配置有优先级，而优先级不一定符合直觉。** 遇到「配了但没生效」，先打印框架实际读到的值，再谈配置对不对。
