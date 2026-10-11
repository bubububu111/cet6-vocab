# AGENTS.md

## 项目简介

六级单词默写系统 —— 纯前端单文件网页应用，用于 CET-6 单词背诵、默写、同义词归类与学习统计。无需服务器，浏览器双击即可运行，所有数据保存在浏览器 localStorage。

## 关键文件

- `六级单词默写.html` —— 唯一交付物。所有功能、样式、内置词库数据都在此文件内，**必须保持自包含**：不得引入外部 JS/CSS/字体/图片/CDN 依赖，否则在 file:// 协议下打开会失效。

## 运行方式

- 本地：直接双击 `六级单词默写.html`，或在浏览器中打开。无需构建、无需安装依赖。注意：数据存于 localStorage，不同浏览器 / 无痕模式之间互不相通；更换浏览器前请先「单词本 → 导出备份」。
- 在线：已部署至 GitHub Pages —— https://bubububu111.github.io/cet6-vocab/ （`index.html` 为跳转页，自动跳到 `六级单词默写.html`；线上与本地数据不互通，各设备独立存储）。
- 源码仓库：https://github.com/bubububu111/cet6-vocab （main 分支，Pages 部署源为该分支根目录；提交到 main 后需等待 1–2 分钟构建生效）。

## 技术栈与约束

- 原生 HTML + CSS + JavaScript（ES6），无框架
- 持久化：localStorage
- 发音：Web Speech API（speechSynthesis），离线时无声音属浏览器正常现象
- 交互全部为原生 DOM 操作 + `onclick` 内联事件，无事件框架

## 数据模型

每个词条对象：

```
{ en, cn, pos, ph, ex, wrong, level, streak }
```

- `en` 英文单词（唯一标识，导入去重依据）
- `cn` 中文释义（可含多个义项，用 `；` 分隔，同义词归类依赖它）
- `pos` 词性标注（可空）：`n.` `v.` `adj.` `adv.` `prep.` `conj.` `pron.` `num.` `phr.`，兼类用 `v./n.` 等
- `ph` 音标（可空）
- `ex` 例句（可空）
- `wrong` 是否为错题（答错置 true，答对置 false）
- `level` 掌握度：0 未掌握 / 1 学习中 / 2 已掌握
- `streak` 连续答对次数

**掌握度规则**：连续答对 3 次 → level 升为 2（已掌握）；答错 → streak 清零、level 降为 0。

**词性显示规则**：默写卡片「看英文」时显示 音标·词性，「看中文」时只显示词性（避免音标剧透英文拼写）；单词本、选择题（中文选项）、同义词组中均展示词性。

导入 / 导出格式：`英文|释义|音标|例句|词性`（一行一条，| 分隔，词性/音标/例句可省略；旧版 4 字段备份文件依然兼容，词性自动为空）。

## localStorage 键（新增键必须在此登记）

| 键 | 内容 |
| --- | --- |
| `cet6words` | 词库主数据（数组） |
| `cet6stats` | `[{d:'YYYY-MM-DD', total, right, wrong}]` 每日练习统计 |
| `cet6checkin` | `['YYYY-MM-DD', ...]` 打卡日期集合 |
| `cet6wrong` | 历史遗留键，不再读写，勿删除（兼容旧版本数据） |

## 核心函数速查

- `loadWords()` 首次访问自动载入 `PRESET_WORDS`（内置 453 词库，A–Z 高频六级词，全部含词性/音标/例句）
- `parseLines(text)` 解析导入文本并去重（按 en），支持 2~5 字段（英文|释义|音标|例句|词性）；`importWords()` / `importFile()` / `loadPreset()` 调用它
- `nextWord()` 展示当前词并进入下一个；`checkAnswer()` 判题，更新掌握度 + 统计；`quizPick(i)` 选择题判题
- `normalize(s)` 判题比对：去空格标点、忽略大小写
- `markRight(w)` / `markWrong(w)` 掌握度状态机（连对 3 次规则）
- `keysOf()` / `related()` / `buildClusters()` / `groupLabel()` 同义词归类：释义按标点拆关键词，关键词相同或互相包含（长度 ≥2）即视为同义，用并查集聚类
- `recordStat(ok)` 记录每日统计与打卡；`streakDays()` 连续打卡天数；`renderHeat()` 30 天热力图（绿≥80% 蓝≥50% 橙≥20% 红<20%）
- `speak(en)` Web Speech 朗读；`speakIdx(i)` 按词库索引朗读
- `renderBook()` 单词本（搜索 + 掌握度徽章）；`renderClusters()` 词义分类视图
- `esc(s)` 所有插入 innerHTML 的词条字段必须经过它转义

## 修改约定

1. 新增功能必须保持：单文件、纯前端、零外部依赖。
2. 词条新增字段时，须在 `loadWords()` 的 map 中补默认值，保证旧 localStorage 数据兼容。
3. 改动判题逻辑时，同步更新「连对 3 次 → 已掌握」规则说明（含 UI 提示 title）。
4. 新增 localStorage 键时更新上方表格。
5. 内置词库 `PRESET_WORDS` 格式固定为 `{en, cn, pos, ph, ex}`，释义中的共同关键词越统一，词义分类效果越好；词性标注使用标准缩写（n. / v. / adj. / adv. / prep. / conj. / pron. / num. / phr.）。
6. 不要在 HTML 中插入第三方统计 / CDN 脚本。
