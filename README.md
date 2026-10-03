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
| **缓存条数显示** | 设置面板里直接显示当前缓存了多少条 |
| **缓存导出 / 导入** | 把缓存导成 JSON 文件备份，随时导入恢复（换浏览器、清数据、重装都可以用） |
| **作品列表分页 / 虚拟渲染** | 一次排序拿到的作品不再全部塞进 DOM，只渲染当前这一页；每页件数与翻页方式都可在设置里选 |
| **空闲卡顿修复** | 修掉了「每秒对全部作品做一次全量扫描」的定时器逻辑——这是页面放着不动、CPU 仍然高企的主因 |
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
| **缓存条数：N（点击刷新）** | — | 显示当前缓存条数，点一下可刷新 |
| **导出缓存** | — | 把缓存下载成一个 `pixiv-bookmark-cache-<时间>.json` 文件 |
| **导入缓存** | — | 选择之前导出的 JSON 合并进来。同一个作品取**时间戳更新**的那条，不会用旧数据覆盖新数据；也接受裸的 `{作品ID: [收藏, 点赞, 浏览, 时间戳]}` 格式 |
| **每页显示的作品数** | `60` | 一次往页面里放多少件作品。填 **`0` 表示不分页、全部渲染**（退回原版行为）。原版是全部渲染，作品一多页面会持续卡顿 |
| **作品列表翻页方式** | `分页器` | `分页器`=DOM 里始终只有当前页（最省资源）；`无限滚动`=滚到底部自动追加下一批，另有「加载更多」按钮兜底 |

---

## 性能：为什么不把作品一次性全部渲染

原版（以及 3.8.7.x 之前的本修改版）会把一次排序拿到的**全部**作品一次性塞进 DOM。
把「每次排序时统计的最大页数」调大（比如 96 页 ≈ 5760 件）会同时踩到两个坑：

1. **DOM 元素过多** —— 几千个元素同时参与布局与合成，滚动和悬停都会变卡，内存能涨到 GB 级。
2. **每秒一次的全量扫描** —— 搜索页被标记为「有自动加载」，脚本注册了一个每 1000ms 执行的
   定时器用于发现 pixiv 懒加载出来的新作品。它每次都遍历容器里的**每一个**元素，各做多次
   DOM 查询（`find('a')` / `find('svg')` / `find('span')` / `attr` / `addClass`），最后再构造
   一个等长的 jQuery 集合。作品多时单次就要几百毫秒，而绝大多数轮次的结论都是「没有变化」——
   **这才是页面放着不动 CPU 仍然高企、切到后台也降不下来的直接原因**（它也让性能分析工具
   很难看出问题：耗时都是几十毫秒的碎片，不构成单个长任务）。

3.8.8.0 的改法：

- 作品列表改成**按页渲染**，DOM 里只保留「每页显示的作品数」那么多元素（默认 60）
- 那个每秒的定时器加了**前置判断**：先数元素个数，没变化就直接返回，不做全量扫描
  （个数变化时才真正扫描，所以 pixiv 的懒加载检测依然有效）
- 列表中的收藏按钮、作者卡片改为**事件委托**，翻页时不需要重新绑定几千个监听器

实测：元素个数不变时，一小时内对该列表的扫描次数从 **3601 次降到 1 次**。

> 想要原来「一屏到底」的体验，把「每页显示的作品数」填 `0` 即可，
> 但会退回上面两个坑，建议只在数据量小时这么用。非数字、负数或留空都会回退到默认值 60。

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

**清之前想留个底？** 先点 **导出缓存**，会下载一个 JSON 文件；以后任何时候点 **导入缓存** 选它就能恢复（合并式，不会丢掉期间新攒的条目）。

### 淘汰与容量

- 每次保存前清理**已过期**条目（选 `永不更新` 时不清理）
- 超过 **20000 条**时按时间戳淘汰最旧的
- 若 localStorage 写入因超配额失败，会砍掉一半最旧的自动重试一次

实测数据（Chromium，单条约 40 字符）：

| 条数 | 占用 | 一次完整落盘 | 启动读取 |
|---|---|---|---|
| 2,551 | 91 KB | ~1.5 ms | ~0.4 ms |
| 20,000 | 718 KB | ~11 ms | ~3 ms |
| 60,000 | 2.1 MB | ~46 ms | ~12 ms |

Firefox 的 localStorage 每域名配额默认约 **5 MB**，按上面的密度理论上能装约 **12 万条**——上限 20000 是很保守的取值，正常情况下远够用。

---

## 常见问题

**Q：为什么我的缓存只有一两百条？**
排序时只统计「每次排序时统计的最大页数」× 60 左右的作品，一次搜索就这么多。多搜几次会累积，且**按作品 ID 跨搜索共享**。

**Q：为什么打开作品详情页还有请求？**
pixiv 改用 Next.js 后，作品数据不再内嵌在页面里（老版的 `<meta name="preload-data">` 已移除），所以详情页必须请求一次 `/ajax/illust/<id>`。缓存新鲜时会跳过，不再重复请求。

**Q：会不会越用越慢？**
不会。缓存读取是一次 JSON 解析 + 哈希查找；写入按 3 秒防抖合并，排序结束后一次性落盘。

进度文字的刷新做了限流（最快 120 ms 一次）：命中缓存的作品是在微任务里连续处理的，如果每条都去写一次界面，几千条会挤在一起把主线程堵住。

**Q：换了界面语言，缓存档位设置会丢吗？**
不会。档位的**存储值**与语言无关，只有显示文字会变。

---

## 已知限制

- **未经过完整实跑验证**：缓存写入、列表图标已在真实环境观察到生效；但详情页 📌 的实际落位、四个开关的联动、以及缓存达到上限时的淘汰行为，**尚未在真实使用中完整跑过一轮**。
- 详情页的 📌 图标是注入到 pixiv 的 React 渲染子树里的，若 pixiv 重新渲染那个节点（例如你点了收藏），图标可能消失——不影响功能，刷新即可。
- 缓存上限 20000 是按 Firefox 默认配额（约 5 MB）取的保守值，实测单条约 40 字符，理论上限约 12 万条。
- 导出/导入依赖浏览器的下载与文件选择；若浏览器拦截下载，需在地址栏允许本网站下载。
- 导入时**不会**校验作品 ID 是否真实存在，也不会校验收藏数是否合理——只按时间戳决定是否覆盖。
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

The settings panel also shows the current cache size and can **export / import** the cache as a JSON file (useful as a backup — merge on import keeps the newer entry per artwork).

**Performance (3.8.8.0)**: the work list is now rendered page by page instead of dumping every
artwork into the DOM at once, and the search page's 1-second auto-load poll no longer re-scans the
whole list on every tick (it first compares the child count and bails out when nothing changed —
measured: 3601 scans per hour down to 1). Both were the cause of high idle CPU and memory growth
when a large "maximum pages per sort" was used. Settings: *works per page* and *paging mode*
(paginator / infinite scroll).

Licensed under **GPLv3**, same as upstream. No warranty.
