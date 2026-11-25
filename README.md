# vibelab-test-unity

VibeLab 提交测试工程 5：**Unity 导出工程用例**。

- Unity 6000.5.6f1（LTS 分支）导出，Scripting Backend IL2CPP，仅 arm64-v8a
- 导出的 Android Gradle 工程，`launcher` 为唯一 Application module，`unityLibrary` 为 Library module
- `libil2cpp.so` 已在本机预编译并随工程提交（`unityLibrary/src/main/jniLibs/arm64-v8a/`，9.2 MB），`Il2CppOutputProject/` 已移除
- `unityLibrary/build.gradle` 末尾禁用了 `buildIl2Cpp` 任务，平台构建时不会再次编译 IL2CPP
- AGP 9.0.0 / Gradle 9.1.0（Unity 6000.5 模板自带），compileSdk 36 / targetSdk 36 / minSdk 26
- 包名 `com.tuomimi.vibelabunity`，release 沿用模板默认 debug signing，无 keystore、无 `local.properties`
- 指向本机 Unity 安装目录的路径（`unity.androidSdkPath` / `unity.androidNdkPath` / `unity.jdkPath`、两个 module 的 `ndkPath`）保持原样，由平台在构建时处理

## 本机终验

`gradle :launcher:assembleRelease`（显式设置 `ANDROID_HOME`）→ `BUILD SUCCESSFUL`，日志含 `> Task :unityLibrary:buildIl2Cpp SKIPPED`；APK 内 `libil2cpp.so` 的 SHA-256 与 `jniLibs` 中预编译产物一致（`cc140927f16d650f8f559ee8806eaffb98e9248ce7849848011ae86a5f3374f5`），LOAD 段对齐 0x4000。

APK 内的 `libgame.so`、`libswappywrapper.so`、`libc++_shared.so` 由平台构建时现场编译（CMake 链路），不入本仓库。

用途：验证 Unity 那一档提交链路——平台只跑 Gradle 打包、不运行 Unity 工具。若原生工程那几套用例全部通过、本用例在 `buildIl2Cpp` 或 IL2CPP 相关步骤失败，问题在平台对 Unity 导出的处理。
