# DeepSeek Demo for Android

**基于 DeepSeek 官方 [deepseek-ai/chat-demo](https://github.com/deepseek-ai/chat-demo) 进行整合与 Android 适配。**

这是非官方 Android 整合版本。本仓库目前用于分发测试 APK，不代表 DeepSeek 官方 Android 应用。

## 下载

- [Android 0.2.0 测试版 APK](https://github.com/haotian345/deepseek-android/releases/download/android-v0.2.0-test.1/deepseek-android-v0.2.0-debug.apk)
- [SHA-256 校验文件](https://github.com/haotian345/deepseek-android/releases/download/android-v0.2.0-test.1/SHA256SUMS-v0.2.0.txt)
- [版本说明与全部下载](https://github.com/haotian345/deepseek-android/releases)

当前版本为调试签名测试版，最低 Android 7.0（API 24），已在 MuMu / Android 15 上验证。真机兼容性与真实 API 功能仍需进一步验收。

## 整合内容

- 以官方 chat-demo 为基础，使用 Capacitor 封装 Android 客户端。
- API Key 在本机加密保存，历史记录与导入数据保存在应用本地。
- 支持历史 ZIP 导入、实际进度显示及分块导出。
- 导入期间可起草文字，完成后保留草稿并恢复发送。
- 接入 Android 返回、主题、剪贴板及文件分享。
- DeepSeek 标识使用 [LobeHub Icons](https://github.com/lobehub/lobe-icons) 的 SVG 资源。
- 长对话自动摘要压缩（默认约 64K 估算预算），保留原始聊天；可显示进度、中途停止并在重启后复用摘要。
- 可选的跨对话记忆，支持自动提取、手动添加、编辑、删除和清空；默认关闭，入口为「设置 → 记忆」。

聊天生成和相关附件处理依赖 DeepSeek 云端 API，需要使用自己的 API Key。导入历史在本机完成，不会自动同步到 DeepSeek 账号。

生成摘要和提取记忆会额外调用 DeepSeek API，相关旧聊天或选中的记忆会随请求发送。记忆保存在本机独立数据库，现有历史 ZIP 不包含它们；删除聊天不会自动清除独立长期记忆。摘要可能损失细节，本机 token 估算不等于服务端精确用量。

## 安装与更新

下载 APK 后按 Android 提示安装。更新时请覆盖安装；卸载应用会清除本地 Key、设置和历史。

输入 Key 后点击“保存 API Key”，等待验证成功。导入 ZIP 后点击“确认导入”，等待完成提示。

## 来源与许可

- 原项目：[deepseek-ai/chat-demo](https://github.com/deepseek-ai/chat-demo)，基线提交 `5d5dd754ebc1948e977ddb880a9eb5aa25f37b4c`。
- 保留原项目 MIT 许可证，APK 中同时包含 LobeHub Icons 的 MIT 许可说明。
- DeepSeek 名称及标识属于其权利人，本项目为基于官方开源项目的非官方整合版本。
