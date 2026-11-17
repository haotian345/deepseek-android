# DeepSeek Demo for Android

**基于 DeepSeek 官方 [deepseek-ai/chat-demo](https://github.com/deepseek-ai/chat-demo) 进行整合与 Android 适配。**

这是非官方 Android 整合版本。本仓库目前用于分发测试 APK，不代表 DeepSeek 官方 Android 应用。

## 下载

- [Android 0.3.0 测试版 APK](https://github.com/haotian345/deepseek-android/releases/download/android-v0.3.0-test.1/deepseek-android-v0.3.0-debug.apk)
- [配套自建服务端 v0.3.0（含部署文档）](https://github.com/haotian345/deepseek-android/releases/download/android-v0.3.0-test.1/deepseek-storage-v0.3.0.zip)

- [SHA-256 校验文件](https://github.com/haotian345/deepseek-android/releases/download/android-v0.3.0-test.1/SHA256SUMS-v0.3.0.txt)
- [版本说明与全部下载](https://github.com/haotian345/deepseek-android/releases)

当前版本为调试签名测试版，最低 Android 7.0（API 24）。真机兼容性与真实 API 功能仍需进一步验收。

## 整合内容

- 以官方 chat-demo 为基础，使用 Capacitor 封装 Android 客户端。
- API Key 在本机加密保存；聊天默认本地存储，也可主动选择自己的配套服务器。
- 支持历史 ZIP 导入、实际进度显示及分块导出。
- 导入期间可起草文字，完成后保留草稿并恢复发送。
- 接入 Android 返回、主题、剪贴板及文件分享。
- DeepSeek 标识使用 [LobeHub Icons](https://github.com/lobehub/lobe-icons) 的 SVG 资源。
- 长对话自动摘要压缩（默认约 64K 估算预算），保留原始聊天；可显示进度、中途停止并在重启后复用摘要。
- 可选的跨对话记忆，支持自动提取、手动添加、编辑、删除和清空；默认关闭，入口为「设置 → 记忆」。

聊天生成和相关附件处理依赖 DeepSeek 云端 API，需要使用自己的 API Key。历史导入作用于当前所选存储，不会自动同步到 DeepSeek 账号。

生成摘要和提取记忆会额外调用 DeepSeek API，相关旧聊天或选中的记忆会随请求发送。记忆保存在当前所选存储（本地模式使用独立数据库），现有历史 ZIP 不包含它们；删除聊天不会自动清除独立长期记忆。摘要可能损失细节，本机 token 估算不等于服务端精确用量。

## 安装与更新

下载 APK 后按 Android 提示安装。更新时请覆盖安装；卸载应用会清除本地 Key、设置和历史。

输入 Key 后点击“保存 API Key”，等待验证成功。导入 ZIP 后点击“确认导入”，等待完成提示。

## 服务端部署

从上方下载配套服务端 ZIP 并解压，进入其中的 `server` 目录。下面以手机与服务器在同一可信局域网、通过 `8787` 端口直连为例；公网访问应配置 HTTPS 反向代理。

### 使用 Node.js 启动

1. 安装 Node.js 22.13 或更高版本。服务端没有 npm 依赖，无需运行 `npm install`。
2. 在 `server` 目录将 `.env.example` 复制为 `.env`。

Linux / macOS：

```sh
cp .env.example .env
```

Windows PowerShell：

```powershell
Copy-Item .env.example .env
```

3. 生成独立的存储访问令牌：

```sh
node -e "console.log(require('node:crypto').randomBytes(32).toString('hex'))"
```

4. 编辑 `.env`，将生成的令牌填入 `STORAGE_TOKEN`，并设置以下项目；其余配置保留示例文件默认值：

```dotenv
# 必须填写上一步生成的随机令牌
STORAGE_TOKEN=
HOST=0.0.0.0
PORT=8787
API_PATH=/api/storage/v1
DATA_DIRECTORY=./data
```

`STORAGE_TOKEN` 是 App 访问存储服务的独立令牌，要求 16–256 个不含空格的可打印 ASCII 字符。DeepSeek API Key 在 App 中单独配置。

5. 启动服务，并在服务器防火墙中允许手机所在网络访问 TCP `8787`：

```sh
node --env-file=.env server.mjs
```

保持进程运行。Node 方式的数据默认位于 `server/data`；备份时先停止服务，再备份整个数据目录。`HOST=0.0.0.0` 用于监听网络连接，App 中应填写服务器实际 IP 或域名。

### 使用 Docker Compose 启动

安装 Docker 与 Compose 后，同样先在 `server` 目录创建 `.env` 并填好 `STORAGE_TOKEN`。局域网直连还需将 `.env` 中的 `BIND_ADDRESS` 改为 `0.0.0.0`；默认 `127.0.0.1` 仅供宿主机访问，适合连接本机 HTTPS 反向代理。

```sh
docker compose up -d --build
docker compose logs --tail=20
```

默认端口为 `8787`，数据保存在 Docker 持久卷 `storage-data`。更新时重新执行 `docker compose up -d --build`；停止可用 `docker compose down`。`docker compose down -v` 会删除数据卷。

### 公网访问

为服务配置带有效证书的 HTTPS 域名，并将 App 所用 API 路径原样转发到服务端，例如 `/api/storage/v1`，同时保留 `Authorization` 请求头。服务本身提供 HTTP，由反向代理处理 HTTPS；服务器监听地址和防火墙需允许代理访问。

## App 配置

1. 安装并打开 App，进入「设置 → 数据管理 → 数据存储」。
2. 将待应用的存储方式选择为「服务器」，填写以下内容：

| 字段 | 填写方式 |
| --- | --- |
| 服务器地址 | 局域网示例：`http://192.168.1.10:8787`；HTTPS 示例：`https://chat.example.com`。将示例替换为自己的地址。 |
| 独立 API 路径 | 默认 `/api/storage/v1`，应与服务端 `API_PATH` 及反向代理路径一致。 |
| 存储服务访问令牌 | 填写 `.env` 中的 `STORAGE_TOKEN` 值。 |
| 允许此服务器使用 HTTP | 仅在使用 `http://` 地址时勾选；HTTP 会明文传输令牌和聊天数据，仅用于可信网络。 |

地址栏只填写协议、IP/域名及可选端口，API 路径单独填写。手机上的 `127.0.0.1` 指手机自身，连接电脑时应填写电脑的局域网 IP。

3. 点击「测试连接」，确认成功。如果失败，检查服务是否运行、服务器 IP/端口、防火墙、API 路径及令牌是否一致。
4. 如需把已有本地记录放到服务器，先保持当前生效存储为「本地」，在填好服务器信息后点击「复制本机数据到空服务器」，确认目标并等待完成。目标服务器必须为空；复制包含聊天、已存附件、摘要和长期记忆，本机数据保留。已切到服务器时，可先切回本地再执行复制。
5. 点击「应用配置」。App 会重新加载并使用服务器数据；保存连接配置不会自动复制本地历史。切回本地后可继续查看原来的本机记录。

DeepSeek API Key 仍在 App 中加密保存，配套服务不代理模型调用。服务器模式需要连接存储服务，不提供自动离线同步或多设备自动合并。

### 真机连接服务

在局域网内使用：

1. 手机与运行服务端的电脑或服务器连接同一局域网，并确保网络允许两者互访。
2. Node 启动方式设置 `HOST=0.0.0.0`；Docker 方式设置 `BIND_ADDRESS=0.0.0.0`，并允许手机所在网络访问服务器 TCP `8787` 端口。
3. 在手机 App 中填写服务器实际地址，例如 `http://192.168.1.10:8787`，API 路径填写 `/api/storage/v1`，访问令牌填写服务端的 `STORAGE_TOKEN`；允许 HTTP 后，点击「测试连接」，成功后「应用配置」。将示例 IP 替换成自己的服务器 IP。

需要使用移动网络或在外网访问时，将服务端部署在手机可访问的服务器上，配置 HTTPS 域名，例如 `https://chat.example.com`。App 填写该域名、对应 API 路径和存储访问令牌，再测试连接并应用配置。

## 来源与许可

- 原项目：[deepseek-ai/chat-demo](https://github.com/deepseek-ai/chat-demo)，基线提交 `5d5dd754ebc1948e977ddb880a9eb5aa25f37b4c`。
- 保留原项目 MIT 许可证，APK 中同时包含 LobeHub Icons 的 MIT 许可说明。
- DeepSeek 名称及标识属于其权利人，本项目为基于官方开源项目的非官方整合版本。
