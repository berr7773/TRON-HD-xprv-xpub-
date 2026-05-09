# TRON HD Offline xprv/xpub Tool

单文件离线 TRON HD 钱包工具。可以在浏览器中直接打开 `tron-hd-offline.html`，输入或生成 BIP39 助记词，按 TRON 标准路径派生账户层和收款链层的 `xprv` / `xpub`，并预览指定 index 的 TRON 地址。

线上分发地址：

[https://tron-hd-offline.pages.dev/](https://tron-hd-offline.pages.dev/)

重要：线上地址只适合分发和验证工具。真实资产助记词、`xprv`、私钥建议下载 HTML 后断网打开使用，不要在联网状态输入。

## 功能

- 单文件 HTML，无后端、无外链、无网络请求、无本地存储。
- 中英文界面切换，默认按浏览器语言选择。
- 支持英文、简体中文、繁体中文 BIP39 助记词校验和生成。
- 支持 12 / 15 / 18 / 21 / 24 词 BIP39 助记词导入。
- 支持离线生成 12 词或 24 词助记词。
- 支持可选 BIP39 passphrase，默认留空。
- 输出账户层 `m/44'/195'/account'` 的 `xprv` / `xpub`。
- 输出收款链层 `m/44'/195'/account'/0` 的 `xprv` / `xpub`。
- 支持自定义地址起始 index，从 `m/44'/195'/account'/0/address_index` 开始预览地址。
- 默认隐藏 `xprv` 和私钥，确认后才显示。
- 内置自检，覆盖 BIP39、BIP32、TRON 默认路径和自定义地址 index。

## 路径说明

TRON 的 SLIP44 coin type 是 `195`。本工具默认使用：

```text
账户层:     m/44'/195'/account'
收款链层:   m/44'/195'/account'/0
地址路径:   m/44'/195'/account'/0/address_index
```

默认 `account = 0`、`address_index = 0` 时，第一条地址路径是：

```text
m/44'/195'/0'/0/0
```

## 上传 xpub / xprv 时选哪一层

如果你的系统用于批量生成子地址、自动归集、签名、扫款，通常建议上传收款链层：

```text
m/44'/195'/account'/0
```

原因：

- 收款链层 `xpub` 可以直接派生 `/0`、`/1`、`/2` 等收款子地址。
- 收款链层 `xprv` 可以签名这些收款子地址，用于归集。
- 权限范围比账户层更窄，更贴近“批量收款地址 + 定时归集”的业务场景。

账户层 `m/44'/195'/account'` 也可以继续派生出 `/0/i` 并扫描资产，但它权限范围更大。除非你的系统明确要求从账户层派生多个分支，否则不要优先上传账户层 `xprv`。

## BIP39 多语言提醒

本工具支持英文、简体中文、繁体中文 BIP39 词表，但要注意：

- 同一套钱包必须使用原来的助记词语言、助记词文本和 passphrase 恢复。
- 不能把英文助记词直接翻译成中文助记词来恢复同一个钱包。
- 即使两个助记词来自同一份 entropy，只要助记词文本不同，BIP39 seed 通常也不同，因为 seed 使用助记词文本参与 PBKDF2 计算。

因此，备份时请保存原始助记词文本，不要把跨语言重新编码当作同一个钱包的备份。

## 快速使用

### 本地离线使用

1. 下载或复制 `tron-hd-offline.html`。
2. 断开网络。
3. 用浏览器直接打开该 HTML 文件。
4. 点击“运行内置自检”，确认通过。
5. 输入或生成助记词。
6. 设置账户 index、地址起始 index、地址预览数量。
7. 点击“派生 xprv/xpub”。

### 自定义地址 index

示例：

- 地址起始 index = `0`，数量 = `5`，生成 `/0` 到 `/4`。
- 地址起始 index = `1000`，数量 = `3`，生成 `/1000`、`/1001`、`/1002`。
- 数量 = `1` 时，只推导指定那一个地址。

输入范围：

- 账户 index：`0` 到 `2147483647`
- 地址起始 index：`0` 到 `2147483647`
- 地址预览数量：`1` 到 `20`

如果 `地址起始 index + 地址预览数量 - 1` 超过 `2147483647`，工具会拒绝派生。

## Cloudflare Pages 部署

本项目是静态 HTML，推荐使用 Cloudflare Pages Direct Upload，不需要 Worker、KV、D1 或后端服务。

当前部署目录为：

```text
cloudflare-dist/
  index.html
  _headers
```

其中 `index.html` 来自 `tron-hd-offline.html`，`_headers` 用于给线上页面补充 CSP、`Referrer-Policy`、`X-Content-Type-Options` 等安全响应头。

重新准备部署目录：

```powershell
Remove-Item .\cloudflare-dist -Recurse -Force
mkdir .\cloudflare-dist
Copy-Item .\tron-hd-offline.html .\cloudflare-dist\index.html
```

首次登录：

```powershell
npx wrangler login
```

创建 Pages 项目：

```powershell
npx wrangler pages project create tron-hd-offline --production-branch=main
```

部署：

```powershell
npx wrangler pages deploy .\cloudflare-dist --project-name tron-hd-offline --branch main
```

官方文档：

- [Cloudflare Pages Direct Upload](https://developers.cloudflare.com/pages/get-started/direct-upload/)
- [Wrangler Pages commands](https://developers.cloudflare.com/workers/wrangler/commands/pages/)

## 验收检查

部署或修改后建议检查：

- 直接打开 `tron-hd-offline.html` 可以运行。
- 打开线上地址可以正常显示页面。
- 点击“运行内置自检”显示通过。
- 使用公开测试助记词：

```text
abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon abandon about
```

在默认路径 `m/44'/195'/0'/0/0` 下，应得到地址：

```text
TUEZSdKsoDHQMeZwihtdoBiN46zxhGWYdH
```

- 地址起始 index = `2`、数量 = `1` 时，第一行路径应为 `m/44'/195'/0'/0/2`。
- 地址起始 index = `1000`、数量 = `3` 时，应生成 `/1000`、`/1001`、`/1002`。
- 浏览器 DevTools 的 Network 面板不应出现外部脚本、字体、图片或 API 请求。
- `xprv` 和私钥默认隐藏，点击确认后才显示。
- “清空/重置”会清除当前页面状态。

## 安全说明

- `xprv` 等同于对应路径下的签名权限，泄露后可导致资产被转走。
- 收款链层 `xprv` 可控制该收款链下所有子地址。
- `xpub` 不能直接转账，但会暴露该链下所有派生地址的隐私。
- 处理真实资产前，建议在可信设备上断网打开本地 HTML。
- 使用完真实助记词或私钥后，建议关闭浏览器窗口。
- 不要把真实助记词粘贴到不可信网页、聊天窗口、远程桌面或日志系统中。

## 标准

- BIP39 mnemonic / seed
- BIP32 HD key and standard `xprv` / `xpub` serialization
- BIP44 path structure
- SLIP44 TRX coin type `195`
- TRON address: secp256k1 public key -> Keccak-256 -> last 20 bytes -> `0x41` prefix -> Base58Check

## 文件结构

```text
tron-hd-offline.html      主工具文件，可直接离线打开
cloudflare-dist/          Cloudflare Pages Direct Upload 部署目录
README.md                 项目说明
```

