# 影视源配置 · 网页部署包

把 JSON 源配置变成一个可访问的网址，既能给播放器导入，也能当成一个看得见的页面分享。

## 包里有什么

```
空壳源网页包/
├── config.json   ← 播放器真正读取的配置（24 个点播源，隐私码已改为 kongke2026）
├── index.html    ← 展示页，读取同目录 config.json，显示隐私码 + 源清单
└── README.md
```

两个文件必须放在**同一个目录**下，`index.html` 靠相对路径 `./config.json` 取数据。

---

## 方案 A：GitHub Pages（5 分钟，免费）

1. 新建仓库，比如 `my-vod-source`，可见性选 **Public**（Private 仓需要登录才能访问 raw，播放器取不到）。
2. 把 `config.json`、`index.html` 一起上传到仓库根目录。
3. 仓库 → Settings → Pages → Source 选 **Deploy from a branch** → 分支 `main`、目录 `/ (root)` → Save。
4. 等 1~2 分钟，得到两个地址：
   - 展示页：`https://<你的用户名>.github.io/my-vod-source/`
   - 配置地址：`https://<你的用户名>.github.io/my-vod-source/config.json`

播放器里填的是**配置地址**（`config.json`），不是展示页地址。展示页只是给人看的。

## 方案 B：不上传页面，只要一个 URL

如果只想给播放器用，不需要展示页，直接托管 `config.json` 即可：

| 方式 | 地址形态 | 备注 |
|---|---|---|
| GitHub raw | `https://raw.githubusercontent.com/<用户>/<仓>/main/config.json` | 国内可能不稳，加代理前缀如 `https://ghfast.top/` 拼在前面 |
| jsDelivr | `https://cdn.jsdelivr.net/gh/<用户>/<仓>@main/config.json` | 有 CDN 缓存，改完配置要等几分钟生效 |
| Cloudflare Pages | `https://<项目名>.pages.dev/config.json` | 和 Pages 一样，拖文件夹上传即可 |
| 自己的服务器 / 对象存储 | `https://你的域名/config.json` | 注意设置 `Content-Type: application/json`，且不要开鉴权 |

## 方案 C：本地先验

双击 `index.html` 会因为浏览器限制读不到 `config.json`，页面会提示"未找到"。想看效果就在目录下执行：

```bash
python3 -m http.server 8080
```

然后访问 `http://127.0.0.1:8080/`。手机连同一 WiFi 时，把地址里的 `127.0.0.1` 换成电脑的局域网 IP 也能打开。

---

## 隐私码到底是怎么起作用的

它不是登录密码，而是配置里的一个普通字段：

```json
"privateKey": "kongke2026"
```

- **谁能看到**：任何人打开 `config.json` 这个链接都能看到这行字，展示页也会把它显示在右上角——这是设计如此，不是漏洞。
- **它控制什么**：`blockTypeName` 里列出的分类（伦理片、理论片、福利等）默认隐藏，在 App 里输入这个码之后才显示。
- **公开分享意味着**：别人不需要你告诉，也能从配置文件里读到码。它只适合"圈内半公开"的分发，别当访问凭证用。

**换成你自己的码**：改 `config.json` 里的 `privateKey` 一处即可，展示页会自动显示新值，不用改 HTML。建议用纯字母数字，别用中文或特殊符号，部分播放器对非 ASCII 婉码支持不稳。

---

## 常见问题

**播放器提示"配置加载失败"**
先把这个 `config.json` 地址粘到浏览器地址栏，能直接看到一屏 JSON 才算对。如果浏览器也 404，是 Pages 没生效或路径写错；如果浏览器能看、播放器不能，多半是仓库设成了 Private。

**改了配置但播放器还是旧的**
jsDelivr 和部分 CDN 会缓存几分钟到几小时。换 raw 地址，或在地址后面加 `?v=20261125` 强制刷新。

**页面显示"未找到 config.json"**
两个文件不在同一目录，或者你打开的是 `file://` 本地路径。按方案 C 起个本地服务。

**点了复制没反应**
浏览器限制 `file://` 协议下的剪贴板权限。部署到 https 域名后就正常。

**想加直播源**
在 `config.json` 里把 `tvSources` 从 `[]` 填成数组，格式是 `{"name":"...","m3uUrl":"...","block":false}`，`m3uUrl` 必须是 m3u 链接。展示页会自动统计直播源数量。

---

## 一句提醒

`desc` 里写的"片库约 35 万"是采集接口自报的数量，不是实测能播的片数；接口域名随时会换，失效了把 `url` 换成同站新域名就行，其余字段不用动。
