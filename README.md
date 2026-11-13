# World Unlock

一个纯前端的 Minecraft（基岩版）存档解密工具，主要面向网易版存档。所有处理都在浏览器本地完成，文件不会上传到任何服务器。

**在线使用**：https://listudioorg.github.io/world-unlock/

**English**: A browser-based Minecraft (Bedrock) world decryptor, primarily targeting NetEase worlds. Everything runs locally — nothing is uploaded.

---

## 功能特性

- **ZIP 模式**：上传整个存档压缩包，自动定位 `db` 目录，解密后输出一个可直接使用的完整存档包。
- **文件夹模式**：通过 File System Access API 直接读写本地 `db` 目录，原地解密（会覆盖原文件）。
- **中英文切换**：一键切换界面语言，选择保存在本地。
- **明暗主题**：浅色 / 深色主题切换，同样持久化保存。
- **零上传**：不依赖任何后端服务，解密逻辑全部在浏览器内执行。
- **单文件部署**：整个工具就是一个 `index.html`，可托管在任意静态页面服务上。

## 解密原理

网易版存档的核心是 LevelDB 数据库目录（`db`）。该目录下的数据文件被施加了 XOR 流加密，加密文件开头带有固定标识：

```

80 1D 30 01

```

（部分版本为 `90 1D 30 01`）

密钥并非硬编码，而是从存档自身推导：

1. 读取 `CURRENT` 文件的加密数据，跳过前 4 字节标识。
2. 以 `MANIFEST-xxxxxx` 的**文件名**加上一个换行符（`0x0A`）作为种子。
3. 将两者循环异或，得到密钥。
4. 用该密钥对每个带标识的文件跳过前 4 字节后循环异或，即还原明文。

由于加密与解密使用同一个 XOR 操作，整个过程不需要任何外部密钥。

> 注意：部分“资源中心”存档会进行二次加密（特征为 `CURRENT` 大小为 80 字节），本工具无法处理这类存档。

## 使用方法

### 方式一：ZIP 上传（推荐）

1. 将存档打包为 `.zip`。
2. 打开 https://listudioorg.github.io/world-unlock/ ，点击「选择 ZIP 文件」或将文件拖入上传区域。
3. 点击「开始解密」，等待进度完成。
4. 点击「下载解密后的存档」，得到 `原名_decrypted.zip`。

### 方式二：选择文件夹

1. 在支持的环境下（见下文）点击「选择文件夹」。
2. 选择存档中的 `db` 目录。
3. 点击「开始解密」。

> ⚠️ **文件夹模式会直接覆盖原文件，请务必先备份存档。** 该模式会对目录内**所有**带加密标识的文件生效，请确认选择的是目标存档的 `db` 目录。

## 浏览器兼容性

| 功能 | 要求 |
| --- | --- |
| ZIP 模式 | 任意现代浏览器；需能加载 JSZip |
| 文件夹模式 | 桌面版 Chromium 系浏览器（Chrome / Edge 等），且运行在 **HTTPS 或 localhost** 下 |

GitHub Pages 默认就是 HTTPS，文件夹模式可以直接使用。通过 `file://` 直接打开本地文件时，文件夹模式不可用，ZIP 模式仍可正常工作。

### 关于离线使用

页面通过 CDN 加载 [JSZip](https://stuk.github.io/jszip/)：

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js"></script>
```

如果需要完全离线使用，请下载 jszip.min.js 并将上述标签替换为内联的 <script>...</script>。文件夹模式不依赖 JSZip。

本地运行

```bash
git clone https://github.com/LiStudioorg/world-unlock.git
cd world-unlock
```

直接用浏览器打开 index.html 即可使用 ZIP 模式。若需要测试文件夹模式，请用本地服务器并访问 localhost：

```bash
# 任选其一
python3 -m http.server 8080
npx serve .
```

然后访问 http://localhost:8080。

部署

本项目为纯静态页面，无需构建。将 index.html 推送到仓库默认分支后，在仓库 Settings → Pages 中选择从分支部署即可。当前线上地址为：

https://listudioorg.github.io/world-unlock/

常见问题

Q：提示「未找到 CURRENT 文件」？
A：上传的 ZIP 不是有效的存档，或压缩包内未包含 db 目录。请确认压缩包结构。

Q：提示「该存档未加密」？
A：存档本身未加密，或使用了本工具不支持的加密格式（如二次加密的资源中心存档）。

Q：解密后存档仍无法加载？
A：请确认解密的是完整的 db 目录，且 MANIFEST / CURRENT 等关联文件完整。建议优先使用 ZIP 模式，避免原地覆盖带来的风险。

Q：文件夹按钮点不了？
A：当前环境不支持 File System Access API。请使用桌面版 Chromium 系浏览器，并通过 HTTPS 或 localhost 访问。线上 Pages 地址已满足 HTTPS 要求。

免责声明

· 本工具仅供个人备份与存档迁移等合法用途使用。
· 请勿用于绕过付费内容、传播他人存档或任何侵犯他人权益的行为。
· 使用本工具造成的任何数据丢失或损坏，作者不承担责任。操作前请务必备份。
· 本项目与网易、Mojang、Microsoft 均无关联。

致谢

· 感谢 JSZip 提供浏览器端 ZIP 读写能力。
· 感谢开源社区公开分享的存档格式分析资料与讨论。

License

本项目采用 Apache License 2.0 授权，详见 LICENSE。

```
Copyright 2025 LiStudioorg

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
