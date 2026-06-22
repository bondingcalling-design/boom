# 一触连爆 (Boom · One-Tap Chain Reaction) — Handoff 文档

> 最后更新：2026-06-18 ｜ 版本：v10 ｜ 单人开发者：胡佳仪

## 1. 这是什么
一个原创**休闲小游戏**：满屏漂浮的彩色球，每次点击炸开一个冲击波，碰到的球跟着炸 → 连环引爆。
两种模式：
- **闯关模式 (endless)**：每关只能点一下，炸够目标数过关，关卡无限、越来越难。
- **清屏模式 (clear)**：给有限点击次数，要清空全场，用得越少 ⭐ 越多；含障碍物。

目标：先把"好玩 + 留存"做扎实，再移植到微信/抖音小游戏上线变现。

## 2. 在哪里 / 地址
| 项 | 位置 |
|---|---|
| **开发源文件** | `C:\Users\胡佳仪\merge-game\chain.html`（单 HTML 文件，含全部 HTML/CSS/JS） |
| **线上地址** | https://bondingcalling-design.github.io/boom/ |
| **部署仓库** | GitHub `bondingcalling-design/boom`（GitHub Pages，根目录 `index.html`） |
| **本地部署副本** | `C:\Users\胡佳仪\boom\index.html`（= chain.html 的拷贝） |
| 搁置的原型 | 同目录 `grow.html`(养肥再跑)、`hole.html`(偏食黑洞) |

## 3. 怎么更新上线（重要）
本机 **没有装 gh**；GitHub 凭证存在 Windows 凭证管理器里（账号 `bondingcalling-design`，token 有 repo 权限）。更新流程：
```bash
# 1) 改 C:\Users\胡佳仪\merge-game\chain.html
# 2) 同步到部署仓库并推送：
cd "/c/Users/胡佳仪/boom"
cp "/c/Users/胡佳仪/merge-game/chain.html" index.html
export TOKEN=$(printf "protocol=https\nhost=github.com\n\n" | git credential fill 2>/dev/null | sed -n 's/^password=//p')
git add -A
git -c user.name="Bondingcalling" -c user.email="3080295096@qq.com" commit -m "说明"
git -c credential.helper='!f() { echo username=bondingcalling-design; echo password=$TOKEN; }; f' push origin main
# 3) GitHub Pages 约 1 分钟后自动更新，网址不变
```
> 注意：github.io 国内访问偏慢、微信内点链接常被拦（需"在浏览器打开"）。真分发归宿是小游戏平台。

## 4. 代码结构（chain.html 单文件）
DOM 实现（不是 Canvas）。核心：
- **状态**：`mode`("endless"/"clear")、`level`、`score`、`best`、`coins`、`bubbles[]`、`rings[]`、`obstacles[]`、`popped`、`goal`、`chainN`、`tapsLeft`、`armed`(装备的道具)。
- **主循环** `loop()`(requestAnimationFrame)：移动球(撞墙/撞障碍反弹) → 更新冲击波环 `rings` 并判定触发 `popBubble` → 连锁炸完(`chaining && rings.length===0`)调 `postChain()` 结算。
- **点击** `tap()`：受 mode 规则限制（闯关一次、清屏扣点击数）；若已装备道具则释放道具效果。
- **特殊球** 在 `popBubble()` 内分支：`cross`(✛炸排+列)、`big`(✸巨爆)、`rainbow`(🌈同色全炸)、`plus`(➕仅清屏+1点击)。
- **货币/道具**：`coins`(localStorage `cb_coins`)；`armItem()` 花币装备 → 下一次点击释放；`COST={cross:30}`(彩虹已删除)。当前唯一可买道具=✛十字炸。
- **难度核心杠杆**：冲击波半径 `addRing` 默认 **46**(原60，缩小=连锁更难蔓延=整体变难)；big球环96、cross环58。游戏太简单时优先调这个。
- **续命系统(金币主要用途)**：差一点过不了关时 `tryRescueOrEnd()`→`offerRescue()`，花币在原盘面再炸一击(`doRescue()`)，价格 `rescueCost()=30*2^continues`(每续翻倍)。这是金币的核心价值，也是未来"看广告免费续一次"的变现接入点。
- **音频**：纯 WebAudio 合成，无音频文件。`playPop`(连锁音调递增)、BGM(`startBGM`，和弦琶音循环 C–Am–F–G，非随机)、`fanfare`(过关和弦)。音量：playPop 0.34 / fanfare 0.42 / BGM note 0.14。
- **过关庆祝** `celebrate()`：彩带 `confetti()` + `fanfare()` + 金光 `winflash` + 横幅。

### 难度调参在哪（最常改的地方）
- `levelConf(lv)`：闯关。`count=min(38,14+lv*2)`；过关比例 `min(0.85,0.18+lv*0.055)`；`speed=min(2.4,0.9+lv*0.06)`。
- `clearConf(lv)`：清屏。`count=min(46,18+lv*3)`；`budget=5`(点击数)；`obs=min(6,lv-1)`(障碍数)。
- 球大小：`spawnBubble` 内 `baseR=max(10,16-level*0.4)`（随关卡变小）。
- 特殊球出现率：`spawnBubble` 内按 mode 的概率分支；**闯关前 3 关不出特殊球**(`early`)。

### localStorage 键
`cb_best_e`/`cb_best_c`(各模式最高分)、`cb_hist_endless`/`cb_hist_clear`(历史)、`cb_coins`(金币)、`cb_sfx`/`cb_bgm`/`cb_vib`(开关)。

## 5. 版本历史
- v1 核心玩法；v2 震动/BGM/历史/难度重做(球封顶+比例&球速递增)；v3 特殊球✛✸；v3.1 删冗余按钮+平衡；v3.2 点击即振+调难；v4 清屏模式+模式菜单+多地图+障碍；v4.1 前几关降难；v4.2 球随关卡变小；v4.3 BGM音量↑+前3关无特殊球；**v5 货币系统+道具(🌈/✛)+新特殊球(🌈同色全炸/➕加点击)**；**v6 过关反馈(彩带+音效+金光)**；v6.1/6.2 彩虹削弱+涨价(全场→半径75内最多3个，50→100币)；**v7 续命系统**(差一点花币再炸一次，价格翻倍递增)；v7.1 背景音乐重做(和弦琶音循环)+音量↑；**v8 删除彩虹 + 整体调难(冲击波半径 60→46)**；v8.2 音量大幅提升+主控压缩器；**v9 官方皮肤系统**(攒币解锁+切换，`applySkin()` 渲染普通球，localStorage `cb_skins`/`cb_skin`)；**v9.1 沙雕表情包皮肤(emoji) + 自定义上传图片皮肤**(canvas 缩128存 `cb_customimg`，兼"上传照片消除朋友"裂变)。⚠️ 版权：奶龙/HelloKitty/网红猫/表情包等 IP 不做(侵权)，只用 emoji 或让用户自传。
- **v10 每日登录奖励**(连续签到送币、断签重置，`checkDailyLogin()`，localStorage `cb_lastlogin`/`cb_streak`)。当前阶段=留存验证，下一步每日挑战。

## 6. 开放问题 / 已知坑
- **网页振动**：iOS Safari 完全不支持；安卓部分可用。**背景音乐**网页端偶发问题。→ 都留到**微信小游戏版**用原生 `wx.vibrateShort` + 原生音频解决。
- **微信小游戏是 Canvas 环境，没有 DOM**；当前是 DOM 实现，移植 = 改写成 Canvas（一笔实打实工作量）。建议网页验证好玩+留存后再一次性移植。

## 7. 后续路线图（按价值）
1. （内容已较丰富）先收朋友反馈，验证好玩度/留存。
2. **皮肤系统**：多种小球皮肤/主题 + 用户自定义皮肤。
3. **🔥 裂变玩法(用户重点提议)**：上传朋友照片当小球(消除="消除朋友")；上传音频，球被消除时播放。社交传播强。
4. **微信小游戏移植**：Canvas 改写 + 接**免费好友排行榜** + 原生分享/振动/音乐。需用户本人注册小游戏账号(实名)。
5. **变现**：货币系统已埋好；未来"金币不够→看广告得币/道具"= 激励视频变现接入点。**用户明确：现在先不接广告**。

## 8. 关键决策与原因
- **为什么是这个玩法**：用户和朋友爱玩休闲游戏(对倾听等自有产品无使用欲，见创始人-用户匹配)；原创"连锁"避免"这不就是XX"的换皮感。
- **为什么先网页 GitHub Pages**：最快验证好玩，不需要平台账号/审核。
- **为什么个人主体只能靠广告变现**：个人微信小游戏无内购(IAP 需版号/企业资质)，只能流量主激励视频。
- **协作约定**：不再频繁换玩法，盯住「一触连爆」迭代到好玩+留存够，再上平台。每次小改即时部署、让用户在手机上真玩、按真实反馈调。

## 9. 待办计划：UI 大改（温馨可爱 + 全面去 emoji）— 用户已确认，择期执行
**动机**：用户觉得当前满屏 emoji"一股 AI 味"、整体偏丑，要求换成**温馨可爱**风、UI 完全重做、布局自行调整。（评估过现有 skill，无匹配"重绘现有游戏 UI"的，直接用设计判断手写。）

**铁律**：游戏逻辑全部不动，**只改视觉**——即 CSS + 注入视觉的几个点（spawnBubble 特殊球/障碍、applySkin、SKINS、COLORS、THEMES、banner/celebrate/signin/HUD/tip/开关 的文案）。**所有元素 id 保持不变**，避免 JS 失效。预计做法=整份 chain.html 重写（保留全部 JS 逻辑）。

**设计系统（温馨可爱）**：
- 背景：奶油暖色渐变（如 `#fff1e2→#ffe0c9`）；文字用暖棕 `#6b5444`（别用纯白，太冷）。
- 卡片/面板：奶白 `#fffaf4` + 柔和阴影 `rgba(214,150,100,.18)`，大圆角(≥18px)。
- 按钮：胖乎乎糖果风——实色珊瑚 `#ff9e7a`，底部 3D 阴影 `box-shadow:0 4px 0 #e87e5a`，按下下沉；ghost 款奶油色。
- 小球配色：马卡龙暖系 `["#ff9aa2","#ffc48a","#ffe08a","#a8e6a1","#8fd3e8","#a9b8ff","#e3a9ff"]`，带白色高光(radial-gradient)+柔和边/影，在暖色场地上能凸显。
- THEMES 改成暖色渐变（奶油/粉/薄荷/薰衣草/天蓝），不要原来的深蓝科技风。

**去 emoji 替换清单（逐个落地）**：
- 普通球：CSS 糖霜高光渐变（无 emoji）。
- 特殊球：✛十字→CSS 白色十字标记(子元素双向线性渐变成"+")；✸巨爆→更大+金色脉冲发光(无符号)；➕加点球→薄荷绿球+白色"+1"文字。
- 障碍 🪨 → CSS 灰色圆角方块（顶部渐变做立体感）。
- 皮肤：删掉 emoji 款(动物/水果/沙雕表情)，换成**质感款**：糖霜(classic)/果冻(jelly)/马卡龙(macaron)/柔光(glow)/自定义上传(custom，保留)。applySkin 改为纯 CSS 质感。
- UI 文案/图标：🪙→CSS 金币小圆点或"金"字；🔊🎵📳→文字胶囊"音效/音乐/震动"(关时变暗)；🎨→"外观皮肤"；过关 🎉/💥→纯文字"过关啦！"；⭐→用 ★(符号字形，非彩色 emoji)；📅→"每日签到"；结束页 😅 去掉；tip 里 emoji 去掉换成普通说明。

**布局**：顶栏(标题+分数/最高)、信息条(关卡/目标/剩余)、道具条(金币+十字炸)、底部开关，统一成奶油卡片风、圆角、留白舒服。自行微调间距与层级。

**验收**：通篇无彩色 emoji；整体奶油暖色、圆润可爱；功能与改前完全一致(双模式/特殊球/续命/货币/皮肤/每日签到照常)。
