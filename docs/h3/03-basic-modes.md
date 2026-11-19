# 🖼️ H3 基础模式实战：T2VA / I2VA / FL2VA / L2VA

- 作者：像素幻想Lab
- 原图标注：9月5日修改
- 资料状态：方法参考
- 复核：2026-09-05
- [查看原图](../../images/h3/03-h3-basic-modes.png)

> 整理说明：本文转录原图可见内容。滚动拼接重复的段落和代码块已合并；被表头遮挡的表格行已注明；代码右侧超出原图的部分统一标注 `[原图右侧截断]`。截断标记是整理注释，不属于可直接执行或使用的提示词。没有根据上下文补写不可见的英文内容。图片中的来源和模型说明按原文保留，未另作事实核验。

> **资料状态：方法参考 | 复核：2026-09-05。** 示例改编自 MiniMax H3 官方提示词指南。提示词正文为英文，讲解为中文。

四种基础模式共用三大字段，差别只在两处：开头有没有“参考图对齐指令行”，以及主描述怎么围绕参考图展开。先选对模式，再套对应结构。

## 选模式速查

| 模式 | 开头指令行 | 主描述展开策略 |
| --- | --- | --- |
| T2VA 纯文本 | 无 | 直接从文本构建完整时间线 |
| I2VA 首帧图 | 首帧完全引用 Picture 1（0.00 秒） | 首帧锚定 → 动作启动 → 连续发展 → 结果反应 |
| FL2VA 首尾帧 | Picture 1 对齐 0.00 秒，Picture 2 对齐 S.SS 秒 | 首帧状态 → 中间变化 → 差异收窄 → 末帧落地 |
| L2VA 尾帧 | Picture 1 对齐 S.SS 秒（属于最后一个镜头）[原图表头遮挡部分字形，按可辨内容转录] | 合理前状态 → 动作过渡 → 末镜收敛 → 落版 [原图表头遮挡部分字形，按可辨内容转录] |

三行指令行都有固定写法。I2VA 固定为：

```text
For the target video, at 0.00 seconds into the target video, <Picture 1> (from [Shot 1]) is[原图右侧截断]
```

FL2VA 固定为（N 是最后一个镜头的编号，S.SS 是视频总时长，保留两位小数）：

```text
How the reference pictures align with the target video – Picture 1 (from Shot 1) aligns wi[原图右侧截断]
```

L2VA 固定为：

```text
How the reference pictures align with the target video – <Picture 1> (from [Shot N]) aligns[原图右侧截断]
```

指令行必须是提示词的第一行，之后空一行再写三大字段。

## T2VA：从零搭建时间线（早餐店案例）

没有参考图时，你可以在符合用户意图的前提下，自行补充场景、人物、动作和声音细节，把时间线写满。

原图代码块标题：T2VA 完整示例。其可见内容如下（原图右侧截断，正文并不完整）：

```text
integrated_multimodal_description: [Shot 1] Live-action, cinematic, a medium-wide shot fra[原图右侧截断]

overall_soundscape: Wooden shutters scrape open over a quiet street as trays clink softly[原图右侧截断]

non_diegetic_music: A soft acoustic-guitar pattern at a moderate tempo, joined by sparse up[原图右侧截断]
```

拆解：Shot 1 无时间戳，风格句开头；运镜三要素完整；台词带编号和音色；Shot 2 在第 5 秒切镜并带来新信息（蒸汽特写），台词用 carry over 跨镜延续；声景只写环境与动作声；配乐写了乐器、速度和收尾方式。

## I2VA：从首帧发展（火车窗边案例）

Picture 1 就是第 0 秒的真实画面，属于 [Shot 1]。先用一句话确认图里的人物、服装、位置和环境保持不变，再从图里“长”出新动作。

原图代码块标题：I2VA 完整示例。其可见内容如下（原图右侧截断，正文并不完整）：

```text
For the target video, at 0.00 seconds into the target video, <Picture 1> (from [Shot 1]) is[原图右侧截断]

integrated_multimodal_description: [Shot 1] Live-action, cinematic, the young woman shown[原图右侧截断]

overall_soundscape: The train wheels produce a steady metallic rhythm beneath a low ventila[原图右侧截断]

non_diegetic_music: Sustained cello notes at a slow tempo with widely spaced piano tones, g[原图右侧截断]
```

拆解：描述从“图里已有的东西不动”开始（remains beside... preserving...），然后才是新动作（抬眼、折信）；运镜 trucks right 小幅度慢速，符合车内空间的克制感。

## FL2VA：首尾帧之间铺路径（雨中开伞案例）

两张图分别是开头和结尾。正文不要重复描述两张静态图，而是写连接它们的运动路径。FL2VA 通常用单镜头，方便模型从首帧连续插值到末帧；确需多镜头时要明确写出。末帧必须在最后一个镜头结束时到达。

原图代码块标题：FL2VA 完整示例（8 秒单镜头）。其可见内容如下（原图右侧截断）：

```text
How the reference pictures align with the target video – Picture 1 (from Shot 1) aligns wi[原图右侧截断]

integrated_multimodal_description: [Shot 1] Live-action, cinematic, a rain-soaked cyclist b[原图右侧截断]

overall_soundscape: Rain falls steadily on the pavement, followed by the metallic click of[原图右侧截断]

non_diegetic_music: N/A
```

拆解：从 Picture 1 的姿态和构图起步（begins in the position and framing established by Picture 1），中间是一串可观察的动作（松把手、举伞、撑开、旋转伞柄），结尾显式写按 Picture 2 落定（settles into ... established by Picture 2）。用户没提配乐，写 N/A。

## L2VA：收敛到尾帧（玻璃杯坠落案例）

Picture 1 是视频的最后一帧，属于最后一个镜头，不天然属于开头。先虚构一个与意图一致的合理前状态，再让人物、物体、机位逐步逼近参考图。

原图代码块标题：L2VA 完整示例（6 秒单镜头）。其可见内容如下（原图右侧截断）：

```text
How the reference pictures align with the target video – <Picture 1> (from [Shot 1]) aligns[原图右侧截断]

integrated_multimodal_description: [Shot 1] Live-action, cinematic, a close shot begins wi[原图右侧截断]

overall_soundscape: Fingertips tap the glass before it scrapes across the tabletop, falls,[原图右侧截断]

non_diegetic_music: A low electronic pulse at a slow tempo, ending immediately after the g[原图右侧截断]
```

拆解：开头状态（完好的杯子在桌边、图中同款手入画）与尾帧兼容；中段是完整动作链；结尾显式写“碎片停滑，落成 Picture 1 的确切布局、手位、机位和光线”。

## 时长核对

指令行和切镜时间必须与目标时长一致：8 秒写 8.00-second mark，6 秒写 6.00；S.SS 一律保留两位小数。切镜时间严格递增，最后一段内容要在时长内完成，不能写一个 15 秒的切镜去填一条 8 秒的视频。

## 原图完整性说明

- 已保留引言、模式表、三种固定指令行、四个案例的说明/可见代码/拆解，以及时长核对。
- 原图多次重复了固定指令行、T2VA 标题和说明、I2VA 标题和说明，重复部分合并为一次；拼接造成的其他段落重复残片亦已合并。
- L2VA 模式表行被重复表头遮挡，表格内已明确标注。
- 英文代码块没有显示横向溢出部分，无法从这张图片恢复完整提示词；可见的尾部词片段保留在截断标记之前。
- 原图重复水印为 `ikigai`；作者头像、点赞图标等界面装饰未转写为正文，原始图片完整保留。

---

[上一篇：H3 提示词结构解析](02-prompt-structure.md) · [下一篇：H3 全参考模式实战](04-reference-mode.md) · [返回目录](../../README.md)
