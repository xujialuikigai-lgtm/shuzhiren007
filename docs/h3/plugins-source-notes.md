# H3 插件逐仓库源码笔记

核对日期：2026-11-20。本文是[下载与使用指南](plugins-guide.md)的详细依据，保留各仓库固定提交、实现边界、下载入口和接线说明。仅静态调研，未执行模型生成。

## ComfyUI-H3-LongVideo 源码调研

调研日期：2026-11-20。只读研究 README、Python 实现及两份示例工作流，未安装依赖、下载权重或执行生成。

固定版本：[`f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e`](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/commit/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e)，提交信息为“Open generation controls for approved talking projects waiting to run”。

### 核心定位

这是一个 ComfyUI 长音频分段生成与项目控制插件。它负责准备音频、切分和审核、素材及提示词、逐段执行、重做恢复和最终合并，单段画面仍由 H3 模型与采样链生成。支持本机 ComfyUI 和 RunningHub 两种执行后端。公开的两个独立控制节点为 MV Generator、Talking Avatar；另有 MV Prompt Expander。依据：[README 3–5](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/README.md#L3-L5)、[注册节点表](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/nodes.py#L496-L518)。

### 当前两条工作流的区别

| 路线 | 输入与用途 | 单段条件与采样 |
| --- | --- | --- |
| MV Generator | 完整歌曲；面板里上传参考图、逐段编辑分镜和提示词，适合演唱/音乐MV；可逐段不同参考图 | Ref2VA、SelfLift-Avatar；示例使用 YC 的 H3SigmaRefiner、KJNodes 内存处理节点 |
| Talking Avatar | 完整口播音频、一张人物连场景图、宽高、共用提示词、后端及音频处理选项 | 当前完整示例默认“原图首尾锚定”，同一原图作为每段首尾约束，原生 H3 ImageToVideo + AddGuide + 标准 SamplerCustom/K 链；不是 MV 的 SelfLift 采样链 |

依据：[README 22–32](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/README.md#L22-L32)、[RH执行分工13–17](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/MV_RUNNINGHUB.md#L13-L17)、[TalkingAvatar 输入输出与策略](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/talking_rh/__init__.py#L236-L284)。

实际两份示例：[MV完整画布](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/examples/MiniMaxH3MV%E6%95%B0%E5%AD%97%E4%BA%BA%E5%B7%A5%E4%BD%9C%E6%B5%81%20By%20%E5%83%8F%E7%B4%A0%E5%B9%BB%E6%83%B3Lab.json)、[口播完整画布](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/examples/MiniMaxH3%E5%8F%A3%E6%92%AD%E6%95%B0%E5%AD%97%E4%BA%BA%E5%B7%A5%E4%BD%9C%E6%B5%81%20By%20%E5%83%8F%E7%B4%A0%E5%B9%BB%E6%83%B3Lab.json)。JSON中的节点类型、模型文件名及属性已逐项读取。

**依赖结论**：教程截图里的三个仓库不能解释成所有当前口播工作流都必须同时安装。MV 示例确实使用另外两个插件；当前口播示例未使用 SelfLift-Avatar 与 YC 节点，但仍使用 KJNodes 两个内存优化节点。作者 README 也明确将两条路线分开。旧项目可能仍依赖旧节点，应按实际画布安装。

### 节点接口

- `H3LVUnified`（MiniMax H3 · MV Generator）：完整 AUDIO、singing 模式、最长/目标分段时长、ASR参数、项目/片段索引；可选本地/RH和音频处理。审核前运行只分析，确认后控制器才逐段加载。输出包括原声片段、人声片段、生成帧数、保存前缀、素材包、最多6张参考图、片段提示词和24fps。代码：[Analyze定义与完整音频覆盖校验](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/nodes.py#L287-L368)、[LoadSegment输出](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/nodes.py#L371-L425)、[Unified](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/nodes.py#L428-L481)。
- `H3LVPromptExpand`（MiniMax H3 · MV Prompt Expander）：素材包、vision/text/manual模式、扩写模型名、规则和修订号，输出 h3_prompt。默认示例为vision，使用图像和分镜包调用可配置的聊天补全API，生成六字段Ref2VA英文提示词。manual分支无需此模型调用。代码：[扩写分支与HTTP调用](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/expansion.py#L571-L623)、[节点接口](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/expansion.py#L645-L660)。
- `H3LVTalkingAvatar`（MiniMax H3 · Talking Avatar）：AUDIO、参考IMAGE、宽高、后端、RH工作流ID、区域/实例、项目、fixed_prompt、直接使用/提取人声，以及生成策略。输出 project_id、audio、reference_image、frames、width、height、prompt、fps、filename_prefix、reference_audio。代码：[定义与接口](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/talking_rh/__init__.py#L236-L284)。

另注册了旧口播、内部参考捕获、遮罩、CLIP读取、分段状态、合并、冻结前缀恢复节点。当前公开节点与兼容节点不能混淆。依据：[节点映射](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/nodes.py#L496-L518)。

### “无限时长”的实现与边界

1. **实质是多段生成再合成**，没有一次无限长采样。MV最大分段设置5–15秒；自动切分结合音量包络、ASR文字/字时间戳和节奏结构，要求能用4秒至最大时长覆盖音频；UI可人工审核和改切点。依据：[分段逻辑](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/core.py#L371-L395)、[README 26–34](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/README.md#L26-L34)。
2. 当前首尾锚定口播的非末段长度遵循107到345帧、间隔17帧的网格，末段可补齐。按24fps即约4.46–14.38秒的合法生成长度，而不是任意秒数。分段优先选停顿并权衡段长；末段保留真实长度。依据：[flat.py 17–91](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/talking_rh/flat.py#L17-L91)。
3. **连续声音来自整条原声**：MV合并先去除片段音轨、按实际保留帧裁切并统一24fps，再concat视频与source.wav；首尾锚定口播也按帧拼接并复用source.wav。不是靠每段新生成的音频拼接维持整曲。依据：[MV assemble](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/controller.py#L737-L799)、[口播 assemble](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/talking_rh/flat.py#L230-L259)。
4. 当前口播同图首尾约束优先稳定衣色、构图和画面尺度，但作者明确承认手势相似、回到参考姿态、口型自然度需逐片检查。因此“口播一镜到底”应理解为固定画面风格下的分段成片，不能保证物理上连续的一次拍摄或绝对无缝。依据：[口播说明7–29](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/README_TALKING_RH.md#L7-L29)。
5. 已完成片段支持局部重做，成功结果保留；未知RH任务要先绑定原ID避免重复提交，暂停等待当前任务结束；旧“前段续接”项目可能重建依赖的后续段，而当前同图首尾口播和MV只重做选中段。依据：[README 32–34](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/README.md#L32-L34)、[恢复规则](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/MV_RUNNINGHUB.md#L19-L26)。
6. 源码未声明“任意总时长必然成功”的保证；增加总长意味着更多采样任务、磁盘素材、执行时间和可能的RH费用。这是由逐段执行架构推得的实际限制，而非已测量上限。MV片段索引UI上限10000也说明“无限”并非严格数学意义。依据：[节点索引范围](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/nodes.py#L371-L380)。

### 本地权重、ComfyUI版本、硬件与API

- 本地模式：真正执行本机ComfyUI采样队列，不使用RH Key或云权重；需用户安装真实存在的本地模型。依据：[本地模型校验](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/talking_rh/local.py#L35-L63)、[本地入队](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/talking_rh/local.py#L100-L111)。
- 两份示例的模型名包含 `minimax_h3_fl2va_int8_convrot.safetensors`、`qwen3vl_32b_minimax_h3_int8_convrot.safetensors`、H3视频VAE和音频VAE、两组LoRA；MV采样器还配置 `h3_upscaler_lms_v0.1.safetensors`。这些是画布参考文件名，不代表插件自带或自动提供权重。证据为上述两份固定版本示例；README明确要求选自己的模型，也支持不使用LoRA。[安装说明](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/README.md#L9-L22)、[口播模型参数](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/README_TALKING_RH.md#L9-L11)。
- ComfyUI：README没有指定另一个ComfyUI fork；示例将 H3ReferenceToVideo/ImageToVideo/AddGuide 标成 `comfy-core`。MV示例记录core `0.42.5`，口播示例记录 `0.44.0`，只是导出时元数据，不是已证明的最低兼容版本。必须具有实际画布所需原生H3节点，不能把它们误认为YC插件提供的节点。示例KJNodes记录commit `74b7f1731cfd0208ae50b9ffc55145564c25e65f`；MV SelfLift记录 `5b93b6c0ce4f13cbdf1358b5f8d54433cb2540fc`，YC记录 `efbe83f69ffa97a251a575ce42edc7bf8005da0c`。
- README未给固定显卡/显存门槛，不能据此承诺低配机器可本地采样。依赖列表为 `faster-whisper==1.2.1`、`requests>=2.31`、`soundfile>=0.12`、`mediapipe>=0.10.31,<0.11`，另需ffmpeg/ffprobe；提取人声还需与PyTorch匹配的torchaudio。ASR支持CUDA/CPU并在auto时回退CPU；这仅适用于ASR，不能外推为H3生成同样可流畅CPU运行。[requirements](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/requirements.txt#L1-L4)、[ASR实现](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/worker.py#L59-L115)。
- 人声提取使用torchaudio `hdemucs_high`，以8秒窗口和1秒重叠处理，首次可能下载检查点；ASR首次也可能下载faster-whisper模型。[提取实现](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/nodes.py#L197-L245)、[ASR缓存与下载](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/worker.py#L39-L56)。
- RunningHub模式：RH每次只计算一段，本地不加载H3生成权重，但仍需本地ComfyUI和画布节点定义；要将配套单段云图另存到自己的RH工作台，填自己的ID、区域、实例与Key。不是填任意OpenAI BaseURL就能调用H3。Key存用户目录 `H3LongVideo/runninghub_api.json`，兼容环境变量 `RUNNINGHUB_API_KEY`，扩写API独立配置。[RH说明](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/MV_RUNNINGHUB.md#L3-L17)。
- Prompt Expander调用可配置的 `/chat/completions`；图文扩写会将缩放后的参考图以base64上传所配置模型服务，text仅文本，manual无需调用该接口。该API承担提示词扩写，不能与H3权重生成或RH执行API混淆。[扩写处理](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/expansion.py#L571-L623)。

### 下载地址与可复现使用步骤

以下命令和步骤只供操作参考，本次未执行安装。

#### 插件与工作流下载

- 插件仓库：[GitHub](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo)。固定版本源码包：[ZIP下载](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/archive/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e.zip)。解压安装时目录应为 `ComfyUI/custom_nodes/ComfyUI-H3-LongVideo`，不要多嵌套一层。
- MV工作流，固定版本原始JSON：[下载](https://raw.githubusercontent.com/Zzz1phyrf/ComfyUI-H3-LongVideo/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/examples/MiniMaxH3MV%E6%95%B0%E5%AD%97%E4%BA%BA%E5%B7%A5%E4%BD%9C%E6%B5%81%20By%20%E5%83%8F%E7%B4%A0%E5%B9%BB%E6%83%B3Lab.json)。跟随main的原始JSON：[最新入口](https://raw.githubusercontent.com/Zzz1phyrf/ComfyUI-H3-LongVideo/main/examples/MiniMaxH3MV%E6%95%B0%E5%AD%97%E4%BA%BA%E5%B7%A5%E4%BD%9C%E6%B5%81%20By%20%E5%83%8F%E7%B4%A0%E5%B9%BB%E6%83%B3Lab.json)。
- 口播工作流，固定版本原始JSON：[下载](https://raw.githubusercontent.com/Zzz1phyrf/ComfyUI-H3-LongVideo/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/examples/MiniMaxH3%E5%8F%A3%E6%92%AD%E6%95%B0%E5%AD%97%E4%BA%BA%E5%B7%A5%E4%BD%9C%E6%B5%81%20By%20%E5%83%8F%E7%B4%A0%E5%B9%BB%E6%83%B3Lab.json)。跟随main的原始JSON：[最新入口](https://raw.githubusercontent.com/Zzz1phyrf/ComfyUI-H3-LongVideo/main/examples/MiniMaxH3%E5%8F%A3%E6%92%AD%E6%95%B0%E5%AD%97%E4%BA%BA%E5%B7%A5%E4%BD%9C%E6%B5%81%20By%20%E5%83%8F%E7%B4%A0%E5%B9%BB%E6%83%B3Lab.json)。保存为 `.json` 后在ComfyUI导入；不要把网页HTML保存成JSON。
- H3主权重、Qwen编码器、VAE和LoRA：本仓库README及示例没有给出可验证的这些权重下载页，不猜造下载地址。需要从教程提供的网盘或各权重发布者另行获得，并核对模型类别和文件名。
- 人声分离检查点有源码明确的直接地址：[hdemucs_high_trained.pt](https://download.pytorch.org/torchaudio/models/hdemucs_high_trained.pt)，源码预期334697255字节、SHA-256 `a004b2790d73ffeaa535db458a1a79b539dfdbafbccc31f275d07e632ebd7816`；目录为 `ComfyUI/models/H3LongVideo/hdemucs_high_trained.pt`。[地址与路径源码](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/nodes.py#L21-L61)。

#### 安装与本地使用

在已有ComfyUI的 `custom_nodes` 目录执行：

```bash
git clone https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo.git
```

在ComfyUI目录，使用管理该ComfyUI环境的uv安装依赖（将占位符替换为实际解释器路径）：

```bash
uv pip install --python "<ComfyUI使用的Python解释器路径>" -r custom_nodes/ComfyUI-H3-LongVideo/requirements.txt
```

此uv命令是对作者“使用ComfyUI Python安装requirements”的环境适配写法，非原仓库逐字命令。另需ffmpeg/ffprobe可从PATH调用，以及画布实际需要的外部节点和模型；不能在错误的独立Python环境里安装依赖后期待ComfyUI自动可见。[安装依据](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/README.md#L7-L22)。

1. 重启ComfyUI、刷新浏览器，分别导入MV或口播JSON。若出现缺失节点，按节点来源补齐；两张画布不能混用RH工作流ID。
2. 口播：LoadAudio加载整条语音，LoadImage加载人物连背景参考图，选择画布模型、LoRA（可不用）、CLIP和两种VAE；分辨率宽高应为32倍数。Talking Avatar选择“本地”，示例策略为“原图首尾锚定”；可改共用提示词。干声选直接使用；有背景音乐可选提取人声。
3. 口播连接关系（来自实际示例）：LoadAudio.AUDIO→TalkingAvatar.audio；LoadImage.IMAGE→TalkingAvatar.reference_image；ResolutionSelector宽高→TalkingAvatar宽高；TalkingAvatar.reference_image同时接MiniMaxH3ImageToVideo.first_frame/last_frame；TalkingAvatar.audio接音频VAE编码及MiniMaxH3AddGuide.audio；frames/prompt/width/height接ImageToVideo对应输入。采样采用SamplerCustom，解码及CreateVideo/SaveVideo按原画布保留。无需手工重建，导入完整JSON最直接。
4. MV：加载整曲，MV Generator选择“本地”，在审核面板上传参考图、检查切点及逐段提示词；如使用视觉/文本扩写，先给Prompt Expander配置可用的模型API、BaseURL和Key；也可manual手工给定提示词。实际示例中Generator的人声音频和最多6张参考图接Ref2VA，segment_material接PromptExpander，条件与AV latent进入SelfLiftAvatarH3Sampler，再分离、解码、保存。
5. 点击画布运行准备项目，进入分段面板试听、检查并保存确认；之后在面板点击“开始/继续生成”。全部片段完成自动合并，或点“仅重新合并”。需要局部重做时先“将本段加入重做”，再“开始/继续生成”。[使用依据](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/README.md#L24-L40)、[口播步骤](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/README_TALKING_RH.md#L13-L23)。
6. 在 `ComfyUI/output/H3LongVideo/MV/final_videos` 或 `ComfyUI/output/H3LongVideo/口播/final_videos` 取完整成片；projects是素材、状态和内部任务，logs是日志。[输出目录](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/README.md#L42-L50)。

#### RunningHub使用

本地ComfyUI仍需安装插件与节点定义并导入完整示例。另将与MV/口播对应的单段计算工作流导入自己的RunningHub工作台、保存，取得自己的workflow ID；在控制节点选择RunningHub、填ID/区域/实例，通过“RH API设置”保存该站Key，先点“检查RH配置（不生成）”，再准备/审核/开始生成。本机不执行H3采样分支，云端实际模型与采样参数以RH保存的图为准。国内/海外站Key不同；MV和口播分别需要对应的单段图，不能随意使用一个公开教程ID。源码定义的RH基地址为 `https://www.runninghub.ai` / `https://www.runninghub.cn`，不是截图中的 `https://www.zzai.app/v1`。[RH配置与分工](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/MV_RUNNINGHUB.md#L3-L17)、[基地址源码](https://github.com/Zzz1phyrf/ComfyUI-H3-LongVideo/blob/f8e1e4dfaab93d6fe318f41db48b963b44f8ee0e/mv_rh.py#L29-L41)。

扩写API只给辅助LLM使用 `/chat/completions`；它不会代替本地H3模型或RH生成后端。截图BaseURL可能是教程采用的某个扩写服务入口，但这个版本仓库没有规定必须使用它，也没有验证兼容性。

已确认以上为固定版本README、示例节点清单及源码的静态行为。没有实际启动ComfyUI、生成口播/MV、调用收费API，也没有验证截图中的API转发平台与这个版本的兼容性。音画同步、接缝质量、显存峰值、生成耗时均需真实环境测试，不能由仓库说明代替。

---

## selflift-Avatar 源码调研

核查日期：2026-11-20。固定版本：`460f88629b35f792a2ca68b219f13c1ba46e2ee3`（当前 main，提交标题 Document ComfyUI Manager installation）。仅阅读 README、全部 16 个非测试 Python 模块及示例工作流，未安装、未运行模型或仓库测试。

### 结论

这是 **ComfyUI / MiniMax H3 的实验采样器插件**，负责在采样中正确传递音视频遮罩、保留输入音频，并采用低分辨率先采样、高分辨率后采样及分块来降低资源压力。它不是独立数字人基础模型，也不是后期口型同步工具，不带模型权重。仓库明确称非官方实验项目，未做系统口型准确率和全部组合测试。[定位与限制](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/README_CN.md#L5-L7)、[不是后期口型同步](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/README_CN.md#L240-L257)。

### 节点和输入输出

| 节点 | 职责 |
| --- | --- |
| `SelfLiftAvatarH3Sampler` | 主 H3 音视频采样器，低清采样后升到目标分辨率，处理 AV latent/mask |
| `SelfLiftAvatarH3SpatialAnchoredSampler` | 独立实验 A/B 节点，空间条带分块时提供全帧低清锚定，可关闭锚定做对照 |
| `SelfLiftAvatarImageSampler` | 对兼容 rectified-flow 图像模型做渐进分辨率采样 |
| `SelfLiftAvatarH3TST` | H3 Temporal State Transport 模型补丁，输入输出均为 MODEL |

来源：[节点表](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/README_CN.md#L211-L238)、[实际注册](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/nodes.py#L793-L825)。

主采样器必需输入：`low_res_model`、正负 `CONDITIONING`、视频 `VAE`、含目标尺寸/时长和音视频内容的 `LATENT`、`SAMPLER`、`SIGMAS`、seed、CFG、分辨率切换步数、低清比例、SelfLift 修正参数与 upscaler 选项；可选接 `high_res_model` 和省显存模式。输出仍是 `LATENT`，需下游 VAE 解码和视频封装，不直接输出 MP4。[完整接口](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/nodes.py#L525-L557)。

仅接受 rectified-flow 模型及标准 Euler（`s_churn=0`）；源码对此直接校验报错，并非任意 KSampler 可替换。[校验源码](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/nodes.py#L119-L142)。

### 音频保留怎么实现

典型流程：输入音频裁剪 → H3 音频 VAE 编码 → 音频 `SolidMask=0` → 与视频 latent 合并 → 本插件采样 → 后续解码与封装。遮罩 `0=保留`、`1=生成`、中间值是软约束；音频、帧数和 FPS 必须对应同一段时间。[作者连接说明](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/README_CN.md#L240-L257)。

它不是仅在最后把音轨贴回：`native_mask_model` 克隆模型，打包各路 anchor/mask，在 `OUTER_SAMPLE` 包装器中于 H3 原生条件处理之前注入静态遮罩，并用独立的干净 anchor 替换 inpaint anchor。低清阶段仅缩小视频空间尺寸，音频流保持完整；最终对精确 mask=0 的位置恢复原始 anchor。[注入源码](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/avatar_sampling.py#L10-L44)、[低清流构建](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/nodes.py#L321-L342)、[最终恢复](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/nodes.py#L504-L511)、[恢复条件](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/avatar_masks.py#L106-L118)。

只支持静态输入遮罩，动态 `denoise_mask_function` 被移除并警告。输入 latent 本身必须含有要保留的真实内容，mask=0 不会凭空产生音频。[源码](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/avatar_sampling.py#L20-L31)、[说明](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/README_CN.md#L297-L312)。

正确保留音频不等于嘴型绝对准确。仓库自己明确要求检查画面，不能因为最终合成使用原音轨、听到原声音就认定嘴部被正确驱动。[限制](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/README_CN.md#L254-L257)。

### 本地推理、模型和 API 的区别

- 插件在本地调用 `comfy.samplers.sample` 两阶段采样。模型和 VAE 由工作流作为对象输入；外部 latent upscaler 从本地 `ComfyUI/models/latent_upscale_models/` 读取 `.pth` 或 `.safetensors`。[低清采样](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/nodes.py#L390-L399)、[高清采样](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/nodes.py#L480-L499)、[本地权重加载](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/h3_upscaler.py#L318-L351)。
- 本次静态扫描全部 16 个非测试 Python 文件和示例 JSON，没有找到 `zzai.app`、`seedance.nz`、`api_key` 或 requests/httpx/aiohttp 等 HTTP 客户端调用。其核心采样不依赖截图那两个第三方 API 地址；截图所列 API 是否由其他插件/网盘工作流使用，不能由本仓库推定。证据入口：[完整源码树](https://github.com/slmonker/selflift-Avatar/tree/460f88629b35f792a2ca68b219f13c1ba46e2ee3)、[主采样模块导入](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/nodes.py#L17-L44)。
- 安装要求已有能跑的 H3 ComfyUI 环境，自备基础模型、VAE、LoRA 及可选 upscaler。`pyproject.toml` 写 Python ≥3.10、`dependencies=[]`，表示本包未额外声明 pip 依赖，不表示无需 PyTorch/ComfyUI 等环境。[安装前提](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/README_CN.md#L186-L208)、[包元数据](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/pyproject.toml#L1-L13)。

### 省显存与硬件边界

- `spatial`：高分辨率视频空间分条带，有接缝和构图形变风险；带遮罩时仅支持视频全生成、音频全保留，音频和时间轴不按空间块切开。
- `global_stream`：保留全序列上下文，按投影/MLP/head group 分批，需要适配的 native SLA/sol-attn；缺少或遇到其他 attention override 时可回退到全序列 dense 路径。
- `global`：全上下文兼容路径，显存更高。
- `global_ram`：用主机 RAM 暂存，GPU 压力较低但更慢。

来源：[当前模式说明](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/README_CN.md#L71-L105)、[源码模式和参数](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/nodes.py#L545-L552)、[示例工作流回退条件](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/example_workflows/README.md#L28-L42)。

没有可靠的固定显卡容量门槛：分辨率、时长、模型权重、batch、参考素材和卸载策略都会影响峰值。作者明确不承诺“几块对应几 GB”、速度提升比例或避免 OOM，节点值只是估算。[硬件限制说明](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/README_CN.md#L259-L278)。

### 与其他两个插件的关系

本仓库在采样层提供 `SelfLiftAvatarH3Sampler`；没有导入 H3-LongVideo 或 YCNodes Python 模块，也没有以它们作为包依赖，因此不能称三者在 Python 安装层面互相强制依赖。H3-LongVideo 若通过 ComfyUI 节点注册表寻找本采样器，则是上层工作流组合关系，需以该仓库源码另行确认。[本包注册](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/nodes.py#L814-L825)、[本包依赖](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/pyproject.toml#L13)。

### 后续使用需要注意的实际版本差异

当前 main 已将 `model` / `model_hires` 输入改名为 `low_res_model` / `high_res_model`；旧工作流可能需要重新添加节点和接线。main 中的接口更名、全局上下文和长视频 upscaler 修复不全在旧 `v0.1.4-experimental` 发布包里，不能把“下载最新发布包”视为拥有当前主分支全部能力。[迁移说明](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/README_CN.md#L33-L69)、[upscaler 修复说明](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/README_CN.md#L9-L22)。

本文没有执行任何安装、模型下载、推理、付费 API 请求或实测性能验证。

### 下载入口与具体使用

#### 插件与示例工作流

- 仓库：[slmonker/selflift-Avatar](https://github.com/slmonker/selflift-Avatar)。
- 固定版本源码 ZIP：[460f8862 源码包](https://github.com/slmonker/selflift-Avatar/archive/460f88629b35f792a2ca68b219f13c1ba46e2ee3.zip)。此入口包含当前 main 的修复；旧 Release 包与当前 main 有差异。
- 作者已打包版本：[v0.1.4-experimental](https://github.com/slmonker/selflift-Avatar/releases/tag/v0.1.4-experimental)。
- 示例工作流 JSON：[下载固定版本 H3+selflift-avatar-fixed.json](https://raw.githubusercontent.com/slmonker/selflift-Avatar/460f88629b35f792a2ca68b219f13c1ba46e2ee3/example_workflows/H3%2Bselflift-avatar-fixed.json)。
- [示例说明](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/example_workflows/README.md)。

作者的 Git 安装方法是在 `ComfyUI/custom_nodes` 内克隆成 `selflift-Avatar` 文件夹，更新用 `git pull --ff-only`；ZIP 安装要用新目录替换旧目录，旧副本移到 `custom_nodes` 外，避免双重加载，之后重启后端并强刷前端。[安装原文](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/README_CN.md#L186-L208)。

```sh
git clone https://github.com/slmonker/selflift-Avatar.git ComfyUI/custom_nodes/selflift-Avatar
```

本调研仅列出作者命令，没有执行安装。

#### 从示例起步

1. 先准备能运行 H3 的 ComfyUI、对应基础模型、文本编码器、音视频 VAE、工作流所用 LoRA 与 latent upscaler。
2. 下载 JSON 导入 ComfyUI，补齐缺失节点并把模型下拉框改为自己实际下载的文件；把 `LoadImage` 替换为自己的参考角色图。
3. `CLIPLoader`、视频 VAE、音频 VAE、图片和提示词进入 `MiniMaxH3ReferenceToVideo`，它输出 positive 条件与目标 AV latent；positive 另分支经 `ConditioningZeroOut` 得到 negative。
4. 模型链进入 `SelfLiftAvatarH3Sampler.low_res_model`；positive、negative、视频 VAE、目标 latent、Euler 和 sigmas 分别连入采样器；high_res_model 可以不接。
5. 采样后的同一 `LATENT` 分别进 `VAEDecode` 与 `VAEDecodeAudio`，再把图像和音频接 `CreateVideo` / `VHS_VideoCombine` 合成并保存。
6. 若只验证采样结果，可先使用示例已有 `VHS_VideoCombine` 分支；示例还包括 DLSS 去噪/插帧、预览覆盖等第三方后处理，它们不是本采样器的核心输入条件。

以上接线来自 JSON 中节点 136、173、232、121、122、194、180 与 links；[固定版本工作流源码](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/example_workflows/H3%2Bselflift-avatar-fixed.json)。

**示例与“上传音频驱动数字人”的区别：**当前示例 JSON 没有 `LoadAudio` / 音频编码 / mask=0 链路，`MiniMaxH3ReferenceToVideo` 的参考音频插口也是空的，属于图片与提示词生成音画的工作流。不能声称导入这个 JSON 后仅替换音频就完成音频驱动。要用外部口播/歌曲驱动，需按 README 增加“加载/裁剪输入音频 → H3 音频 VAE 编码 → 音频 mask=0 → 合并 AV latent”的链路，再交采样器，保证 duration/frames/FPS 与音频一致；用户也可用 H3-LongVideo 提供的上层工作流完成该编排。[JSON 证据](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/example_workflows/H3%2Bselflift-avatar-fixed.json)、[音频接线规范](https://github.com/slmonker/selflift-Avatar/blob/460f88629b35f792a2ca68b219f13c1ba46e2ee3/README_CN.md#L240-L257)。

#### 模型下载来源（工作流注释列出的链接）

模型总入口：[Comfy-Org/MiniMax-H3](https://huggingface.co/Comfy-Org/MiniMax-H3)。以下是示例 JSON 的 `MarkdownNote` 节点 117 中所列的基准模型链接：

| 类型 / 放置目录 | 文件下载 |
| --- | --- |
| `models/vae/` | [minimax_h3_video_vae_fp16.safetensors](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/vae/minimax_h3_video_vae_fp16.safetensors) |
| `models/vae/` | [minimax_h3_audio_vae_fp32.safetensors](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/vae/minimax_h3_audio_vae_fp32.safetensors) |
| `models/diffusion_models/` | [minimax_h3_ref2va_pruned_int8_convrot.safetensors](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/diffusion_models/minimax_h3_ref2va_pruned_int8_convrot.safetensors) |
| `models/text_encoders/` | [qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/text_encoders/qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors) |
| `models/loras/` | [minimax_h3_ref2v_turbo_4step_v0.1_comfyui_bf16.safetensors](https://huggingface.co/Comfy-Org/MiniMax-H3/resolve/main/loras/minimax_h3_ref2v_turbo_4step_v0.1_comfyui_bf16.safetensors) |

这些链接来自工作流作者注释，未下载模型来验证可用性。**注释列出的基准模型与 JSON 各加载节点实际保存的选择不完全相同**：实际还有 hybrid H3、量化 VAE、`dmad_minimax_h3_4step_full_critic_comfy.safetensors`、`H3_speed_slider_1.1.safetensors` 等作者本地文件名。不能保证只下载上表就原样运行；应根据自己的模型组合调整加载节点与采样设置，不猜补其余权重的下载地址。

示例采样器实际保存的 latent upscaler 名称是 `minimax_h3_latent_upscaler_3d_conv_v1_bf16.safetensors`，本仓库不附该权重；插件只声明从本地 `models/latent_upscale_models/` 查找，本调研未在该仓库说明中找到它的单独下载链接，故不拼接假定地址。

---

## ComfyUI-YCNodes-MiniMax-H3：本地 H3 推理辅助节点

核对版本：`efbe83f69ffa97a251a575ce42edc7bf8005da0c`，核对日期：2026-11-20。已读取 README、初始化文件、全部 7 个 Python 节点文件及仓库示例工作流。仅静态阅读，未安装或运行。

### 职责与节点

该仓库提供 ComfyUI 内部的条件、模型补丁、噪声调度及采样节点。源码使用 `torch`、`comfy.*`、`node_helpers` 和 `comfy_api.latest`；已检查的全部 Python 节点中没有 HTTP 客户端或 API key 输入。它需要能支持 H3 的 ComfyUI 核心和本地模型，不能把“安装本节点包”等同于“模型已就绪”。[初始化加载逻辑](https://github.com/yichengup/ComfyUI-YCNodes-MiniMax-H3/blob/efbe83f69ffa97a251a575ce42edc7bf8005da0c/__init__.py#L33-L63)、[Tail 的导入与依赖](https://github.com/yichengup/ComfyUI-YCNodes-MiniMax-H3/blob/efbe83f69ffa97a251a575ce42edc7bf8005da0c/py/minimax_i2v_tail.py#L22-L30)

| 源码节点名 | 实际作用与输入输出 |
| --- | --- |
| `MiniMaxH3ImageToVideoTail` | 输入 CLIP、VAE、prompt、首末帧及可选已有 LATENT；已有 LATENT 直接透传，并重算帧数和首末帧条件；输出 CONDITIONING 与 LATENT。用于二次采样衔接，不是把生成时长无限延长的控制器。 |
| `H3PromptRelay` | 输入 MODEL、CONDITIONING、CLIP、LATENT 和以 `\|` 分隔的局部提示词；构造时间对应的注意力掩码，输出打补丁的 MODEL 与原 CONDITIONING。 |
| `H3DistanceAttentionPatcher` | 输入 MODEL 及距离/时间/sigma 等参数；创建时空距离注意力惩罚，输出 MODEL。画质改善是节点目标，本次没有运行验证。 |
| `H3DynamicCFGScheduler` | 输入 MODEL 与早晚 CFG、sigma 区间；区间内线性插值引导强度，返回 MODEL。 |
| `H3SigmaRefiner` | 输入 SIGMAS 和额外步数；重采样低噪区间的调度尾段，返回 SIGMAS。增加计算步数，并非无成本增强。 |
| `H3TiledSampler` | 输入 NOISE、GUIDER、SAMPLER、SIGMAS、LATENT；空间 H×W 分块采样与窗口融合，返回两个 LATENT。分块路径恢复原音频张量。 |
| `H3CondNoiseAug` | 向 CONDITIONING 写入视觉/音频参考加噪系数，返回 CONDITIONING。是否生效依赖下游 H3 实现识别对应字段。 |

各节点实现：[Tail](https://github.com/yichengup/ComfyUI-YCNodes-MiniMax-H3/blob/efbe83f69ffa97a251a575ce42edc7bf8005da0c/py/minimax_i2v_tail.py#L129-L208)、[Prompt Relay](https://github.com/yichengup/ComfyUI-YCNodes-MiniMax-H3/blob/efbe83f69ffa97a251a575ce42edc7bf8005da0c/py/h3_prompt_relay.py#L303-L481)、[Distance Attention](https://github.com/yichengup/ComfyUI-YCNodes-MiniMax-H3/blob/efbe83f69ffa97a251a575ce42edc7bf8005da0c/py/H3DistanceAttentionPatcher.py#L65-L194)、[Dynamic CFG](https://github.com/yichengup/ComfyUI-YCNodes-MiniMax-H3/blob/efbe83f69ffa97a251a575ce42edc7bf8005da0c/py/h3_dynamic_cfg_scheduler.py#L51-L75)、[Sigma Refiner](https://github.com/yichengup/ComfyUI-YCNodes-MiniMax-H3/blob/efbe83f69ffa97a251a575ce42edc7bf8005da0c/py/h3_sigma_refiner.py#L49-L96)、[Tiled Sampler](https://github.com/yichengup/ComfyUI-YCNodes-MiniMax-H3/blob/efbe83f69ffa97a251a575ce42edc7bf8005da0c/py/h3_tiled_sampler.py#L273-L346)、[Cond Noise Aug](https://github.com/yichengup/ComfyUI-YCNodes-MiniMax-H3/blob/efbe83f69ffa97a251a575ce42edc7bf8005da0c/py/h3_cond_noise_aug.py#L25-L62)。

### 本地模型与依赖边界

仓库示例包含 `UNETLoader` 加载 H3 int8 主模型、`CLIPLoader` 加载 `qwen3vl_32b_minimax_h3_int8_convrot.safetensors`、两个 `VAELoader` 分别加载 H3 视频与音频 VAE，还出现多个其他 H3 节点和 LoRA。说明完整示例依赖不限于这个节点包，也不能以 README“无第三方依赖，仅需 PyTorch”推断任意原版 ComfyUI 都可直接运行。这里仅核实示例所用文件名和节点类型，没有独立核验模型下载来源、最低显存或整个工作流的可运行性。[示例工作流](https://github.com/yichengup/ComfyUI-YCNodes-MiniMax-H3/blob/efbe83f69ffa97a251a575ce42edc7bf8005da0c/example/MiniMax_H3_refer%20tiled%20second%20sampling.json)

### 文档与源码差异

1. README 写 6 个节点，当前源码额外有 `H3CondNoiseAug`，初始化会扫描并加载 `py/` 下所有 Python 文件，实际定义了 7 个节点。[README 总览](https://github.com/yichengup/ComfyUI-YCNodes-MiniMax-H3/blob/efbe83f69ffa97a251a575ce42edc7bf8005da0c/README.md#L1-L20)、[新增节点](https://github.com/yichengup/ComfyUI-YCNodes-MiniMax-H3/blob/efbe83f69ffa97a251a575ce42edc7bf8005da0c/py/h3_cond_noise_aug.py#L57-L63)
2. README 的 Tiled 表格还是 `tile_axis` / `n_tiles`；当前 INPUT_TYPES 已改为 `tile_mode`、`h_tiles`、`v_tiles`、`vram_budget_frac` 等，分块沿 H×W 进行。接线与参数应依据所装版本。[当前参数](https://github.com/yichengup/ComfyUI-YCNodes-MiniMax-H3/blob/efbe83f69ffa97a251a575ce42edc7bf8005da0c/py/h3_tiled_sampler.py#L292-L346)
3. README 明确把 Tiled 用于低噪二采高清精修，提示不适合纯噪声重去噪；该节点不能代替完整首轮生成方案。[适用范围](https://github.com/yichengup/ComfyUI-YCNodes-MiniMax-H3/blob/efbe83f69ffa97a251a575ce42edc7bf8005da0c/README.md#L131-L137)
4. `H3CondNoiseAug` 自身只写条件字段，没有在该节点内直接对 latent 做混噪；调节值能否生效需要下游模型读取字段，不能仅凭控件出现就断言生效。[实际实现](https://github.com/yichengup/ComfyUI-YCNodes-MiniMax-H3/blob/efbe83f69ffa97a251a575ce42edc7bf8005da0c/py/h3_cond_noise_aug.py#L47-L54)

---

[返回下载与使用指南](plugins-guide.md) · [返回仓库目录](../../README.md)
