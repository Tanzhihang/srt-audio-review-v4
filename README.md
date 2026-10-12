# SRT Audio Review V4

**把字幕校稿推进到语义与声音：错字、术语、断句、时间定位，一套完整流程。**

由 **[谭智航 · Tanzhihang](https://github.com/Tanzhihang)** 制作并开源。

**[下载完整源码 · v0.1.0](srt-audio-review-v4-v0.1.0.zip)** · 包含 48 个源码、规则、词库、文档与测试文件，下载后解压安装。

[English](README.en.md) · [工作流与命令（源码包内）](srt-audio-review-v4-v0.1.0.zip) · [贡献指南](CONTRIBUTING.md) · [MIT License](LICENSE)

自动字幕常见的问题，远不止一个错别字：术语被拆开，重复词跨了两条字幕，完整词被切成两半，改好了文字却对不上声音。SRT Audio Review V4 为这些问题建立了一条可追溯的处理链——从全文校对，到逐边界语义复核，再到原音定位、整组写入和最终验收。

这是面向 Codex 等支持 Agent Skills 的本地工作流，包含规则、领域词库和 Python 工具。语义判断由执行 Skill 的智能体完成，脚本负责候选召回、证据整理和结构校验；单独运行扫描脚本不会自动完成全部校稿。

## 六项核心能力

| 能力 | 实际处理方式 |
|---|---|
| **原音驱动的听写校准** | 本地 Whisper 全量转写，生成逐词时间戳；疑难词和断句组复用同一份音频证据 |
| **逐边界语义复核** | 逐一检查全部 `N−1` 个相邻字幕边界，覆盖拆词、定中分离、动宾错位、术语断裂和重复启动 |
| **连续语义组修复** | 用 B 编号组织连续字幕块，链式问题合并成完整组，整组修改并防止重叠写入 |
| **文字与声音对齐** | 每个重新断句组都记录原音定位依据，仅调整组内时间，保持组首开始和组尾结束 |
| **多重交付闸门** | 结构、跨块候选、残留口水词、字符守恒、音频定位证据、逐边界台账分别校验 |
| **可维护的术语系统** | 通用、手机数码、汽车词库按主题加载；自定义词优先，常用词保护清单防止误改 |

## 为什么断句需要单独处理

下面是一个示意例子：

```text
修改前
[1] 一期视频告诉你这个原
[2] 厂升顶帐篷

语义重排后
[1] 一期视频告诉你
[2] 这个原厂升顶帐篷
```

“原厂”应完整归属后一条字幕。V4 会先完成文字校准，再冻结底稿；断句阶段只移动已有字符与空格，去掉空白后整篇字符序列必须一致。同时根据原音重定组内切换时间。上面仅展示文字关系，具体时间必须从对应音频获取。

## 安装与使用

需要 Python **3.10+**、`ffmpeg`（含 `ffprobe`），以及能运行本地脚本的 Skill 执行环境。默认 Whisper `small`、CPU、逐词时间戳；首次安装会下载依赖和模型，模型权重不包含在仓库中。

### 1. 获取项目

点击 [完整源码包](srt-audio-review-v4-v0.1.0.zip)，在文件页面下载 `srt-audio-review-v4-v0.1.0.zip`，解压后进入项目目录。源码 ZIP 包含下文列出的全部规则、脚本、词库、文档和测试。

```bash
unzip srt-audio-review-v4-v0.1.0.zip
cd srt-audio-review-v4
```

### 2. 安装 Skill

将 `skills/srt-audio-review-v4` 整个目录复制到你的 Skill 目录。当前 Codex 官方说明使用 `~/.agents/skills/`，也支持项目内 `.agents/skills/`。旧环境若使用其他目录，请以该环境的配置为准，避免同时装入同名副本。[官方说明](https://learn.chatgpt.com/docs/build-skills)

```bash
mkdir -p ~/.agents/skills
cp -R skills/srt-audio-review-v4 ~/.agents/skills/
```

仓库根目录也提供标准 `plugin.json`，可用于支持该格式的插件加载流程；此仓库发布不代表已上架插件商店。[打包规范](https://developers.openai.com/plugins/build/plugins)

### 3. 初始化本地音频环境

确认允许下载依赖和模型后运行：

```bash
python3 skills/srt-audio-review-v4/scripts/setup_local_whisper.py
```

安装后，Python 依赖和模型默认放在 `~/.cache/srt-audio-review-v4/`。可用 `SRT_V4_CACHE` 指定其他缓存目录。初始化与转写应使用同一套 Python 环境，避免复用不兼容的二进制依赖。

### 4. 给智能体的使用提示

```text
使用 $srt-audio-review-v4 校对这份 SRT，对应原音频也已提供。
主题是汽车。保留我指定的车型和术语写法。
完成全文校对、全部相邻边界审查、重新断句组的原音定位和所有最终验收。
输出新的 SRT 和修改、验收报告。
```

默认直接交付验收通过的新文件；遇到会改变原意且证据无法消除的歧义时提问。需要逐项确认时，在提示中加上“先给我待确认清单”。

## 处理链与交付物

```text
原始 SRT + 原音频
       ↓
只读基线、规则加载、全文候选扫描
       ↓
本地 Whisper + 听写/术语/口水词校准
       ↓
冻结音频校准底稿
       ↓
逐边界复核 → B 组重排 → 原音定位
       ↓
整组原子写入 → 四项最终验收
       ↓
新 SRT + 修改记录 + 验收报告 + 边界台账
```

交付保留字幕块数、编号、内部行数、换行风格和末尾空行结构。未进入 B 组的时间码保持不变。完整运行步骤和关键命令见 [完整源码包](srt-audio-review-v4-v0.1.0.zip) 中的 `docs/WORKFLOW.md`。

## 音频、隐私与能力边界

- 转写在本机执行，不调用外部语音转写 API，不上传原音频。首次初始化需要联网下载软件和模型。
- 使用云端智能体执行文字校稿时，字幕文本或转写文本可能进入该平台的上下文；“本地转写”不代表整个智能体工作流离线。
- Whisper 输出和规则扫描均是辅助证据。工具无法保证零误识别或证明语义判断正确，全部边界仍须实际阅读复核。
- 领域词库是可修改的写法偏好与候选映射，并非实时产品数据库；部分别名本身也成立，必须结合上下文，不能整库机械替换。
- 默认 CPU 全量转写，耗时取决于音频长度、硬件与模型。未提供性能或准确率基准。

## 项目结构

```text
srt-audio-review-v4/
├── README.md / README.en.md
├── LICENSE / CONTRIBUTING.md / CHANGELOG.md
├── plugin.json
├── docs/WORKFLOW.md
├── examples/demo.srt
└── skills/srt-audio-review-v4/
    ├── SKILL.md
    ├── agents/openai.yaml
    ├── references/     # 校对规则、领域词库、数据契约
    ├── scripts/        # 扫描、转写、组写入、验收和报告工具
    └── tests/          # 断句、原子写入及验收回归测试
```

## 开发与验证

```bash
python3 -m unittest discover -s skills/srt-audio-review-v4/tests -v
```

测试覆盖跨块候选、连续组合并、冲突拒绝、音频证据缺失、原文不匹配、内部时间调整和文字守恒等约束。测试不下载模型，也不能替代真实音频的效果评估。`examples/demo.srt` 为专门编写的演示字幕，含待校对问题，不包含真实客户素材或配套原音。

## 作者与开源

**制作 / 工作流设计：[谭智航 · Tanzhihang](https://github.com/Tanzhihang)**

本项目使用 [MIT License](LICENSE)。本地语音识别依赖 [OpenAI Whisper](https://github.com/openai/whisper)，Whisper 与其依赖遵循各自许可证。欢迎提交可公开、已脱敏的字幕案例、词库改进和脚本修复。
