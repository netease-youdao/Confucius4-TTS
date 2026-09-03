<div align="center">
    <img src="https://2901733926.github.io/Confucius4-TTS/Confucius4-TTS.jpg" alt="Confucius4-TTS" width="35%">
    <h1>Confucius4-TTS: 多语种跨语种零样本TTS</h1>
    <p><b>一种音色，任意语言。</b></p>
</div>

<div align="center">
    <a href="./README.md"><img src="https://img.shields.io/badge/README-EN-red"></a>
    &nbsp;&nbsp;&nbsp;&nbsp;
    <a href="https://arxiv.org/abs/2608.11650"><img src="https://img.shields.io/badge/arXiv-2608.11650-b31b1b.svg"></a>
    &nbsp;&nbsp;&nbsp;&nbsp;
    <a href="./LICENSE"><img src="https://img.shields.io/badge/code_license-Apache%202.0-blue"></a>
    &nbsp;&nbsp;&nbsp;&nbsp;
    <a href="https://confucius4-tts.youdao.com/gradio"><img src="https://img.shields.io/badge/Demo-在线体验-purple"></a>
    &nbsp;&nbsp;&nbsp;&nbsp;
    <a href="https://2901733926.github.io/Confucius4-TTS/"><img src="https://img.shields.io/badge/GitHub.io-Demo_Page-blue?logo=GitHub&style=flat-square"></a>
    &nbsp;&nbsp;&nbsp;&nbsp;
    <a href="https://huggingface.co/netease-youdao/Confucius4-TTS"><img src="https://img.shields.io/badge/%F0%9F%A4%97%20Hugging%20Face-Model-yellow"></a>
    &nbsp;&nbsp;&nbsp;&nbsp;
    <a href="https://modelscope.cn/models/netease-youdao/Confucius4-TTS"><img src="https://img.shields.io/badge/ModelScope-Model-blue"></a>
    &nbsp;&nbsp;&nbsp;&nbsp;
</div>
<br>
Confucius4-TTS 是一款基于大语言模型（LLM）的先进文本转语音（TTS）系统，专为多语种和跨语种语音合成而设计。基于语音编码器 + 大语言模型（LLM）架构构建，能够在保持说话人音色一致的同时，实现跨语种的高质量语音生成。
在线 Demo 页面体验：[https://confucius4-tts.youdao.com/gradio]

**✨ 核心特性**

- **支持 14 种语言**：中文、英文、日语、韩语、德语、法语、西班牙语、印尼语、意大利语、泰语、葡萄牙语、俄语、马来语、越南语 *（更多语言即将推出）*
- **无约束声音克隆**：无需参考文本
- **跨语种声音迁移**：跨 14 种语言的无口音语音合成
- **零样本声音迁移**：无需额外训练即可克隆声音
- **无缝情感迁移**：克隆情感，而非仅仅是声音
- **强泛化能力**：在真实多语种场景中表现稳定

凭借强大的跨语种泛化能力，Confucius4-TTS 允许用户在保持相同音色的同时无缝切换语言，提供流畅、自然且富有表现力的语音。

## Contents

- [环境安装](#-环境安装)
- [推理](#-推理)
- [训练](#-训练)
- [性能](#-性能)
- [引用](#引用)

## 🛠 环境安装

### 环境要求

- Python 3.10
- CUDA 12.6

### 安装步骤

1. 克隆仓库：

```bash
git clone https://github.com/netease-youdao/Confucius4-TTS.git
cd Confucius4-TTS
```

2. 创建并激活 conda 环境：

```bash
conda create -n confuciustts python=3.10 -y
conda activate confuciustts
```

3. 安装依赖：

```bash
pip install -r requirements.txt
```

## 🚀 推理

对于访问 HuggingFace 受限的环境，运行前可设置镜像端点：

```bash
export HF_ENDPOINT=https://hf-mirror.com
```

### 基础用法

使用提供的 `example.py` 脚本进行 zero-shot TTS 合成：

```bash
python example.py \
    --prompt_wav path/to/reference.wav \
    --text "要合成的文本" \
    --lang zh \
    --out output.wav \
    --config config/inference_config.yaml
```

也可以直接使用 Python API：

```python
import torch
import torchaudio
from confuciustts.cli.inference import ConfuciusTTS

model = ConfuciusTTS(
    config_path="config/inference_config.yaml",
    device="cuda" if torch.cuda.is_available() else "cpu",
)

audio = model.generate(
    text="你好，欢迎使用 Confucius4-TTS。",
    lang="zh",
    prompt_wav="path/to/reference.wav",
    verbose=True,
)

torchaudio.save("output.wav", audio.cpu(), model.sample_rate)
```

### vLLM 用法

上面的基础路径使用 HuggingFace Transformers 运行 Text2Semantic（T2S）自回归阶段。如需更快的 T2S 生成，可切换到 **vLLM** 后端（`example_vllm.py`），通过 PagedAttention 加速 LLM 阶段。

当前支持 **vLLM 0.16.0（V1 engine）**。更早的 vLLM 版本未经过测试，可能无法运行，因为模型注册和 `GPUModelRunner` 的 monkey-patch 针对的是 v1 engine 内部实现。

> ⚠️ `vllm` 对 GPU 架构 / 驱动 / CUDA 有特定要求，可能无法在你的硬件上运行；直接装进基础环境可能破坏其它依赖。建议从基础环境**克隆一个独立的 conda 环境**，只安装 vLLM 的附加依赖。

```bash
# 1. 从基础环境克隆一个专用环境（继承 confuciustts 的依赖）
conda create -n confuciustts_vllm --clone confuciustts
conda activate confuciustts_vllm

# 2. 安装 vLLM 附加依赖
pip install -r requirements_vllm_add.txt

# 3. 运行 vLLM 示例（模型在首次运行时自动下载）
python example_vllm.py \
    --prompt_wav path/to/reference.wav \
    --text "要合成的文本" \
    --lang zh \
    --out output_vllm.wav
```

通过 `--stream` 参数同时支持非流式与流式生成：

```bash
# 非流式：合成整段语音并保存为一个 .wav
python example_vllm.py \
    --prompt_wav path/to/reference.wav \
    --text "要合成的文本" \
    --lang zh \
    --out output_vllm.wav

# 流式：逐块生成（model.generate_stream），此处拼接为一个 .wav
python example_vllm.py \
    --prompt_wav path/to/reference.wav \
    --text "要合成的文本" \
    --lang zh \
    --out output_vllm_stream.wav \
    --stream
```

### Web Demo

提供了一个 Gradio 网页界面，可在浏览器中进行交互式 zero-shot 声音克隆：上传参考音频、输入文本、选择语言并点击生成。

```bash
python webui.py --port 7860
```

然后在浏览器打开 `http://<服务器IP>:7860`。参考音频通过 HTTP 上传到服务器，因此客户端浏览器无需与服务器共享文件系统。

### 在线服务

如需程序化调用（例如从另一台服务/机器调用 TTS），可使用 FastAPI 服务。它同时提供非流式与流式接口，并以 HTTP 文件上传方式接收参考音频，因此客户端与服务器无需在同一台机器上。

```bash
python server.py --port 8000
```

| 接口 | 方法 | 说明 |
|---|---|---|
| `/api/tts` | POST（multipart） | 非流式：返回完整的 `.wav`（PCM 16-bit） |
| `/api/tts/stream` | POST（multipart） | 流式：原始 int16-LE PCM 分块（采样率见 `X-Sample-Rate` 头） |

请求表单字段：`text`、`lang`（如 `zh`、`en`、`ja`）、`reference`（音频文件上传）。

```bash
# 非流式 → out.wav
curl -F "text=要合成的文本" -F "lang=zh" -F "reference=@path/to/reference.wav" \
     http://localhost:8000/api/tts -o out.wav

# 流式 → 原始 int16 PCM（22050 Hz，单声道），按 X-Sample-Rate 头播放
curl -F "text=要合成的文本" -F "lang=zh" -F "reference=@path/to/reference.wav" \
     http://localhost:8000/api/tts/stream -o out.pcm
```

## 🚀 微调

Confucius4-TTS 采用「语音编码器 + LLM」架构，训练流程涵盖以下两个模块：
- **Text2Semantic（T2S）**：根据文本与说话人条件生成语义 token 序列。
- **Semantic2Acoustic（S2A）**：流匹配模型，将语义 token 转换为梅尔频谱图。

### 1. 准备预训练模型

下载两个外部模型：

```bash
# Wav2Vec2-BERT（说话人条件化 & 语义特征提取）
huggingface-cli download facebook/w2v-bert-2.0 \
    --local-dir pretrained/w2v-bert-2.0

# Amphion MaskGCT（语义编解码器实现）
git clone https://github.com/open-mmlab/Amphion.git external/Amphion
```

下载完成后，目录结构如下：

```
checkpoints/
├── t2s_model.safetensors        # T2S 预训练权重
├── s2a_model.pt                 # S2A 预训练权重
├── wav2vec2bert_stats.pt        # 语义特征归一化统计量
├── special_tokens_map.json      # 分词器文件
├── tokenizer.json
├── tokenizer.model
└── tokenizer_config.json
pretrained/
├── w2v-bert-2.0/                # Wav2Vec2-BERT 模型
└── campplus/
    └── campplus_cn_common.bin   # CAMPPlus 说话人编码器权重
external/
└── Amphion/                     # MaskGCT 语义编解码器实现
```

### 2. 准备训练数据

训练数据为 **TSV 文件**（制表符分隔），不含表头，包含以下 5 列：

| 列名 | 说明 |
|---|---|
| `lang` | 语言代码（如 `zh`、`en`、`ja`） |
| `wav_path` | 目标音频路径 |
| `norm_text` | 归一化后的文本 |
| `semantic_ids_path` | 预提取的语义 token（`.npy` 文件路径） |
| `ref_audio_paths` | 参考音频路径，支持多个用逗号分隔 |

在 `config/train_t2s.yaml` 中配置训练/验证集路径：

```yaml
data:
  train_data_path:
    - data/train.tsv
  val_data_path:
    - data/val.tsv
```

### 3. 启动 T2S 训练

在 `config/train_t2s.yaml` 中设置预训练 T2S 权重路径：

```yaml
paths:
  t2s_checkpoint: checkpoints/t2s_model.safetensors
```

**单机训练：**

```bash
python -m confuciustts.cli.train_t2s -c config/train_t2s.yaml
```

### 4. 启动 S2A 训练

在 `config/train_s2a.yaml` 中设置权重路径。`t2s_checkpoint` 指向冻结的 T2S 骨干网络；`s2a_checkpoint` 为可选项，用于从预训练 S2A 模型继续训练：

```yaml
paths:
  t2s_checkpoint: checkpoints/t2s_model.safetensors
  s2a_checkpoint: checkpoints/s2a_model.pt   # 可选：从预训练 S2A 权重继续训练
```

**单机训练：**

```bash
python -m confuciustts.cli.train_s2a -c config/train_s2a.yaml
```

S2A 训练过程中，T2S 模型、说话人编码器（Wav2Vec2-BERT）和风格编码器（CAMPPlus）均处于冻结状态，只有流匹配 S2A 模型参与训练。

## 📊 性能

Confucius4-TTS 在多语种及跨语种零样本 TTS 基准测试中表现优异，兼具高可懂度与说话人相似度。

> WER/CER 越低越好（↓），SIM 越高越好（↑）。

### CV3-eval 跨语种

<details>
<summary><b>CV3-eval 跨语种结果（点击展开）</b></summary>

| Direction | Metric | Confucius4-TTS | CosyVoice2† | CosyVoice3-0.5B† | CosyVoice3-1.5B† | OmniVoice† | VoxCPM2 |
|---|---|---:|---:|---:|---:|---:|---:|
| en→zh | CER↓ | **6.16** | 13.50 | 8.48 | 8.01 | 6.53 | 6.29 |
| ja→zh | CER↓ | 4.87 | 48.10 | 6.86 | 6.78 | 52.64 | **4.20** |
| ko→zh | CER↓ | 1.28 | 7.70 | 5.24 | 3.30 | 1.71 | **1.20** |
| zh→en | WER↓ | **3.19** | 17.10 | 6.83 | 5.39 | 3.72 | 3.84 |
| ja→en | WER↓ | **3.44** | 11.20 | 5.86 | 5.94 | 5.25 | 4.10 |
| ko→en | WER↓ | **3.42** | 13.10 | 18.30 | 13.70 | 3.91 | 5.69 |

† 需要参考文本。

</details>

### X-Voice Benchmark

<details>
<summary><b>X-Voice 跨语种结果（点击展开）</b></summary>

| Direction | Metric | Confucius4-TTS | X-Voice | IndexTTS2 | OmniVoice† | VoxCPM2 |
|---|---|---:|---:|---:|---:|---:|
| de→zh | CER↓ | **2.86** | 3.07 | 3.46 | 7.79 | 3.62 |
| en→zh | CER↓ | 3.21 | **3.06** | 3.78 | 3.30 | 3.35 |
| fr→zh | CER↓ | **2.70** | 3.01 | 3.53 | 8.16 | 3.75 |
| ja→zh | CER↓ | 3.50 | **3.39** | 4.11 | 60.88 | 4.53 |
| ko→zh | CER↓ | **2.86** | 3.13 | 2.90 | 7.35 | 6.33 |
| th→zh | CER↓ | 2.82 | **2.79** | 3.08 | 2.85 | 5.96 |
| vi→zh | CER↓ | **2.75** | 2.78 | 2.98 | 6.59 | 3.65 |

† 需要参考文本。

</details>

### Seed-TTS-eval

<details>
<summary><b>Seed-TTS-eval 中英文测试集结果（点击展开）</b></summary>

| System | English WER↓ | English SIM↑ | Chinese CER↓ | Chinese SIM↑ |
|---|---:|---:|---:|---:|
| Confucius4-TTS | 1.49 | 0.700 | 0.94 | 0.765 |
| Confucius4-TTS (Continuation)† | 1.68 | 0.715 | 1.15 | 0.766 |
| Seed-TTS† | 2.25 | **0.762** | 1.12 | **0.796** |
| Qwen3-TTS† | **1.24** | 0.714 | **0.77** | 0.770 |
| FishAudio S2† | 1.79 | 0.643 | 0.98 | 0.737 |
| OmniVoice† | 1.62 | 0.740 | 0.87 | 0.777 |
| VoxCPM2† | 1.70 | 0.752 | 0.97 | 0.793 |
| X-Voice | 1.91 | 0.627 | 1.47 | 0.746 |

† 需要参考文本。

</details>

### MiniMax-MLS-Test

<details>
<summary><b>MiniMax-MLS-Test 结果（点击展开）</b></summary>

| Language | Metric | Confucius4-TTS | Confucius4-TTS (Continuation)† | MiniMax-Speech | ElevenLabs | Qwen3-TTS† | FishAudio S2† | OmniVoice† | VoxCPM2† |
|---|---|---:|---:|---:|---:|---:|---:|---:|---:|
| German | WER↓ | **0.47** | 0.68 | 1.91 | 0.57 | 1.24 | 0.55 | 0.80 | 1.12 |
|  | SIM↑ | 0.775 | 0.777 | 0.733 | 0.614 | 0.768 | 0.706 | 0.804 | **0.805** |
| French | WER↓ | 3.66 | 4.87 | 4.10 | 5.22 | **2.86** | 3.90 | 3.58 | 3.42 |
|  | SIM↑ | 0.723 | 0.755 | 0.628 | 0.535 | 0.716 | 0.658 | **0.776** | 0.738 |
| Indonesian | WER↓ | 1.12 | 1.41 | 1.24 | **1.06** | – | 2.93 | 1.34 | 1.17 |
|  | SIM↑ | 0.765 | 0.767 | 0.729 | 0.660 | – | 0.736 | 0.777 | **0.795** |
| Korean | CER↓ | 1.84 | 2.50 | 1.75 | 1.87 | 1.76 | **1.62** | 2.66 | 3.34 |
|  | SIM↑ | 0.812 | 0.824 | 0.776 | 0.700 | 0.790 | 0.742 | 0.831 | **0.837** |
| Thai | WER↓ | **1.56** | 2.47 | 2.70 | 73.94 | – | 6.66 | 2.93 | 2.19 |
|  | SIM↑ | 0.773 | 0.807 | 0.800 | 0.588 | – | 0.749 | **0.847** | 0.841 |
| Japanese | CER↓ | 4.14 | 4.05 | 3.52 | 10.65 | 3.82 | 3.52 | 3.59 | **3.51** |
|  | SIM↑ | 0.788 | 0.806 | 0.776 | 0.738 | 0.771 | 0.753 | 0.821 | **0.825** |
| Vietnamese | WER↓ | 1.61 | 1.59 | **0.88** | 73.42 | – | 14.11 | 0.95 | 4.19 |
|  | SIM↑ | 0.751 | 0.753 | 0.743 | 0.369 | – | 0.693 | 0.775 | **0.793** |
| Italian | WER↓ | 1.30 | 3.26 | 1.54 | 1.74 | **0.95** | 1.49 | 1.20 | 1.34 |
|  | SIM↑ | 0.787 | 0.791 | 0.699 | 0.579 | 0.752 | 0.764 | **0.813** | 0.779 |
| Portuguese | WER↓ | 2.48 | 3.91 | 1.88 | **1.33** | 1.53 | 1.57 | 1.83 | 1.71 |
|  | SIM↑ | 0.796 | 0.801 | 0.805 | 0.711 | 0.805 | 0.777 | **0.866** | 0.842 |
| Spanish | WER↓ | 1.02 | 1.65 | 1.03 | 1.08 | 1.13 | 0.95 | **0.81** | 1.32 |
|  | SIM↑ | 0.778 | 0.794 | 0.762 | 0.615 | 0.814 | 0.734 | 0.814 | **0.829** |
| Russian | WER↓ | 4.64 | 5.42 | 4.28 | 3.88 | **3.21** | 4.24 | 4.63 | 4.53 |
|  | SIM↑ | 0.787 | 0.796 | 0.761 | 0.675 | 0.784 | 0.768 | 0.784 | **0.807** |

† 需要参考文本。

</details>

---

## 🤝 商务合作

如有 Confucius4-TTS 相关的商业合作、产品集成或其他商务需求，欢迎联系我们：

- **联系电话：** 010-82558901
- **商务邮箱：** AIcloud_Business@corp.youdao.com

---

## 致谢

Confucius4-TTS 基于以下开源项目构建：

- **[Qwen3-TTS](https://github.com/QwenLM/Qwen3-TTS)** — 说话人编码器（ECAPA-TDNN）及文本嵌入投影层架构
- **[CosyVoice](https://github.com/FunAudioLLM/CosyVoice)** — 文本归一化流程
- **[Amphion / MaskGCT](https://github.com/open-mmlab/Amphion)** — 语义编解码器实现
- **[w2v-BERT 2.0](https://huggingface.co/facebook/w2v-bert-2.0)** — 语义特征提取与说话人条件化
- **[Seed-VC](https://github.com/Plachtaa/seed-vc)** — Flow matching 架构参考
- **[BigVGAN](https://github.com/NVIDIA/BigVGAN)** — 高保真神经声码器，用于梅尔频谱图到波形的合成

---

## 引用

如果您在研究或项目中使用了 Confucius4-TTS，请考虑引用：

```bibtex
@misc{wang2026confucius4tts,
  title         = {Confucius4-TTS: Transcript-Free Cross-Lingual Zero-Shot TTS with a Learnable Speaker Encoder},
  author        = {Huaxuan Wang and Huimin Wang and Ruiyu Zhang and Yingjie Li and Yitao Duan},
  year          = {2026},
  eprint        = {2608.11650},
  archivePrefix = {arXiv},
  primaryClass  = {cs.SD},
  url           = {https://arxiv.org/abs/2608.11650}
}
```
