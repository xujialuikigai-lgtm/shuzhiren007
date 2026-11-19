# 🧱 H3 提示词结构解析：三大核心字段与镜头语法

作者：像素幻想Lab · 原图标注：9月5日修改。

> 资料状态：方法参考 | 复核：2026-09-05。本页讲基础模式（T2VA / I2VA / FL2VA / L2VA）的字段语法；全参考模式的六段式见本专区第 4 篇。

整理说明：本页转录[第 2 张原图](../../images/h3/02-h3-prompt-structure.png)中全部可辨识的独特内容，合并滚动拼接造成的重复段落。`[原图截断]` 是整理标记，不是提示词正文；遮挡和缺字均另行说明，未猜补原图之外的内容。正文是原图资料的转录，不代表另行核验了其中的模型规则。

一条 H3 基础模式提示词 = （可选的参考图对齐指令行）+ 空行 + 三个核心字段。三个字段必须原样使用英文小写、下划线写法，顺序固定：

整体骨架：

```text
对齐指令行（只有 I2VA / FL2VA / L2VA 才有，见第 3 篇实战）

integrated_multimodal_description: [Shot 1] ...

overall_soundscape: ...

non_diegetic_music: ...
```

## integrated_multimodal_description：沿时间线写的主描述

这是提示词的主体。每个细节都要对应“看得见或听得见”的东西：视觉风格、开场构图、人物外观与位置、场景与关键道具、动作与反应、镜头切换、台词和同步的声音。**[Shot 1]** 开头先用一句声明整体风格和开场构图，常见风格词如 live-action、cinematic、2D-animated、3D CG、claymation、watercolor、vintage film。

从第二个镜头开始，用递增的切镜时间开头：

```text
[Shot 2] At 00:03.500, the camera cuts to...
```

切镜时间必须严格递增，且全部落在视频时长以内。普通切换用 the camera cuts to、the shot transitions to 这类说法；cross-dissolve、fade、wipe 只在明确需要时使用。每次切镜都应该带来新信息（主体、空间、状态、视角或时间的变化）；如果只是距离或角度的微调，优先写运镜而不是切镜。

## 运镜：类型 + 幅度 + 速度

| 维度 | 常用写法 | 说明 |
| --- | --- | --- |
| 类型 | Zoom In / Zoom Out | 变焦，机位不动 |
| 类型 | Push In / Pull Out | 机位向前 / 向后移动 |
| 类型 | Pan Left / Pan Right | 机位原地水平摇镜 |
| 类型 | Truck Left / Truck Right | 机位水平平移 |
| 类型 | Tilt Up / Tilt Down | 机位原地俯仰 |
| 类型 | Pedestal Up / Pedestal Down〔原图表头遮挡了字形下部〕 | 整机升降〔原图表头遮挡了字形下部〕 |
| 类型 | Arc Shot / Tracking Shot〔原图表头遮挡了字形上部〕 | 环绕主体 / 跟随运动主体〔原图表头遮挡了字形上部〕 |
| 类型 | Static Shot / POV | 固定机位 / 主观视角 |
| 类型 | Shake Slightly / Shake Strongly | 轻微 / 剧烈晃动 |
| 类型 | Roll Clockwise / Counterclockwise | 绕镜头轴滚转 |
| 幅度 | with small / large amplitude | 中等幅度默认不写 |
| 速度 | at slow / fast speed | 常速默认不写 |

运镜要写成镜头内的自然英语句子，而不是堆在句尾的标签：

```text
The camera pushes in with small amplitude at slow speed toward the folded letter in her ha[原图截断]
The camera pans right with large amplitude at fast speed, revealing the open doorway.
```

第一句末尾在 `ha` 后仍有不完整字母笔画；右侧内容被代码框裁掉，不能确定完整结尾。

## 说话人、台词与画外音

发声的角色用稳定编号 (S1)、(S2)，多人齐声合并写 (S1,S2)；编号跨镜头保持不变，从不发声的角色不编号。说话人首次出现时给出足够的辨识信息：年龄、性别、音高、音色、语速、口音。说话人描述、编号、动作都放在标签外面，标签内只放语言标记和台词原文，逐字保留、不做翻译：

```text
The young woman with a quiet, breathy voice (S1) says: <d>[English] I get off at the next s[原图截断]
```

此句最后的 `s` 只显示部分字形；原图右侧被裁切，台词后文和闭合标签不可见。

画外音固定用 says in an off-screen voiceover，并在画外音之后写明画面人物嘴唇保持闭合。台词跨切镜时，在两段的衔接处使用 `<scenetrans>` 并写明声音跨镜延续；台词被视频结尾截断时用 `<cutoff>`。

## 画面内文字

招牌、字幕、横幅等画面里真实出现的文字，用英文双引号原样保留，不翻译：A red neon sign reading "营业中" glows above the doorway.

## overall_soundscape：整体声景

用 1–4 句英文连成一段，概括全片的环境声、物体动作声和非语言人声（风、雨、脚步、织物摩擦、呼吸、笑声）。台词、歌声、剧情内音乐已经写在主描述里，不要在这里重复。只有用户明确要求全程静音时才写 N/A。

## non_diegetic_music：画外配乐

用 1–3 句描述只有观众能听到的背景音乐：乐器、速度、节奏、强弱变化。不要写抽象情绪词，也不要解释音乐的功能。角色能听到的音乐（收音机、现场演奏、手机播放）属于剧情内声音，写在主描述里。没有配乐就写 N/A。

## 语法错误速查

| 错误写法 | 纠正 |
| --- | --- |
| Shot 1 也加了时间戳 | 首镜不写时间戳，从 Shot 2 起递增 |
| 切镜时间超出视频时长 | 所有时间戳落在时长内并严格递增 |
| 台词被翻译成了英文 | 台词逐字保留原语言，标签里只加语言标记 |
| 风格词堆在句子末尾 | 风格在 [Shot 1] 开头一句声明 |
| 声景里重复台词内容 | 台词只出现在主描述 |
| 配乐写成“悲伤的音乐” | 写乐器、速度、节奏、强弱变化 |

## 原图完整性记录

- 原图右侧裁切了两条英文示例：推镜句和说话人台词句；本页保留可辨识字符并标记截断，没有补写不存在于图内的后文。
- 运镜表中 Pedestal、Arc Shot 两行被重复表头横条遮挡；可辨识词项已转录并标明遮挡位置。
- 原图“画外音”说明、“画面内文字”及“overall_soundscape”在滚动拼接处重复；本页各保留一次，并采用图中显示完整的段落。
- 原图可见重复水印为 `ikigai`，底部有点赞图标；原图本身保留了这些视觉元素。

---

[上一篇：H3 提示词入门](01-prompt-introduction.md) · [下一篇：H3 基础模式实战](03-basic-modes.md) · [返回目录](../../README.md)
