# R8 混淆会让 JNI 库在 release 版崩

**用到 JNI 按类名找类的 Android 库，必须加 keep 规则。**

## 症状

- **debug 构建**：一切正常，集成测试全过
- **release 构建**：第一次真正跑推理时，整个进程 SIGABRT

```
JNI DETECTED ERROR IN APPLICATION: java_class == null
  in call to GetMethodID
  from ...OrtSession.run(...)
Fatal signal 6 (SIGABRT)
```

## 根因

JNI 代码按**类名字符串**去 `FindClass`（如 `"ai/onnxruntime/OnnxValue"`）。R8 一混淆，类名变成 `g`、`e` 这种短名，JNI 找不到就返回 null，`GetMethodID` 拿到 null 就 abort。

注意：**新版构建链默认就会跑 R8**，即使 `build.gradle.kts` 里没写 `isMinifyEnabled = true`。崩溃栈里那个 `r8-map-id-...` 就是证据。

## 为什么难发现

推理被一个前置门控挡着。测试环境的输入喂不进真实数据，所有窗口都被判为无效、根本不进模型——于是这个崩溃**长期隐身**，直到某次真实输入恰好越过阈值才暴露。

**「某个功能从没被执行过」和「它能正常工作」，在观察上无法区分。**

## 修法

```proguard
-keep class ai.onnxruntime.** { *; }
-keepclassmembers class ai.onnxruntime.** { *; }
-dontwarn ai.onnxruntime.**
-keep class <你的封装包>.** { *; }
```

然后显式声明：

```kotlin
isMinifyEnabled = true
proguardFiles(
    getDefaultProguardFile("proguard-android-optimize.txt"),
    "proguard-rules.pro",
)
```

## 一般规律

**任何用 JNI 按名字找类的库**——ONNX Runtime、部分 TFLite 封装、老式 SDK——开 R8 时都需要 keep 规则。文档里通常有，但很容易漏掉，而漏掉的代价只在 release 版出现。
