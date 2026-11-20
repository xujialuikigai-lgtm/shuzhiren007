# H3 数字人插件下载与使用指南

核对日期：2026-11-20。根据三个 GitHub 仓库的 README、节点源码与示例 JSON 整理。已完成静态核对，未安装插件、下载大模型、执行生成或测试付费 API。本文记录的是下表固定提交，`main` 后续更新可能改变接口。

## 1. 先选工作流

| 目标 | 从哪里开始 | 生成链与依赖 |
| --- | --- | --- |
| 一张人物图 + 完整口播音频 | LongVideo 的 **Talking Avatar 口播完整示例** | 当前示例：原生 H3 首尾条件 + 音频条件 + 标准 K 采样；还使用 KJNodes 内存优化节点。当前这份示例不使用 SelfLift / YCNodes |
| 完整歌曲 + 人物图 / 多段分镜 | LongVideo 的 **MV 完整示例** | Ref2VA + SelfLift-Avatar + YCNodes + KJNodes，可逐段修改图与提示词 |
| 单独研究 H3 低清到高清采样 / 音频遮罩 | SelfLift 自带示例 | 需要自行准备 H3 环境；其自带示例没有完整的外部音频驱动接线 |
| 二采高清精修、时间分段提示词或空间分块 | YCNodes 自带二采示例 | 在已有 H3 生成链上加入条件、模型补丁与采样辅助节点 |

依据：[LongVideo 当前说明](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/README.md#L3-L32)、[MV / 口播执行分工](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/MV_RUNNINGHUB.md#L13-L17)。截图里的“三个必装”是教程当时的表述，不能套用到全部当前工作流。

“无限时长”是把整条音频切成短段、逐段生成再合并；不是一次采样无限长视频。MV 单段上限为 15 秒或项目设置的更小值。当前口播每段使用同一张原图作为首尾约束，最终按 24fps 与完整原声合成；仍须检查嘴型、动作重复和接缝。[使用与分段说明](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/README.md#L24-L40)、[口播限制](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/README_TALKING_RH.md#L25-L29)。

## 2. 插件下载地址与核对版本

ZIP 下载后解压到 `ComfyUI/custom_nodes/`，使用表中目录名，避免重复副本和多套一层目录。仓库链接跟随最新代码；固定 ZIP 与本文核对版本一致。

| 插件 | 仓库 / 最新说明 | 固定源码 ZIP | 核对提交 | 目录名 |
| --- | --- | --- | --- | --- |
| H3 长视频编排 | [ComfyUI-H3-LongVideo](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo) | [下载 ZIP](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/archive/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e.zip) | `f8e1e4d` | `ComfyUI-H3-LongVideo` |
| SelfLift 音画采样 | [selflift-Avatar](https://github.com/slmonker/selflift-Avatar) | [下载 ZIP](https://github.com/slmonker/selflift-Avatar/archive/460f88629b35f792a2ca68b219f13c1ba46e2ee3.zip) | `460f886` | `selflift-Avatar` |
| H3 辅助节点 | [ComfyUI-YCNodes-MiniMax-H3](https://github.com/yichengup/ComfyUI-YCNodes-MiniMax-H3) | [下载 ZIP](https://github.com/yichengup/ComfyUI-YCNodes-MiniMax-H3/archive/efbe83f69ffa97a251a575ce42edc7bf8005da0c.zip) | `efbe83f` | `ComfyUI-YCNodes-MiniMax-H3` |
| 示例中的额外依赖 | [ComfyUI-KJNodes](https://github.com/kijai/ComfyUI-KJNodes) | [main ZIP](https://github.com/kijai/ComfyUI-KJNodes/archive/refs/heads/main.zip) | 未进行全仓库调研 | `ComfyUI-KJNodes` |

LongVideo 当前 MV 示例的 SelfLift 端口已经使用 `low_res_model`，与本次核对的 SelfLift 主分支接口一致；但这只是静态接口核对，不代表所有组合已实测。SelfLift 的旧 `v0.1.4-experimental` 发布包并不包含当前主分支的全部改动。[迁移说明](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/README_CN.md#L33-L69)。

### 工作流 JSON 下载

以下均为上游固定提交的原始 JSON。保存为 `.json` 后导入 ComfyUI；不要将 GitHub 网页 HTML 保存成 JSON。

| 工作流 | 下载地址 | 用途 |
| --- | --- | --- |
| LongVideo 口播完整工作流 | [下载 JSON](https://raw.githubusercontent.com/Zzz1phyrf/ComfyUI-H3-LongVideo/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/examples/MiniMaxH3%E5%8F%A3%E6%92%AD%E6%95%B0%E5%AD%97%E4%BA%BA%E5%B7%A5%E4%BD%9C%E6%B5%81%20By%20%E5%83%8F%E7%B4%A0%E5%B9%BB%E6%83%B3Lab.json) | 从整条语音与人物图生成分段口播 |
| LongVideo MV 完整工作流 | [下载 JSON](https://raw.githubusercontent.com/Zzz1phyrf/ComfyUI-H3-LongVideo/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/examples/MiniMaxH3MV%E6%95%B0%E5%AD%97%E4%BA%BA%E5%B7%A5%E4%BD%9C%E6%B5%81%20By%20%E5%83%8F%E7%B4%A0%E5%B9%BB%E6%83%B3Lab.json) | 歌曲、演唱与多段分镜 |
| SelfLift 自带示例 | [下载 JSON](https://raw.githubusercontent.com/slmonker/selflift-Avatar/460f88629b35f792a2ca68b219f13c1ba46e2ee3/example_workflows/H3%2Bselflift-avatar-fixed.json) | 研究采样链；外部音频驱动需另接链路 |
| YCNodes 二采分块示例 | [下载 JSON](https://raw.githubusercontent.com/yichengup/ComfyUI-YCNodes-MiniMax-H3/efbe83f69ffa97a251a575ce42edc7bf8005da0c/example/MiniMax_H3_refer%20tiled%20second%20sampling.json) | 二次采样、latent 放大与分块精修 |

## 3. 模型与辅助工具下载

插件源码不包含 H3 主权重。先按所选工作流检查 `UNETLoader`、`CLIPLoader`、两个 `VAELoader`、LoRA 与 upscaler 的文件名，再下载对应模型。口播首尾帧使用的 FL2VA 与多参考使用的 Ref2VA 不是可以随意互换的同一个配置。

模型入口：[Comfy-Org/MiniMax-H3](https://huggingface.co/Comfy-Org/MiniMax-H3/tree/main)。下表文件已在发布目录中核对存在；未实际下载、校验完整文件或验证其与全部示例的组合。只下载自己所需的一组，不必下载整个模型仓库。

| 类型 | 文件下载 | ComfyUI 下的放置目录 |
| --- | --- | --- |
| FL2VA 主模型，LongVideo 示例使用的文件名 | [minimax_h3_fl2va_int8_convrot.safetensors](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/diffusion_models/minimax_h3_fl2va_int8_convrot.safetensors) | `models/diffusion_models/` |
| Ref2VA 基准模型，SelfLift 示例注释列出 | [minimax_h3_ref2va_pruned_int8_convrot.safetensors](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/diffusion_models/minimax_h3_ref2va_pruned_int8_convrot.safetensors) | `models/diffusion_models/` |
| 文本编码器，LongVideo 示例文件名 | [qwen3vl_32b_minimax_h3_int8_convrot.safetensors](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/text_encoders/qwen3vl_32b_minimax_h3_int8_convrot.safetensors) | `models/text_encoders/` |
| 文本编码器，SelfLift 示例注释列出 | [qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/text_encoders/qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors) | `models/text_encoders/` |
| 视频 VAE | [minimax_h3_video_vae_fp16.safetensors](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/vae/minimax_h3_video_vae_fp16.safetensors) | `models/vae/` |
| 音频 VAE | [minimax_h3_audio_vae_fp32.safetensors](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/vae/minimax_h3_audio_vae_fp32.safetensors) | `models/vae/` |
| Ref2V 加速 LoRA，SelfLift 示例注释列出 | [minimax_h3_ref2v_turbo_4step_v0.1_comfyui_bf16.safetensors](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/loras/minimax_h3_ref2v_turbo_4step_v0.1_comfyui_bf16.safetensors) | `models/loras/` |

发布目录：[主模型](https://huggingface.co/Comfy-Org/MiniMax-H3/tree/main/diffusion_models)、[编码器](https://huggingface.co/Comfy-Org/MiniMax-H3/tree/main/text_encoders)、[VAE](https://huggingface.co/Comfy-Org/MiniMax-H3/tree/main/vae)、[LoRA](https://huggingface.co/Comfy-Org/MiniMax-H3/tree/main/loras)。

**还存在的模型缺口：**SelfLift 示例注释与实际加载节点的文件名不完全一致，实际节点还有 hybrid H3、量化 VAE、dmad LoRA、speed slider 等作者本地选择；LongVideo / SelfLift 示例中的 latent upscaler 与额外 LoRA 也不能从以上表格全部补齐。当前已查阅的三个插件说明没有提供这些文件的完整、相互匹配的下载清单，不能把上表称为“原样跑通整张画布所需的全部文件”。使用自己的模型组合时，应同步修改加载节点和采样参数；LongVideo 作者的口播示例允许不使用 LoRA。[SelfLift 示例源码](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/example_workflows/H3%2Bselflift-avatar-fixed.json)、[口播模型选择](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/README_TALKING_RH.md#L9-L11)。

- FFmpeg / ffprobe：[FFmpeg 官方下载入口](https://ffmpeg.org/download.html)，按操作系统选择现成构建并确保两个命令均可从 PATH 调用。
- LongVideo 人声分离检查点：[hdemucs_high_trained.pt](https://download.pytorch.org/torchaudio/models/hdemucs_high_trained.pt)，源码目标位置为 `ComfyUI/models/H3LongVideo/hdemucs_high_trained.pt`，首次人声提取也可能自动下载。[路径和校验来源](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/nodes.py#L21-L61)
- ASR 首次运行可能下载 faster-whisper 对应模型；提取人声需要与当前 PyTorch 匹配的 `torchaudio`。选“直接使用音频”可跳过人声提取，但不等于跳过其他全部分析步骤。[作者说明](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/README.md#L22-L40)

## 4. 安装步骤

前提：已有能启动的 ComfyUI，并具有工作流所需 H3 核心节点。示例导出元数据中的 ComfyUI 版本号不是已验证的最低兼容版本。若缺 `MiniMaxH3ImageToVideo` / `MiniMaxH3ReferenceToVideo` / `MiniMaxH3AddGuide`，要检查 ComfyUI 的 H3 支持，不能用反复安装 YCNodes 来代替核心节点。

### 4.1 安装源码

下面命令供用户操作参考，本次没有在本机执行安装。先进入实际的 `ComfyUI/custom_nodes` 目录；已有插件时不要重复 clone 到嵌套目录。

```bash
# LongVideo：MV 和口播两条路线都使用
git clone https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo.git

# 完整示例中的 KJNodes 内存处理节点
git clone https://github.com/kijai/ComfyUI-KJNodes.git

# MV 示例或独立使用 SelfLift / YC 时再装以下两个
git clone https://github.com/slmonker/selflift-Avatar.git
git clone https://github.com/yichengup/ComfyUI-YCNodes-MiniMax-H3.git
```

这些 clone 命令获取执行时的最新主分支。需要与本文固定版本对照时，使用第 2 节固定 ZIP；不要把旧节点与新示例混为一个已验证环境。

### 4.2 安装依赖

在 **ComfyUI 根目录**运行，将占位符换成 ComfyUI 真正使用的 Python 解释器。以下用 uv 适配作者要求的“在 ComfyUI Python 环境安装”，不是在另建的无关环境里安装。

```bash
uv pip install --python "<ComfyUI的Python解释器路径>" -r custom_nodes/ComfyUI-H3-LongVideo/requirements.txt
```

LongVideo 固定版本声明：`faster-whisper==1.2.1`、`requests>=2.31`、`soundfile>=0.12`、`mediapipe>=0.10.31,<0.11`。KJNodes 按其自身仓库说明安装对应依赖。SelfLift 未额外声明 pip 依赖；YCNodes 使用已有 PyTorch / ComfyUI 模块，这不代表无需基础环境。[LongVideo 依赖](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/requirements.txt#L1-L4)、[SelfLift 环境前提](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/README_CN.md#L186-L208)。

### 4.3 导入前检查

```bash
ffmpeg -version
ffprobe -version
```

1. 将权重放进对应目录，重启 ComfyUI 后端并刷新浏览器。
2. 导入所选 JSON，检查缺失节点及启动日志。
3. 逐个模型加载节点选择本机实际文件，不保留不存在的作者本地文件名。
4. 先验证一段生成及解码，再执行整条长音频。这是本指南的验证顺序建议，不是已经完成的实测。

## 5. 长口播怎么用

从第 2 节下载 **LongVideo 口播完整 JSON**。操作依据：[上游口播说明](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/README_TALKING_RH.md#L3-L29)。

1. `LoadAudio` 选择整条口播音频，`LoadImage` 选择一张包含人物及背景的参考图。
2. 在画布选好主模型、CLIP、音视频 VAE 和可选 LoRA，设置输出尺寸；选择器输出尺寸应与当前工作流匹配。
3. 在 **MiniMax H3 · Talking Avatar** 选择“本地”或“RunningHub”。干净语音可选直接使用音频；需要分离背景声时选择提取人声。
4. 按需要编辑共用提示词，当前示例使用“原图首尾锚定”策略。
5. 运行画布准备项目，打开分段面板；逐段试听，检查切点并保存确认。
6. 在分段面板点击“开始 / 继续生成”。全部完成后自动合并，或点“仅重新合并”。
7. 某段不满意时先“将本段加入重做”，再“开始 / 继续生成”。当前首尾锚定方案只重做选中段；改音频边界可能影响相邻段。

### 关键接线

```text
LoadAudio ──────────────→ Talking Avatar.audio
LoadImage ──────────────→ Talking Avatar.reference_image

Talking Avatar.reference_image → H3 ImageToVideo.first_frame 和 last_frame
Talking Avatar.frames/prompt/宽高 → H3 ImageToVideo 对应输入
Talking Avatar.audio → 音频 VAE 编码 / H3 AddGuide 音频条件链

H3 条件 + 模型 + AV latent → 标准 SamplerCustom 采样
采样 latent → 分离 AV latent → 视频 VAE 解码 → CreateVideo.images
Talking Avatar.audio ────────────────────────────→ CreateVideo.audio
CreateVideo → SaveVideo
```

当前口播 JSON 只解码采样结果的视频，CreateVideo 的音频直接取 Talking Avatar 的音频片段，没有单独的生成音频解码链。最终成片再使用整条原声。

这是对完整示例连接关系的说明，不是可单独导入的工作流。应保留原 JSON 的音频条件、采样、解码和保存节点。当前口播不应为了套用旧教程而强行换成 MV 的 SelfLift 链。[固定口播 JSON](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/examples/MiniMaxH3%E5%8F%A3%E6%92%AD%E6%95%B0%E5%AD%97%E4%BA%BA%E5%B7%A5%E4%BD%9C%E6%B5%81%20By%20%E5%83%8F%E7%B4%A0%E5%B9%BB%E6%83%B3Lab.json)

当前口播每段共用同一原图，并不要求把前段尾帧接给后段。修改画布模型、LoRA、强度或步数后运行画布同步；只换图可用“同步画布参考图（保留切点）”。已有生成记录时可能建立新版本以保留旧项目；更换音频或尺寸需要重新分析。[参数同步与换图](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/README_TALKING_RH.md#L9-L23)

## 6. 歌曲 MV 怎么用

从第 2 节下载 **LongVideo MV 完整 JSON**。MV 使用另外两个插件的节点，先补齐 SelfLift、YCNodes、KJNodes 及画布需要的其他定义。

1. 加载完整歌曲，在 **MiniMax H3 · MV Generator** 选择后端及分段参数。
2. 准备分段，在审核面板上传参考图、试听并调整切点。可逐段使用不同参考图、编辑分镜与提示词。
3. **MV Prompt Expander** 有视觉、文本、手工模式。视觉 / 文本扩写需要单独配置可用的聊天模型 API；手工模式使用自写提示词，不调用该扩写 API。
4. 检查 Ref2VA 所需参考图、人声音频、提示词和采样链；选择实际存在的主模型、CLIP、双 VAE、LoRA 和 latent upscaler。
5. 保存审核结果，再从分段面板开始生成；单段可重做，完成后合并并使用完整原曲。

```text
完整歌曲 → MV Generator → 原声片段 / 人声片段 / 分段素材包
分段素材包 + 参考图 → Prompt Expander → Ref2VA 提示词
参考图 + 人声片段 + 提示词 → H3 Ref2VA 条件 / AV latent
条件 + 模型 + sigmas → SelfLiftAvatarH3Sampler → 分离 AV latent → 视频解码
视频帧 + original_audio_padded → CreateVideo / 保存片段
所有审核通过的片段 → LongVideo 合并视频 + 完整原曲
```

依据：[MV 使用说明](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/README.md#L22-L40)、[扩写 API 与手工分支](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/expansion.py#L571-L623)、[最终合并实现](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/controller.py#L737-L799)。

当前 MV 示例的片段封装音频来自 Generator 的 `original_audio_padded`，与 SelfLift 单独示例中解码生成音频的分支不同；最终合并仍使用完整原曲。

## 7. RunningHub 与扩写 API 怎么配置

| 服务 | 负责什么 | 需要的配置 |
| --- | --- | --- |
| 本地 H3 | 本地模型执行单段画面采样 | ComfyUI、节点、本地权重和可用计算资源 |
| RunningHub | 云端执行单段工作流 | 自己有权访问的匹配工作流 ID、区域、实例和该站 API Key |
| MV 提示词扩写 API | 辅助模型生成提示词，不负责 H3 采样 | 扩写模型、BaseURL、Key；视觉模式发送参考图 |

RH 操作：将与 MV / 口播相匹配的单段计算工作流导入并保存到自己的 RH 工作台，取得自己的 ID；控制节点选 RunningHub，填 ID / 区域 / 实例，通过“RH API 设置”保存 Key，先点“检查 RH 配置（不生成）”，再分析、审核和生成。此处提供的两份完整本地画布不等于已经替你创建好的 RH 云端工作流 ID。[上游配置说明](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/MV_RUNNINGHUB.md#L3-L17)

RH 模式仍需要本地 ComfyUI 与画布节点定义，但不在本地加载 H3 生成权重；本地负责音频分析、项目管理、下载、预览和合并，RH 一次生成一段。国内 / 海外站 Key 分开配置。Key 存 ComfyUI 用户目录 `H3LongVideo/runninghub_api.json`，不写入工作流；扩写 API 配置独立。[执行分工与设置](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/MV_RUNNINGHUB.md#L5-L17)

截图列出的 `https://www.zzai.app/v1` 不能填成 RunningHub 地址。源码中的 RH 站点是 [runninghub.cn](https://www.runninghub.cn) / [runninghub.ai](https://www.runninghub.ai)；扩写调用的是单独配置的 `/chat/completions`。截图的转发平台是否兼容当前扩写节点尚未验证。[RH 基址](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/mv_rh.py#L29-L41)、[扩写调用](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/expansion.py#L571-L623)

## 8. 单独使用 SelfLift 与 YCNodes

### SelfLift：采样与音频保留

SelfLift 是本地实验采样器，输出 LATENT，后面仍需 VAE 解码与视频保存；不是独立数字人基础模型，也不是后期对嘴软件。主节点要求标准 Euler，`s_churn=0`。[节点与采样限制](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/nodes.py#L119-L142)

外部音频驱动应按作者说明建立：

```text
加载 / 裁剪输入音频 → H3 音频 VAE 编码 → 音频 mask=0
视频 latent + 音频 latent → 合并 AV latent → SelfLift H3 Sampler
采样结果 → 视频 VAE / 音频 VAE 解码 → 合成视频
```

`mask=0` 保留已有内容，`mask=1` 生成；输入 latent 必须真正包含音频，单设 0 不会凭空产生声音。音频长度、帧数与 FPS 应对应同一段时间。**SelfLift 自带 JSON 没有完整的 LoadAudio / 音频编码 / mask=0 链路，不能导入后仅换音频就声称已完成驱动。**做完整长音频项目优先用 LongVideo 对应示例编排。[音频连接规范](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/README_CN.md#L240-L257)、[自带示例](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/example_workflows/H3%2Bselflift-avatar-fixed.json)

当前主节点端口为 `low_res_model` / `high_res_model`；旧工作流的 `model` / `model_hires` 可能需要重接。省显存有 spatial、global_stream、global、global_ram 等路径，兼容性和峰值不同；保留音轨不等于嘴型必然正确，没有固定“多少 GB 显存一定能跑”的承诺。[版本迁移与模式](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/README_CN.md#L33-L105)

SelfLift 的 `spatial` 路径带遮罩时仅支持视频全 1、音频全 0；`global` 系列可处理全序列局部 / 软遮罩，具体兼容条件见[模式说明](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/README_CN.md#L71-L105)。

### YCNodes：7 个实际节点

| 节点 | 插入位置 / 用途 |
| --- | --- |
| `MiniMaxH3ImageToVideoTail` | 一采 LATENT 接 `video_latent`，重算首末帧条件后续跑二采 |
| `H3PromptRelay` | MODEL / CONDITIONING / CLIP / LATENT 接入，局部 prompt 用 `\|` 分段，控制不同时间段关注的文本 |
| `H3DistanceAttentionPatcher` | 模型与采样器间的距离注意力补丁 |
| `H3DynamicCFGScheduler` | 模型与采样器间动态改变 CFG |
| `H3SigmaRefiner` | 调度器的 SIGMAS → 本节点 → 采样器，增加低噪尾段步数 |
| `H3TiledSampler` | 二采用空间 H×W 分块替换对应采样器，分块路径透传音频 |
| `H3CondNoiseAug` | 向 CONDITIONING 写参考加噪系数，由兼容下游实现读取 |

README 还写“6 个节点”，实际源码有第 7 个 `H3CondNoiseAug`。Tiled 参数也已从文档里的 `tile_axis/n_tiles` 改成 `tile_mode/h_tiles/v_tiles/vram_budget_frac`；以安装版本的实际面板为准。分块适合低噪二采精修，不应据此替代纯噪声起点的全局生成。[参考加噪源码](https://github.com/yichengup/ComfyUI-YCNodes-MiniMax-H3/blob/efbe83f69ffa97a251a575ce42edc7bf8005da0c/py/h3_cond_noise_aug.py#L25-L62)、[当前采样参数](https://github.com/yichengup/ComfyUI-YCNodes-MiniMax-H3/blob/efbe83f69ffa97a251a575ce42edc7bf8005da0c/py/h3_tiled_sampler.py#L273-L346)、[二采适用范围](https://github.com/yichengup/ComfyUI-YCNodes-MiniMax-H3/blob/efbe83f69ffa97a251a575ce42edc7bf8005da0c/README.md#L131-L137)

## 9. 成片位置、重做与排错

| 位置 | 内容 |
| --- | --- |
| `ComfyUI/output/H3LongVideo/MV/final_videos/` | MV 完整成片 |
| `ComfyUI/output/H3LongVideo/口播/final_videos/` | 口播完整成片 |
| 同级 `projects/` | 素材、项目状态与任务记录，恢复项目要保留 |
| 同级 `logs/` | 报错日志 |

节点上的“下载错误日志”会生成 `H3LongVideo-errors.log`，用于查看错误阶段、节点和堆栈。[输出与日志说明](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/README.md#L42-L54)

| 现象 | 检查顺序 |
| --- | --- |
| 缺 H3 核心节点 | 检查 ComfyUI H3 支持与节点启动日志，再核对插件来源 |
| 红色节点 / 模型下拉框缺文件 | 核对目录、文件名、模型类别和实际安装版本；不要只保留示例作者本地选择 |
| SelfLift 接口失配 | 对照 `low_res_model/high_res_model`，检查旧图与新节点接口 |
| 有声音但嘴型不对 | 检查音频是否进入生成条件、长度与帧数；不能只看最终 MP4 是否有原音轨 |
| 显存不足 | 核对时长、尺寸、权重与省显存模式；RH 是另一执行后端，画质和性能仍需实测 |
| ASR / 人声提取失败 | 下载日志看具体错误；检查 ffmpeg、ffprobe、torchaudio 匹配、模型下载和联网情况 |
| RH 查询 / 下载失败 | 继续使用原任务 ID 恢复；创建结果未知时先在 RH 核实并绑定原 ID，避免直接重复提交 |
| 仅需重封装成片 | 使用“仅重新合并”；该操作不重新调用 RH 生成 |

RH 恢复行为依据：[任务恢复说明](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/MV_RUNNINGHUB.md#L19-L28)。

## 10. 教程截图中的补充资源

以下保留截图给出的入口，**未验证内容、访问权限、提取码或与本文固定版本的兼容性**；与上文已查阅源码分开记录。

| 项目 | 截图地址 |
| --- | --- |
| B 站视频 | [BV1tJW76dEww](https://www.bilibili.com/video/BV1tJW76dEww) |
| 工作流云端镜像 | [chenyu.cn](https://www.chenyu.cn/api/s/rVOGXmJp) |
| API 平台 | [api.seedance.nz](https://api.seedance.nz/?s=G5rK) |
| BaseURL | `https://www.zzai.app/v1` |
| 模型 / 工作流夸克网盘 | [夸克](https://pan.quark.cn/s/82e7c58eb651) |
| 模型 / 工作流百度网盘 | [百度](https://pan.baidu.com/s/1Os3s0rA8-DV1REZOeR2bWw)，提取码 `xshx` |
| 像素幻想 Lab AI 知识库 | [飞书](https://my.feishu.cn/wiki/space/7577629224702331856?ccm_open_type=lark_wiki_spaceLink&open_tab_from=wiki_home) |

## 11. 本次验证范围

已读取三个仓库的固定提交文档、关键源码与示例工作流，核对当前节点和模型加载选择、下载入口、音频条件和本地 / RH 分工；模型表中的文件名另与 Comfy-Org 发布目录核对。详细依据见[逐仓库源码笔记](plugins-source-notes.md)。

未完成实机安装、全套模型组合验证、MV / 口播推理、显存和耗时测量，也未验证截图第三方平台。各上游示例可能仍需补模型、安装缺失节点或迁移端口；本指南不将静态核对称为“已经跑通”。

[返回仓库目录](../../README.md) · [提示词结构](02-prompt-structure.md) · [Ref2VA 提示词](04-reference-mode.md)
