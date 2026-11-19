# 🎛️ H3 全参考模式实战：Ref2VA 六段式写法

作者：像素幻想Lab｜9月5日修改。

> 资料状态：方法参考｜复核：2026-09-05。依据 MiniMax H3 官方提示词指南整理。六段全部用英文书写，台词和画面文字保留原语言。

整理说明：本页转录第 4 张截图中可辨认的内容，滚动拼接造成的重复片段合并一次。`[原图右侧截断]`、`[原图遮挡]` 是整理者标记，不属于原文，也不是可执行提示词。[查看原始截图](../../images/h3/04-h3-reference-mode.png)。截图中的方法说明仅作为资料保留，未另行核验其官方性。

Ref2VA 是全参考模式：人物图、场景图、参考视频、参考音频一起输入。这时提示词不再是一条三字段结构，而是六段固定结构，字段名和顺序都不能改。

## 六段总览

| 字段 | 作用 |
| --- | --- |
| `subject_definitions` | 给每条参考内容编号，说明它的角色和要跟随的特征 |
| `summary` | 一段话概括任务类型、目标视频与主要参考关系 |
| `retention_analysis` | 逐条说明参考内容被完整保留、部分保留、转移还是弱引用 |
| `detailed_description` | 按播放顺序逐镜头写画面、动作、镜头与声音（主体） |
| `overall_soundscape` | 全片环境声与动作声 |
| [原图遮挡：最后一行无法完整辨认] | [原图遮挡] |

整理注：上表第六行在原图被表头样的灰色条遮挡；本页后文“声音两段”及案例代码明确出现 `non_diegetic_music`，对应画外配乐。此处是跨段对应说明，不是对被遮挡原文的恢复。

## subject_definitions：四类标签

| 标签 | 用法 |
| --- | --- |
| `<Subject N>` | 可复用的可见内容：人、动物、物体、场景、服装、界面、风格、动作 |
| `<Picture N>` | 作为具体帧锚的参考图：首帧、关键帧、尾帧、分镜规划 |
| `<Video N>` | 整片级关系：被剪辑的原片、续拍起点、剪辑节奏来源 |
| `<Audio N>` | 被复制或被参考的音频信号 |

两个易错点：如果某张图只用来定义人物或场景外观，不作为任何帧锚，就不单列 Picture 条目，在对应 Subject 定义里注明来源即可；音频对应目标说话人时，复用该说话人的全局编号，写法如 `<Audio 1> is the voice-timbre reference for <Subject 1> (S1).`

## summary：任务类型前缀

summary 以方括号任务类型开头，按参考内容实际扮演的角色组合，多种关系用 + 连接：

| 类型 | 何时使用 |
| --- | --- |
| [原图遮挡：第一行类型无法完整辨认] | [原图遮挡：仅剩上部字形，不能可靠转录] |
| `reference generation` | 图 / 视频 / 音频只提供生成参考，不作帧锚也不被剪辑 |
| `video editing` | 原片被直接剪辑修改 |
| `video continuation` | 新内容从原片继续、延伸、衔接 |
| `audio reuse` | 音频信号被全部或部分直接复用 |
| `audio reference` | 只参考音色、节奏、音乐风格、台词内容 |

例如从原片续拍、同时用一张图做尾帧，写成 `[video continuation + keyframe completion]`。剪辑类任务在开头用固定句式：`The target video is an edited version of <Video 1>.`

整理注：示例明确出现 `keyframe completion`，但上表第一行被遮挡，未据此补写表格原文。

## retention_analysis：保留关系标记

| 画面标记 | 含义 | 音频标记 | 含义 |
| --- | --- | --- | --- |
| `fully_preserved` | 定义的角色完整保留 | `fully_copy` | 整段音频 1:1 成为最终音轨 |
| `partially_preserved` | 部分特征被改变或只保留一部分 | `partially_copy` | 只复制部分，或复制后有增删替换 |
| `attribute_transfer` | 特征转移到另一个明确目标 | `reference` | 只参考音色、节奏、风格，不复制信号 |
| `weak_reference` | 只保留风格、类别、氛围的大致相似 | `weak_reference` | 只保留类别或氛围的大致相似 |

条目格式固定为：`<Subject 1> (appears in [Shot 1], [Shot 3]): fully_preserved - 具体说明哪些特征被保留`。注意说话人编号 (Sx) 不出现在这一段。

## detailed_description：主体描述

先用一两句英文声明整体风格，再进入 [Shot 1]；后续镜头用 At MM:SS.mmm 标注切镜时间。生成类任务通常写 350–500 个英文单词，对白密集时优先保证台词时间线完整，不必凑字数。引用标签在首次出现处插入并说明其特征，之后沿用、不再重复定义。参考对象说话时保留双重身份，如 `<Subject 2> (S1) turns toward the woman and says, <d>[English] ...</d>`。

一个高频细节：当台词只是被直接复用的 BGM 或完整音轨的一部分、没有真实的发声者时，用 `<Audio N>` 作为声源，不要虚构 (Sx)；只有具体人物、旁白真的在说话时才分配说话人编号。

## 声音两段与参考标签的联动

当同一段参考音频同时包含环境层和配乐层时，按 audible 层分开声明关系：

```text
overall_soundscape: The copied ambience layer from <Audio 1> continues throughout the targe[原图右侧截断]
non_diegetic_music: <Audio 2> is directly reused as the complete audience-only score.
```

整理注：第一行行末单词在原图截断，以上保留可辨认的 `targe` 片段。第二行原文写的是 `<Audio 2>`，保留其原样。

## 完整案例骨架（咖啡店与萨摩耶）

四张参考图 + 两段参考视频 + 一段音色参考，任务类型 `[reference generation + audio reference]`。完整六段较长，这里保留骨架，重点看标注方式：

### 六段骨架示例

```text
subject_definitions:
<Subject 1> is the coffee-shop environment in <Picture 1>, featuring an exposed brick wall[原图右侧截断]
<Subject 2> is the fluffy white Samoyed in <Picture 2>, <Picture 3>, and <Picture 4>, with[原图右侧截断]
<Subject 3> is the young blonde woman in <Video 1>, with long blonde hair and a light-pink[原图右侧截断]
<Subject 4> is the young man in <Video 2>, with short wavy brown hair and a dark-grey hood-[原图右侧截断]
<Audio 1> is the voice-timbre reference for <Subject 3> (S1).

summary:
[reference generation + audio reference] The target video shows <Subject 3> eating a cookie[原图右侧截断]

retention_analysis:
<Subject 1> (appears in [Shot 1], [Shot 2], [Shot 3]): fully_preserved - the brick wall, so[原图右侧截断]
<Audio 1>: reference - its vocal timbre guides the dialogue delivery of <Subject 3> without[原图右侧截断]

detailed_description:
The target video uses a realistic multi-camera sitcom style with warm indoor lighting.
[Shot 1] A medium shot establishes <Subject 1> ... [Shot 2] At 00:03.000, the shot cuts to[原图右侧截断]

overall_soundscape: Soft indoor coffee-shop room tone continues throughout the scene.
non_diegetic_music: N/A
```

整理注：上述是截图本就展示的骨架。代码中间的 `...` 是原图自带省略号；行末的 `[原图右侧截断]` 是整理者添加的缺失标记。边缘单词若仅有部分字形，仅转录可辨部分；不将被隐藏的示例补写为完整提示词。

## 写完自检

- [ ] 每个标签都在 subject_definitions 里有定义，全文含义一致
- [ ] summary 开头有任务类型前缀，且没有引入新标签
- [ ] retention_analysis 覆盖每条标签，标记与定义的角色一致
- [ ] 台词全部在 detailed_description 的 `<d>` 标签内，声音两段没有重复台词
- [ ] 切镜时间递增且都在视频时长内

## 原图完整性记录

- 保留了标题、作者、修改日期、资料状态、全部可读正文、可读表格条目、代码可见片段及 5 项自检。
- 合并了 retention_analysis 表格中重复的 `attribute_transfer` 行、detailed_description 末尾与声音联动说明的重复片段，以及六段骨架开头的重复片段。
- 六段总览最后一行、summary 类型表第一行被遮挡，已标明；多处代码右侧超出截图宽度，已逐行标明。
- 原图中的重复水印 `ikigai`、头像与页面悬浮控件作为视觉元素保留在原图中，未混入正文。

---

[上一篇：H3 基础模式实战](03-basic-modes.md) · [返回目录](../../README.md)
