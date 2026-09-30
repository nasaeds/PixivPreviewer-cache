# Pixiv Previewer Cache Mod

> **这是 [Ocrosoft/PixivPreviewer](https://github.com/Ocrosoft/PixivPreviewer) 的第三方修改版（二改），不是原作。**
> 本仓库是该项目的一个 **Fork**：<https://github.com/nasaeds/PixivPreviewer-cache>
> 原作者：**Ocrosoft** · 许可：**GPLv3** · 基于原版 **3.8.7**

在搜索页按收藏数排序时，原版每次都会重新请求每个作品的收藏数。pixiv 对此有速率限制，所以每次重排都要等很久。本修改版**把收藏数缓存到本地**，重复搜索时直接读本地值，省去绝大部分请求；另外加了深浅色主题和缓存状态标记。

---

> ## ⚠️ AI 生成声明
>
> **本修改版相对原版的全部改动，均由 AI 编程助手生成，不是人工逐行编写的。**
>
> - 原版代码（PixivPreviewer 3.8.7）由原作者 **Ocrosoft** 人工编写。
> - 本仓库新增与修改的部分（收藏数缓存、淘汰策略、深浅色主题、缓存状态标记、新增设置项等）由
>   **AI 编程助手（Hermes Agent）生成**。
> - 这些改动经过静态语法检查、单元测试与无头浏览器渲染验证，并在 AI 自查中修复了发现的缺陷；
>   但**未经完整的真实使用验证**，详见下方「已知限制」。
> - 请在安装前自行评估是否符合你的需求，**风险自负**。

---

## 相比原版新增

| 功能 | 说明 |
|---|---|
| **收藏数本地缓存** | 缓存到 localStorage，带时间戳与有效期；命中则不再发请求 |
| **缓存有效期可选** | 不缓存 / 1天 / 3天 / 5天 / 7天 / 30天 / 60天 / 半年 / 1年 / 永不更新（默认 7 天） |
| **淘汰策略** | 保存时清理过期条目；超过 20000 条时按最旧淘汰；localStorage 写满时自动砍半重试 |
| **缓存状态标记** | 列表里数值后带 ⚡ 表示来自缓存、📌 表示本次新抓到并已缓存；详情页收藏数后带 📌 表示该作品已在缓存中 |
| **详情页 / 预览反哺缓存** | 打开作品详情页、或使用脚本预览时，会把看到的最新收藏数写回缓存 |
| **预览写入开关** | 可分别控制「搜索页以外的预览」和「搜索页预览」是否记录，另有「仅未缓存时获取」选项 |
| **深浅色主题** | 设置面板右上角圆形按钮切换，选择会记住 |
| **档位本地化** | 缓存有效期下拉框的显示文字跟随界面语言（存储值不变，换语言不丢配置） |

---

## 安装

1. 安装 [Tampermonkey](https://www.tampermonkey.net/)（或 Violentmonkey / Greasemonkey）。
2. 点这个链接直接安装 / 更新：

   ```
   https://raw.githubusercontent.com/nasaeds/PixivPreviewer-cache/master/pixiv-previewer-cache.user.js
   ```

   装好后 Tampermonkey 会通过脚本里的 `@updateURL` 自动检查本仓库的更新。

   - 如果 `raw.githubusercontent.com` 访问不了，可以在仓库页打开 [`pixiv-previewer-cache.user.js`](./pixiv-previewer-cache.user.js) 点 **Raw**，或整个复制内容新建脚本。

> **注意**：本修改版的 `@namespace` 与原版不同，因此可以**与原版共存**，不会互相覆盖。若你已装原版，建议先禁用原版，避免两套脚本同时操作同一个搜索页。

---

## 新增设置项

都在设置面板的「排序」分区里：

| 设置项 | 默认 | 说明 |
|---|---|---|
| **收藏数缓存有效期** | `7天` | 超过这个时间的缓存视为失效，会重新获取。选 `不缓存` 等于关闭整个缓存功能 |
| **预览时记录收藏数（搜索页以外）** | 开 | 主开关。关闭后，搜索页以外的预览不再写入缓存 |
| **搜索页预览也记录收藏数** | 开 | 子开关，**仅在主开关开启时可交互**。搜索页的作品排序时基本已经抓过，通常不需要再补 |
| **仅在未缓存时获取收藏数** | 开 | 开启=预览只补未缓存的作品；关闭=每次预览都重新获取并覆盖 |

---

## 收藏数缓存的机制

### 存储位置

- **key**：`PixivPreviewBookmarkCache`（存在 pixiv.net 这个域名的 localStorage 里）
- **结构**：`{ "作品ID": [收藏数, 点赞数, 浏览数, 时间戳] }`

想在浏览器里直接看，F12 → Console：

```js
JSON.parse(localStorage.getItem('PixivPreviewBookmarkCache'))
```

### 什么时候会写缓存

| 场景 | 行为 |
|---|---|
| 搜索页排序 | 命中缓存则直接使用（不发请求）；未命中则请求一次并写入 |
| 打开作品详情页 | 缓存新鲜则跳过；缺失或过期则请求一次并覆盖 |
| 悬停预览作品 | 同上，但受上面三个开关控制 |

三条路径写的是**同一条记录**（每个作品一个键），后写入的整条覆盖先前的。

### 图标含义

| 图标 | 位置 | 含义 |
|---|---|---|
| ⚡ | 搜索结果的收藏数后 | 这个数值**来自缓存**（本次没有发请求） |
| 📌 | 搜索结果的收藏数后 | 这个数值是**本次新抓取并已写入缓存**的 |
| （无） | 搜索结果的收藏数后 | 未缓存，或缓存功能已关闭 |
| 📌 | 详情页收藏数后 | 这件作品的收藏数**已在缓存中** |

### 怎么清缓存

设置面板 →「排序」分区 → **清除缓存** 按钮（会一并清除"关注画师"缓存）。清除后缓存会在下次搜索时重新积累。

### 淘汰与容量

- 每次保存前清理**已过期**条目（选 `永不更新` 时不清理）
- 超过 **20000 条**时按时间戳淘汰最旧的（约 1–1.5 MB）
- 若 localStorage 写入因超配额失败，会砍掉一半最旧的自动重试一次

---

## 常见问题

**Q：为什么我的缓存只有一两百条？**
排序时只统计「每次排序时统计的最大页数」× 60 左右的作品，一次搜索就这么多。多搜几次会累积，且**按作品 ID 跨搜索共享**。

**Q：为什么打开作品详情页还有请求？**
pixiv 改用 Next.js 后，作品数据不再内嵌在页面里（老版的 `<meta name="preload-data">` 已移除），所以详情页必须请求一次 `/ajax/illust/<id>`。缓存新鲜时会跳过，不再重复请求。

**Q：会不会越用越慢？**
不会。缓存读取是一次 JSON 解析 + 哈希查找；写入按 3 秒防抖合并，排序结束后一次性落盘。

**Q：换了界面语言，缓存档位设置会丢吗？**
不会。档位的**存储值**与语言无关，只有显示文字会变。

---

## 已知限制

- **未经过完整实跑验证**：缓存写入、列表图标已在真实环境观察到生效；但详情页 📌 的实际落位、四个开关的联动、以及缓存达到上限时的淘汰行为，**尚未在真实使用中完整跑过一轮**。
- 详情页的 📌 图标是注入到 pixiv 的 React 渲染子树里的，若 pixiv 重新渲染那个节点（例如你点了收藏），图标可能消失——不影响功能，刷新即可。
- 缓存上限 20000 是估计值，未实测过 pixiv.net 的真实 localStorage 配额。
- 预览会为**未缓存**的作品各增加 1 个请求；如果你正被限速，可关掉相关开关。
- 收藏数来自 pixiv 网页端接口，pixiv 改动接口则脚本需要跟进。

---

## 许可与致谢

本项目以 **GNU General Public License v3.0** 发布，与原项目一致。完整许可文本见 [LICENSE](./LICENSE)。

- 原项目：[Ocrosoft/PixivPreviewer](https://github.com/Ocrosoft/PixivPreviewer)（作者 Ocrosoft，GPLv3）
- 本修改版在原版 **3.8.7** 基础上修改，修改日期见脚本头部声明。
- 本程序不含任何担保。

原版依赖的第三方库（GM_config、gif.js 等）通过 `@require` 在运行时加载，未随本仓库分发。

---

## English

A modified fork of [PixivPreviewer](https://github.com/Ocrosoft/PixivPreviewer) (by **Ocrosoft**, GPLv3), based on upstream 3.8.7.

The upstream script re-fetches every artwork's bookmark count on each sort, which pixiv rate-limits. This fork **caches bookmark counts in localStorage** with a configurable TTL, so repeat searches read local values instead of hitting the API. Also adds light/dark themes and cache-status markers (⚡ = value read from cache, 📌 = value just fetched and cached).

Licensed under **GPLv3**, same as upstream. No warranty.
