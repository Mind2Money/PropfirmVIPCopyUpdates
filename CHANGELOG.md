# 更新日志 · Changelog

## v1.6.1 (2026-10-22)

## 简体中文

高缩放比例的笔记本：窗口启动时不再超出屏幕（默认尺寸放不下就最大化，最小尺寸降为 880×600）；显示密度新增默认的「跟随窗口」，较小或较矮的窗口自动使用紧凑模式；侧栏品牌名过长时换行显示。
跟单组编辑器的主账户与跟随账户下拉框显示账户净值、同步状态以及该账户已在哪些跟单组中担任主账户或跟随（未加入则标明）。
修复 Pine 策略市价入场模式下“一笔交易结束后下一笔不下单”的问题：同一入场 ID 的新交易在前一笔仍在与券商核对（止损/止盈是否真的成交）时，其模拟成交被当成旧记录的成交而丢弃，账户空仓而模拟持仓。现在每条镜像记录记住所属交易序号，新交易的市价入场等旧交易核对完成（未成交的真实出场先按市价平掉）后再发送；预挂单模式下在挂单送达券商前就成交的模拟入场同样按市价入场，并记为 `ENTRY_FILLED_BEFORE_RESTING`。
修复下单后紧接着的多余改价：`math.round_to_mintick` 与按跳数计算的止盈止损价带有二进制浮点残差（如 4089.1000000000004），券商回报 4089.1 时被当作价格变化而重复改价，导致 CONFIRMATION_TIMEOUT、`MODIFY_UNCONFIRMED` 与 ExecutionStopped 拒绝。现在这些价格按合约最小跳动取整，镜像发送和比较的价格也按跳动对齐。
修复市价入场的 OSO（入场＋止盈＋止损）复制到跟随账户后保护单被软件自己撤掉的问题：跟随账户的 OSO 刚提交、券商仍报告为 PendingNew 时，若恰好赶上核对（启动后每 30 秒一次）或主账户改单，退出容量按“入场未成交、保护单未挂起”算成 0，两条保护单以 `exit-capacity` 撤销，持仓失去保护。现在券商确认前的 OSO 保护单、以及入场成交已到而持仓回报未到时的保护单，不再按容量改小或撤销；容量撤单在发送前会重新核对。

## English

Laptops at high scaling: the window no longer starts larger than the screen (it opens maximized when the default size does not fit; the minimum is now 880×600), the new default density "Follow window" switches small or short windows to compact automatically, and a long brand name wraps in the sidebar.
The copy-group editor's leader and follower drop-downs show each account's equity, sync state and the groups it already leads or follows (or that it is in none).
Fixed Pine strategies in market entry mode not placing the next trade after one ended: when the script re-entered under the same entry id while the earlier trade was still being verified against the broker (did its stop or target really fill?), the new simulated fill was taken for the old record's and dropped, leaving the account flat while the emulator held a position. Every mirror record now remembers the trade serial it mirrors; the new trade's market entry waits until the ended trade is settled (an unfilled real exit is closed at market first) and is then sent. In resting mode a simulated entry that fills before its order reached the broker now enters at market the same way, noted as `ENTRY_FILLED_BEFORE_RESTING`.
Fixed the spurious price changes sent right after an entry: `math.round_to_mintick` and tick-based profit/loss levels carried binary floating-point residue (4089.1000000000004) that looked like a changed price once the broker reported 4089.1, causing CONFIRMATION_TIMEOUT, `MODIFY_UNCONFIRMED` and ExecutionStopped rejections. These prices now snap to the contract tick, and the mirror sends and compares tick-aligned prices.
Fixed the app cancelling the follower's protection of a copied market-entry OSO (entry with target and stop): when a reconciliation (every 30 seconds after a start) or a leader change ran while Tradovate still reported the just-placed OSO as PendingNew, the exit capacity was read as zero (entry unfilled, brackets not yet suspended) and both brackets were cancelled as `exit-capacity`, leaving the position unprotected. OSO brackets the broker has not acknowledged yet, and protection whose entry fill arrived before the position report, are no longer shrunk or cancelled for capacity; a capacity cancel is checked again just before it is sent.

## Commits

- chore: release v1.6.1 (447fe11)
- test: defer a source change made while the follower OSO is unacknowledged (5a26698)
- fix: keep native OSO brackets the broker has not acknowledged yet (3a459b0)
- feat: fit small logical screens (laptops at high scaling) (3c50125)
- feat: show equity, sync state and group membership in the group editor account lists (5e38b95)
- fix: place the next Pine market entry after a trade ends and stop residue modifies (e96d317)
- chore: point updatesInfo at v1.6.0 (a15b7e2)

## v1.6.0 (2026-10-22)

## 简体中文

浮盈浮亏改按平仓方向的买卖价计算：多头按买一价、空头按卖一价标记，不再使用最新成交价（最新成交可能落在价差另一侧，市价平仓的实际成交会低于按最新成交价估算的数值）。
缺少该侧报价时持仓视为未标记（等待行情），日盈亏上限改用券商资金快照判断，不会用最新成交价顶替；只含最新成交的报价沿用上一次的买卖价。
日盈亏上限的触发记录新增触发时各持仓的标记方向、标记价、最新成交价与持仓均价，便于与平仓结果比对。

软件加固：改为专有许可（不再是 AGPL）。私有引擎在构建期先混淆改名再 AES-256-GCM 加密，运行时在可执行文件内解密，公开发布不含可读源码；每次构建产出的反混淆
sourcemap 作为私有“map 表”，只随私有仓库发布保留，绝不进入公开更新源。首次启动新增许可协议确认（拒绝即退出），接受记录带版本与时间戳。构建默认用 PowerShell 对可执行文件自签名（自签名不受信任，SmartScreen 仍会提示未知发布者；可用 `BT_SIGN_PFX`/`BT_SIGN_THUMBPRINT` 换真证书，或 `BT_SIGN=off` 关闭）。可执行文件经单文件压缩与 gzip 引擎覆盖层压缩，体积约 108MB。

产品名统一为 PropFirmVIP Tradovate：可执行文件为 `PropFirmVIP-Tradovate.exe`，引擎为 `PropFirmVIP-Tradovate.Engine.exe`，窗口标题、通知、浏览器登录临时目录、诊断日志文件名与
`%LOCALAPPDATA%\\PropFirmVIP-Tradovate` 偏好/引擎缓存目录随之更名（旧目录的 desktop.json 首次启动时自动沿用）。

修复网页登录成功后因“登录浏览器清理未完成”而登录失败的问题：浏览器关闭后先以调试端口确认已退出，只有端口仍响应或文件持续占用时才调用 PowerShell
进程清理，因此 PowerShell 被拦截、缓慢或损坏的电脑也能完成登录；临时目录删除会在占用期间限时重试。浏览器已关闭而目录暂时删不掉时，登录照常完成，
连接页列出该目录，可以管理员身份删除（UAC 确认）或打开所在文件夹手动删除；每次启动和网页登录前自动重试。浏览器仍可能在运行时仍判登录失败。
登录失败与目录残留会写入操作日志（只含原因代码和目录随机名，不含本机路径）。

## English

Open P&L is now marked at the side a market close would take: a long at the bid, a short at the offer, no longer the last trade (which can print
on the far side of the spread and overstate what the close realizes). A quote missing that side leaves the position unmarked (awaiting quotes) and
the daily limit falls back to the broker's funds snapshot; the last trade never stands in, and a trade-only quote keeps the last known sides.
The daily-limit breach entry now records each position's side, mark, last trade and average price at the trigger for comparison with the close.

Hardening: relicensed to proprietary (no longer AGPL). The private engine is obfuscated then AES-256-GCM encrypted at build time and decrypted inside the
executable at run time, so public releases carry no readable source; each build's de-obfuscation sourcemaps are the private map, kept for the private repository
release only and never shipped to the public update feed. First run now requires accepting the license agreement (declining exits), recorded with version and
timestamp. The build self-signs the executable by default via PowerShell (self-signed certificates are untrusted, so SmartScreen still warns; set
`BT_SIGN_PFX`/`BT_SIGN_THUMBPRINT` for a real certificate or `BT_SIGN=off` to skip). The executable is compressed — a compressed single-file bundle and a
gzip engine overlay — to about 108MB.

The product is now PropFirmVIP Tradovate everywhere: the executable is `PropFirmVIP-Tradovate.exe`, the engine `PropFirmVIP-Tradovate.Engine.exe`, and the window title,
notifications, browser-login profile folder, diagnostic log file name and the `%LOCALAPPDATA%\\PropFirmVIP-Tradovate` preferences/engine cache follow
(a desktop.json left by the BatchTradovate-named builds is adopted on first start).

A completed browser login no longer fails with "login browser cleanup did not finish": after the browser closes, its debug port confirms
the exit, and the PowerShell process cleanup runs only when that port still answers or files stay busy, so hosts where PowerShell is blocked,
slow or broken can sign in. Removing the temporary folder retries busy files for a bounded time. When the browser has closed but the folder
cannot be removed yet, the login completes and the Connections page lists the folder with administrator removal (UAC confirmation) and
an Open folder action; every start and browser login retries it. A browser that may still be running still fails the login. Failed logins
and leftover folders are journaled with cause codes and the folder's random name, without local paths.

## Commits

- fix: make the release build pass on CI (af317da)
- chore: release v1.6.0 (908ea02)
- docs: update hardening notes for self-signing and compression (60f4455)
- perf: gzip the engine overlay to shrink the executable (902b4aa)
- feat: mark open P&L at the closing side (bid for longs, offer for shorts) (58929c9)
- feat: self-sign the executable by default via PowerShell (f059ba2)
- perf: compress the single-file bundle to shrink the executable (90e5d98)
- docs: add the first-run license screenshot and README note (53ad29c)
- docs: document the hardening (license, engine encryption, map, EULA, signing) (6376064)
- feat: wire optional Authenticode code signing into the build (5648af1)
- feat: gate first run on accepting the proprietary license (6122197)
- feat: emit the private de-obfuscation map beside the executable (39c3050)
- feat: obfuscate engine sources and emit a private sourcemap (9123638)
- feat: encrypt embedded engine assets with AES-256-GCM at build (9ec6cb3)
- fix: tolerate slow loopback refusals when confirming the login browser closed (5be8e80)
- chore: relicense from AGPL-3.0 to proprietary (15e3605)
- fix: keep browser logins when only the profile folder is left (5e012c0)
- chore: stamp log exports with the propfirmvip-tradovate format id (1e281bc)
- chore: point updatesInfo at v1.5.1 (e5c15a3)
- feat: rename the product to PropFirmVIP Tradovate (0870738)

## v1.5.1 (2026-10-22)

## 简体中文

桌面只保留实盘：跟单组、手动面板、Webhook、Pine 策略与定时计划不再提供演练选项，启动均需输入完整组名，建议先用小手数核对；
启动提醒不再要求开启 2FA。应用图标改为企鹅；发布文件改名为 `PropFirmVIP-Tradovate-vX.Y.Z-windows-x64.exe`。

## English

The desktop is live only: copy groups, the manual panel, Webhook, Pine strategies and schedules no longer offer a dry run;
every start needs the exact group name, so verify with a small size first. The startup reminder no longer asks for 2FA.
The application icon is the penguin; release assets are named `PropFirmVIP-Tradovate-vX.Y.Z-windows-x64.exe`.

## Commits

- chore: release v1.5.1 (d907990)
- feat: live-only desktop and the penguin application icon (6f57fae)
- feat: drop the 2FA advice and name release assets PropFirmVIP-Tradovate (77638d5)
- docs: add the PropFirmVIP-styled copy of the user guide (6a4612c)
- docs: propfirmvip customized (f9adaf0)
- docs: add the illustrated Chinese user guide (f92e25e)
- chore: point updatesInfo at v1.5.0 (3f9d766)

## v1.5.0 (2026-10-22)

## 简体中文

品牌声明：每个页面底部新增 `© PropFirmVIP 冒险岛 https://propfirmvip.com/` 版权行（点击打开品牌网址），关于页新增
提供方、技术支持与永久免费声明；两者来自皮肤清单的 `brand.copyright` / `brand.notice`，`data/config.json` 的 `brand` 可覆盖。

海岛皮肤素材调整：侧栏角色改用不拿地图的冒险熊，左上角头像改用用户提供的小熊贴纸；页眉横幅重新绘制，去掉右侧小屋，
右下角改为一只头朝左的手绘小蓝龙，页眉卡片在高窗口下加高到 180 DIP 以显示完整画面。
皮肤清单新增 `header.band`（页眉图的竖向显示区间）、`header.height`（页眉卡片最小高度）、`header.crop`、
`header.mirror`、`header.figureHeight`、`images.headerFigure`（可叠放的小立绘）和 `crops`（按比例裁切某张图片）。

## English

Brand statements: every page ends with the `© PropFirmVIP 冒险岛 https://propfirmvip.com/` copyright line (it opens
the brand address) and the About page shows the provider, support and free-of-charge notice; both come from the
skin manifest's `brand.copyright` / `brand.notice` and `data/config.json` `brand` can override them.

Island skin assets: the sidebar mascot now uses the bear without the map, the top-left avatar is the
user-supplied bear sticker, and the header banner is repainted without the house, with a small hand-drawn blue dragon facing left at the bottom
right; the header card grows to 180 DIP in tall windows so the whole scene shows. The skin manifest gains
`header.band` (vertical band of the header picture), `header.height` (minimum header card height),
`header.crop`, `header.mirror`, `header.figureHeight`, `images.headerFigure` (a small overlaid figure) and
`crops` (relative crops of a picture).

## Commits

- chore: release v1.5.0 (c3ee6ed)
- feat: brand copyright footer on every page and a provider notice on the About page (1f465d4)
- feat: use the supplied bear sticker as the island brand avatar (7e9b18e)
- feat: repaint the island header banner with a hand-drawn dragon (456ab69)
- feat: map-free bear, bear-head avatar and a pixel dragon in the island skin (8480bf8)
- chore: point updatesInfo at v1.0.1 (bbc2e75)

## v1.0.1 (2026-10-22)

## 简体中文

账户页默认折叠「未同步」账户，工具栏的「展开未同步账户 N」按钮可展开或收起；搜索时始终显示全部匹配账户，
点击针对未同步账户的提醒时会自动展开。

## English

The Accounts page folds unsynced accounts by default; the toolbar's "Show N unsynced" button expands or
hides them, a search always shows every match, and opening an alert for an unsynced account expands them.

## Commits

- chore: release v1.0.1 (583fa46)
- feat: fold unsynced accounts on the Accounts page until expanded (24bdce3)
- chore: point updatesInfo at v1.0.0 (4c4ab86)

## v1.0.0 (2026-10-22)

## 简体中文

重新组织桌面界面：侧边栏按配置顺序分为准备（连接、账户）、配置（跟单组）、运行（总览、订单对照、手动下单）和
扩展（Webhook、Pine 策略、活动日志、设置）四组，可手动或在窄窗口自动收起为图标栏。新增「总览」页，汇总五项指标、
正在运行的全部入口（含模式与开始时间）、需要处理的提醒和最近活动；顶部状态条在每个页面显示运行中的跟单组与入口及计划倒计时，
底部状态行分别显示最近操作结果和数据更新时间。跟单组页改为一组一卡（状态、开始时间、跟随统计、操作）加跟随账户、跟单规则、
定时计划、新闻日历、计划记录页签；手动下单、Webhook 与 Pine 策略各自成页，长说明改为「说明」按钮。
新增皮肤：默认使用内置的海岛冒险皮肤（应用名「冒险岛VIP Copy」，左上角带 PropFirmVIP 链接按钮），Material 3 隐藏且不显示明暗切换；
设置页默认不显示皮肤选项；`data/config.json` 写入 `{"showSkinSetting": true}` 后即可在设置内自选 Material 3、海岛冒险或
`data/skins/<名称>/skin.json` 的自定义皮肤，该文件还可选填 `skin`、`theme` 和覆盖品牌文字与链接的 `brand`。
皮肤只改变颜色、圆角、品牌文字、品牌链接和装饰图片。

「关于与更新」页移除了发布页和项目源码按钮，新增 Author / Contributor（github.com/Mind2Money）链接；检查更新改为读取
公开更新仓库 Mind2Money/PropfirmVIPCopyUpdates（子仓库 `updatesInfo/`）的 `updates.json`，发现新版本时显示该版本的下载页，
并可打开更新日志。发布 `vX.Y.Z` 时 CI 会把同一可执行文件同步发布到该公开仓库并更新 `updates.json` 与 `CHANGELOG.md`
（需要 `UPDATES_TOKEN` 密钥），本地可用 `npm run release:updates -- vX.Y.Z <exe>`。

浮盈浮亏改为按行情长连接逐秒计算：账户同步后以同一令牌连接 md-demo/md 行情 WebSocket，只订阅有持仓的合约，
按「净持仓 ×（最新成交价 − 持仓均价）× 核实的每点价值」计算并显示行情时间；桌面最多每秒刷新一次。
新增「今日盈亏」列（当日已实现 + 浮盈浮亏）。券商拒绝行情、行情中断或每点价值未核实时退回 30 秒券商快照并说明原因；
净值和保证金仍只来自券商快照，不用行情推算。

新增按账户设置的日盈利目标 / 日亏损上限：每个报价、余额变化和资金快照都会核对；每个券商交易日首次触及即撤销该账户活动订单、
市价平仓并锁定当日新开仓（减仓、撤单仍允许，锁定的跟随账户复制被拒绝并记录），交易日切换、删除或重新保存规则后解锁。
平仓请求发送前持久化、只凭券商回报确认、未知结果不重发；可选「仅提醒」模式。规则持久保存并绑定账户身份，
实盘自动平仓规则保存时须输入完整账户名。

新增「Pine 策略」交易入口：在程序内运行 Pine Script v4–v6 的 strategy() 脚本，K 线来自 Tradovate 行情流，
历史 K 线预热后按收盘（或脚本声明的逐笔）执行；内置撮合模拟器按 TradingView 规则成交，
每个已选账户把入场单与止盈止损作为券商端括号单镜像下单，之后的出场价变化改括号腿，撤单同步撤销。
模拟与真实成交持续比对：模拟已出场而券商括号单未成交时按市价平仓一次；券商拒绝、结果未知、行情断开、脚本错误、
跟单组变化都停止运行且不重发；手动停止撤销本次挂着的入场单。重启后策略保持停止。
新增 `node scripts/pine-replay.mjs`，用 CSV K 线回放脚本并与 TradingView 交易列表对照。
策略可选「触发后市价进场 + OCO」入场方式：券商端不挂入场单，模拟成交入场时按市价进场并附带止盈止损括号单。

编辑和删除跟单组不再要求重新连接、同步全部账户或核对账户身份：离线账户跳过检查，已同步账户仍有持仓或活动订单时照旧拒绝。
无法核实为空仓的旧映射以“未核实”归档，继续观察券商回报直到平台显示订单结束，不发送任何请求。
删除连接只要求相关跟单组已停止且没有发送中的请求，不再检查账户持仓和订单。

新增券商订单与指令标签诊断日志：保存关联账户收到的 `orderVersion.text` 和 `command.clOrdId`，
即使跟单组已停止也会记录，并在操作日志摘要中直接显示对应标签。

修复 TradingView 组合单仅在入场腿携带 `GTD` 到期时间、止盈和止损腿缺少 `expireTime` 时整张 OSO/OCO
被“订单字段无效（expiry）”拒绝的问题。组合腿现在继承同一策略中已验证的到期时间，不使用 `Day` 替代。

删除连接及其账户、保存账户关系不再弹出额外确认；跟单组停止、账户同步、身份指纹和关系冲突校验保持不变。

删除账户页的“账户关系”页签及其关系声明入口；已保存关系、冲突校验和跨环境组启动规则继续保留。

修复日盈亏上限自动平仓已成交、却显示「平仓未完成 · 请核对」的问题。原因：该合约有挂单（如 OCO 括号单）时，
Tradovate 平仓接口先撤单，回执为不带订单号的空 `{}`，之后才下市价平仓单；此前只认带订单号的回执，20 秒后误判为未确认。
现在这类回执的确认条件是：持仓归零，且该合约全部订单已结束、成交量最终；500 毫秒内证据不全时读一次快照。
已结束但仍有未确认步骤的平仓，会在之后的券商回报（或重启后的同步快照）补齐证据时转为完成，期间不发送任何请求；
结果未知的请求仍不会因空仓而被确认。跟单组「全平」与定时平仓改用同一确认规则，不再停在第一个带挂单的账户。

## English

The desktop workspace was reorganized: the sidebar follows the setup order with Setup (connections,
accounts), Configure (copy groups), Run (overview, order board, manual orders) and Extras (Webhook,
Pine strategies, activity, settings), and collapses to an icon rail by hand or in narrow windows. A new
Overview page gathers the five counters, everything that is running (with mode and start time), open
alerts and the latest activity; live-state chips in the top bar show running groups, entries and
schedule countdowns on every page, and the status line reports the last operation's outcome separately
from the data-update time. The group page shows one group's state, start time, follower counts and
actions with tabs for followers, copy rules, schedules, the news calendar and schedule history; manual
orders, Webhook and Pine strategies are separate pages, and long explanations moved behind a Details
button. Skins: the built-in island skin is the default (app name 冒险岛VIP Copy with a PropFirmVIP link
button at the top left); Material 3 is hidden and the light/dark choice disappears with it. Custom skins
load from `data/skins/<name>/skin.json`. Settings shows no skin option until `data/config.json`
contains `{"showSkinSetting": true}`, after which Material 3, the island skin and custom skins can be
chosen there; the file may also set `skin`, `theme` and `brand` overrides for the text and link. A skin
changes colors, radii, brand text, the brand link and decorative pictures only.

The About page drops the release-page and source-repository buttons and adds an Author / Contributor
link (github.com/Mind2Money). The update check now reads `updates.json` from the public feed repository
Mind2Money/PropfirmVIPCopyUpdates (submodule `updatesInfo/`), shows the download page of a newer
release and links the changelog. Publishing a `vX.Y.Z` tag also publishes the same executable to that
public repository and updates its `updates.json` and `CHANGELOG.md` (CI needs the `UPDATES_TOKEN`
secret; locally `npm run release:updates -- vX.Y.Z <exe>`).

Open P&L now follows the market-data stream every second: after synchronization the app connects the
md-demo/md quote WebSocket with the same token, subscribes only held contracts and computes
`net position × (last trade − position average) × verified value per point`, showing the quote time;
the desktop refreshes at most once a second. A **Day P&L** column (realized today + open P&L) was
added. Refused quotes, a quote outage or an unverified point value fall back to the 30-second broker
snapshot with the reason; equity and margins remain broker snapshots only.

Per-account daily profit target / loss limit: evaluated on every quote, balance change and funds
snapshot. The first breach per broker trading day cancels the account's working orders, closes its
positions at market and locks new exposure for the day (reductions and cancels stay allowed; copies
to a locked follower are refused and logged). The lock clears on the next broker day or when the rule
is removed or saved again. Close requests are persisted before sending, confirmed only by broker
evidence and never resent when unknown; an alert-only mode is available. Rules persist, bind the
account identity, and a Live flatten rule needs the exact account name typed.

Added the Pine strategies trading entry: a Pine Script v4–v6 strategy() runs inside the app on bars
from the Tradovate market-data stream, warms up on history and executes at bar close (or on every
tick when declared). The built-in emulator fills orders by TradingView's rules; each selected
account mirrors entries with their take-profit and stop-loss as broker-side brackets, modifies the
legs when exits change and cancels what the script cancels. Simulated and real fills are compared
continuously: an exit filled only in the emulator closes the position at market once; rejections,
unknown outcomes, feed loss, script errors and group changes stop the run without resending, and a
user stop cancels the run's resting entries. Restart leaves strategies stopped.
`node scripts/pine-replay.mjs` replays a script over CSV bars and compares it with a TradingView trade list.
A strategy can enter at market on trigger instead of resting orders: nothing rests at the broker, and the
emulator's entry fill sends a market OSO with the exit levels as brackets.

Editing and deleting a copy group no longer require reconnecting, synchronizing every account or
reviewing account identities: offline accounts are skipped, while a synchronized account that still
holds a position or working order still refuses the change. Mappings that cannot be verified flat
are archived as unverified and observed until broker evidence shows their orders ended; nothing is
sent for them. Removing a connection needs only its groups stopped and no request in flight;
account positions and orders are no longer checked.

Broker order and command tag diagnostics now record `orderVersion.text` and `command.clOrdId` from
associated accounts even while their copy group is stopped, and show the tag directly in the activity
log summary.

Fixed linked orders being refused as “Invalid order field (expiry)” when TradingView supplied a GTD
expiry on the entry leg but omitted it from the take-profit and stop-loss legs. Linked legs now inherit
the one verified strategy expiry instead of being changed to Day.

Removing a connection and its accounts or saving an account relationship no longer requires an extra
confirmation. Group-stop, synchronization, account-fingerprint and relationship-conflict checks remain.

Removed the Account relationships tab and declaration controls from the Accounts page. Persisted
relationships, conflict checks and cross-environment group rules remain active.

Fixed a filled daily-limit flatten being shown as “Flatten incomplete · review”. When the contract
has working orders (such as an OCO bracket), Tradovate's liquidation cancels them first, answers with
an empty `{}` without an order ID and places the market close afterwards; the close was only accepted
with an order ID and timed out as unconfirmed after 20 seconds. Such an acknowledgement is now
confirmed by a flat position with every order of the contract ended and its fills final, with one
snapshot read when that evidence is still missing after 500 ms. A finished flatten whose only gap is
an unconfirmed acknowledged step completes when later broker events, or the snapshot after a restart,
supply the evidence; nothing is sent, and an unknown outcome is still never confirmed by flatness.
Group flatten and scheduled closes use the same rule and no longer stop at the first account with
working orders.

## Commits

- fix: push the update feed with UPDATES_TOKEN instead of the checkout credentials (cb04757)
- fix: make release publishing idempotent and commit the feed on top of origin/main (3a2ae31)
- chore: release v1.0.0 (558b223)
- chore: point updatesInfo at v0.3.0 (e198d25)
- feat: read updates from the public feed repository and sync releases to it (137377a)
- feat: show the skin setting with a one-key data/config.json (85e649c)
- feat: hide the skin setting and let data/config.json choose the skin (308adf9)
- feat: default to the island skin with data/config.json deployment settings (ea8e6d9)
- perf: skip the exposure scan while a daily-limit step is unconfirmed (326448d)
- fix: confirm liquidations Tradovate acknowledges without an order ID (73107e2)
- feat: reorganize the desktop workspace and add picture skins (a5eed3e)
- feat: add a market entry mode to Pine strategies (bf85886)
- test: replace hanging browser cleanup integration (b5a3da0)
- refactor: remove account relationships page (b141b58)
- feat: remove account action confirmations (9698108)
- fix: inherit GTD expiry across linked orders (15e66e2)
- feat: log broker order and command tags (51356e9)
- test: isolate browser cleanup fixture from Node workers (f1984ad)
- fix: keep Pine mirrors following the script and accept live bars under any chart id (8375ee5)
- fix: keep desktop snapshots alive while a Pine run resolves its contract (bf1eb49)
- feat: edit or delete stopped groups without reconnecting or reviewing identities (c7a4282)
- feat: run Pine strategies in-app and mirror their orders to accounts (cf483bd)
- feat: mark open P&L from the quote stream and add per-account daily limits (2370b01)

## v0.3.0 (2026-10-22)

## 简体中文

重新组织桌面界面：侧边栏按配置顺序分为准备（连接、账户）、配置（跟单组）、运行（总览、订单对照、手动下单）和
扩展（Webhook、Pine 策略、活动日志、设置）四组，可手动或在窄窗口自动收起为图标栏。新增「总览」页，汇总五项指标、
正在运行的全部入口（含模式与开始时间）、需要处理的提醒和最近活动；顶部状态条在每个页面显示运行中的跟单组与入口及计划倒计时，
底部状态行分别显示最近操作结果和数据更新时间。跟单组页改为一组一卡（状态、开始时间、跟随统计、操作）加跟随账户、跟单规则、
定时计划、新闻日历、计划记录页签；手动下单、Webhook 与 Pine 策略各自成页，长说明改为「说明」按钮。
新增皮肤：默认使用内置的海岛冒险皮肤（应用名「冒险岛VIP Copy」，左上角带 PropFirmVIP 链接按钮），Material 3 隐藏且不显示明暗切换；
设置页默认不显示皮肤选项；`data/config.json` 写入 `{"showSkinSetting": true}` 后即可在设置内自选 Material 3、海岛冒险或
`data/skins/<名称>/skin.json` 的自定义皮肤，该文件还可选填 `skin`、`theme` 和覆盖品牌文字与链接的 `brand`。
皮肤只改变颜色、圆角、品牌文字、品牌链接和装饰图片。

「关于与更新」页移除了发布页和项目源码按钮，新增 Author / Contributor（github.com/Mind2Money）链接；检查更新改为读取
公开更新仓库 Mind2Money/PropfirmVIPCopyUpdates（子仓库 `updatesInfo/`）的 `updates.json`，发现新版本时显示该版本的下载页，
并可打开更新日志。发布 `vX.Y.Z` 时 CI 会把同一可执行文件同步发布到该公开仓库并更新 `updates.json` 与 `CHANGELOG.md`
（需要 `UPDATES_TOKEN` 密钥），本地可用 `npm run release:updates -- vX.Y.Z <exe>`。

浮盈浮亏改为按行情长连接逐秒计算：账户同步后以同一令牌连接 md-demo/md 行情 WebSocket，只订阅有持仓的合约，
按「净持仓 ×（最新成交价 − 持仓均价）× 核实的每点价值」计算并显示行情时间；桌面最多每秒刷新一次。
新增「今日盈亏」列（当日已实现 + 浮盈浮亏）。券商拒绝行情、行情中断或每点价值未核实时退回 30 秒券商快照并说明原因；
净值和保证金仍只来自券商快照，不用行情推算。

新增按账户设置的日盈利目标 / 日亏损上限：每个报价、余额变化和资金快照都会核对；每个券商交易日首次触及即撤销该账户活动订单、
市价平仓并锁定当日新开仓（减仓、撤单仍允许，锁定的跟随账户复制被拒绝并记录），交易日切换、删除或重新保存规则后解锁。
平仓请求发送前持久化、只凭券商回报确认、未知结果不重发；可选「仅提醒」模式。规则持久保存并绑定账户身份，
实盘自动平仓规则保存时须输入完整账户名。

新增「Pine 策略」交易入口：在程序内运行 Pine Script v4–v6 的 strategy() 脚本，K 线来自 Tradovate 行情流，
历史 K 线预热后按收盘（或脚本声明的逐笔）执行；内置撮合模拟器按 TradingView 规则成交，
每个已选账户把入场单与止盈止损作为券商端括号单镜像下单，之后的出场价变化改括号腿，撤单同步撤销。
模拟与真实成交持续比对：模拟已出场而券商括号单未成交时按市价平仓一次；券商拒绝、结果未知、行情断开、脚本错误、
跟单组变化都停止运行且不重发；手动停止撤销本次挂着的入场单。重启后策略保持停止。
新增 `node scripts/pine-replay.mjs`，用 CSV K 线回放脚本并与 TradingView 交易列表对照。
策略可选「触发后市价进场 + OCO」入场方式：券商端不挂入场单，模拟成交入场时按市价进场并附带止盈止损括号单。

编辑和删除跟单组不再要求重新连接、同步全部账户或核对账户身份：离线账户跳过检查，已同步账户仍有持仓或活动订单时照旧拒绝。
无法核实为空仓的旧映射以“未核实”归档，继续观察券商回报直到平台显示订单结束，不发送任何请求。
删除连接只要求相关跟单组已停止且没有发送中的请求，不再检查账户持仓和订单。

新增券商订单与指令标签诊断日志：保存关联账户收到的 `orderVersion.text` 和 `command.clOrdId`，
即使跟单组已停止也会记录，并在操作日志摘要中直接显示对应标签。

修复 TradingView 组合单仅在入场腿携带 `GTD` 到期时间、止盈和止损腿缺少 `expireTime` 时整张 OSO/OCO
被“订单字段无效（expiry）”拒绝的问题。组合腿现在继承同一策略中已验证的到期时间，不使用 `Day` 替代。

删除连接及其账户、保存账户关系不再弹出额外确认；跟单组停止、账户同步、身份指纹和关系冲突校验保持不变。

删除账户页的“账户关系”页签及其关系声明入口；已保存关系、冲突校验和跨环境组启动规则继续保留。

修复日盈亏上限自动平仓已成交、却显示「平仓未完成 · 请核对」的问题。原因：该合约有挂单（如 OCO 括号单）时，
Tradovate 平仓接口先撤单，回执为不带订单号的空 `{}`，之后才下市价平仓单；此前只认带订单号的回执，20 秒后误判为未确认。
现在这类回执的确认条件是：持仓归零，且该合约全部订单已结束、成交量最终；500 毫秒内证据不全时读一次快照。
已结束但仍有未确认步骤的平仓，会在之后的券商回报（或重启后的同步快照）补齐证据时转为完成，期间不发送任何请求；
结果未知的请求仍不会因空仓而被确认。跟单组「全平」与定时平仓改用同一确认规则，不再停在第一个带挂单的账户。

## English

The desktop workspace was reorganized: the sidebar follows the setup order with Setup (connections,
accounts), Configure (copy groups), Run (overview, order board, manual orders) and Extras (Webhook,
Pine strategies, activity, settings), and collapses to an icon rail by hand or in narrow windows. A new
Overview page gathers the five counters, everything that is running (with mode and start time), open
alerts and the latest activity; live-state chips in the top bar show running groups, entries and
schedule countdowns on every page, and the status line reports the last operation's outcome separately
from the data-update time. The group page shows one group's state, start time, follower counts and
actions with tabs for followers, copy rules, schedules, the news calendar and schedule history; manual
orders, Webhook and Pine strategies are separate pages, and long explanations moved behind a Details
button. Skins: the built-in island skin is the default (app name 冒险岛VIP Copy with a PropFirmVIP link
button at the top left); Material 3 is hidden and the light/dark choice disappears with it. Custom skins
load from `data/skins/<name>/skin.json`. Settings shows no skin option until `data/config.json`
contains `{"showSkinSetting": true}`, after which Material 3, the island skin and custom skins can be
chosen there; the file may also set `skin`, `theme` and `brand` overrides for the text and link. A skin
changes colors, radii, brand text, the brand link and decorative pictures only.

The About page drops the release-page and source-repository buttons and adds an Author / Contributor
link (github.com/Mind2Money). The update check now reads `updates.json` from the public feed repository
Mind2Money/PropfirmVIPCopyUpdates (submodule `updatesInfo/`), shows the download page of a newer
release and links the changelog. Publishing a `vX.Y.Z` tag also publishes the same executable to that
public repository and updates its `updates.json` and `CHANGELOG.md` (CI needs the `UPDATES_TOKEN`
secret; locally `npm run release:updates -- vX.Y.Z <exe>`).

Open P&L now follows the market-data stream every second: after synchronization the app connects the
md-demo/md quote WebSocket with the same token, subscribes only held contracts and computes
`net position × (last trade − position average) × verified value per point`, showing the quote time;
the desktop refreshes at most once a second. A **Day P&L** column (realized today + open P&L) was
added. Refused quotes, a quote outage or an unverified point value fall back to the 30-second broker
snapshot with the reason; equity and margins remain broker snapshots only.

Per-account daily profit target / loss limit: evaluated on every quote, balance change and funds
snapshot. The first breach per broker trading day cancels the account's working orders, closes its
positions at market and locks new exposure for the day (reductions and cancels stay allowed; copies
to a locked follower are refused and logged). The lock clears on the next broker day or when the rule
is removed or saved again. Close requests are persisted before sending, confirmed only by broker
evidence and never resent when unknown; an alert-only mode is available. Rules persist, bind the
account identity, and a Live flatten rule needs the exact account name typed.

Added the Pine strategies trading entry: a Pine Script v4–v6 strategy() runs inside the app on bars
from the Tradovate market-data stream, warms up on history and executes at bar close (or on every
tick when declared). The built-in emulator fills orders by TradingView's rules; each selected
account mirrors entries with their take-profit and stop-loss as broker-side brackets, modifies the
legs when exits change and cancels what the script cancels. Simulated and real fills are compared
continuously: an exit filled only in the emulator closes the position at market once; rejections,
unknown outcomes, feed loss, script errors and group changes stop the run without resending, and a
user stop cancels the run's resting entries. Restart leaves strategies stopped.
`node scripts/pine-replay.mjs` replays a script over CSV bars and compares it with a TradingView trade list.
A strategy can enter at market on trigger instead of resting orders: nothing rests at the broker, and the
emulator's entry fill sends a market OSO with the exit levels as brackets.

Editing and deleting a copy group no longer require reconnecting, synchronizing every account or
reviewing account identities: offline accounts are skipped, while a synchronized account that still
holds a position or working order still refuses the change. Mappings that cannot be verified flat
are archived as unverified and observed until broker evidence shows their orders ended; nothing is
sent for them. Removing a connection needs only its groups stopped and no request in flight;
account positions and orders are no longer checked.

Broker order and command tag diagnostics now record `orderVersion.text` and `command.clOrdId` from
associated accounts even while their copy group is stopped, and show the tag directly in the activity
log summary.

Fixed linked orders being refused as “Invalid order field (expiry)” when TradingView supplied a GTD
expiry on the entry leg but omitted it from the take-profit and stop-loss legs. Linked legs now inherit
the one verified strategy expiry instead of being changed to Day.

Removing a connection and its accounts or saving an account relationship no longer requires an extra
confirmation. Group-stop, synchronization, account-fingerprint and relationship-conflict checks remain.

Removed the Account relationships tab and declaration controls from the Accounts page. Persisted
relationships, conflict checks and cross-environment group rules remain active.

Fixed a filled daily-limit flatten being shown as “Flatten incomplete · review”. When the contract
has working orders (such as an OCO bracket), Tradovate's liquidation cancels them first, answers with
an empty `{}` without an order ID and places the market close afterwards; the close was only accepted
with an order ID and timed out as unconfirmed after 20 seconds. Such an acknowledgement is now
confirmed by a flat position with every order of the contract ended and its fills final, with one
snapshot read when that evidence is still missing after 500 ms. A finished flatten whose only gap is
an unconfirmed acknowledged step completes when later broker events, or the snapshot after a restart,
supply the evidence; nothing is sent, and an unknown outcome is still never confirmed by flatness.
Group flatten and scheduled closes use the same rule and no longer stop at the first account with
working orders.
