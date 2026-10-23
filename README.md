<div align="center">
    <img src="https://2901733926.github.io/Confucius4-TTS/Confucius4-TTS.jpg" alt="Confucius4-TTS" width="35%">
    <h1>Confucius4-TTS: a Multilingual and Cross-Lingual Zero-Shot TTS Engine</h1>
    <p><b>One voice. Any language.</b></p>
</div>

<div align="center">
    <a href="./README.zh.md"><img src="https://img.shields.io/badge/README-中文版本-red"></a>
    &nbsp;&nbsp;&nbsp;&nbsp;
    <a href="https://arxiv.org/abs/2608.11650"><img src="https://img.shields.io/badge/arXiv-2608.11650-b31b1b.svg"></a>
    &nbsp;&nbsp;&nbsp;&nbsp;
    <a href="./LICENSE"><img src="https://img.shields.io/badge/code_license-Apache%202.0-blue"></a>
    &nbsp;&nbsp;&nbsp;&nbsp;
    <a href="https://huggingface.co/spaces/CharlesW/Confucius4-TTS"><img src="https://img.shields.io/badge/Demo-在线体验-orange"></a>
    &nbsp;&nbsp;&nbsp;&nbsp;
    <a href="https://2901733926.github.io/Confucius4-TTS/"><img src="https://img.shields.io/badge/GitHub.io-Demo_Page-blue?logo=GitHub&style=flat-square"></a>
    &nbsp;&nbsp;&nbsp;&nbsp;
    <a href="https://huggingface.co/netease-youdao/Confucius4-TTS"><img src="https://img.shields.io/badge/%F0%9F%A4%97%20Hugging%20Face-Confucius4TTS-yellow"></a>
    &nbsp;&nbsp;&nbsp;&nbsp;
    <a href="https://modelscope.cn/models/netease-youdao/Confucius4-TTS"><img src="https://img.shields.io/badge/ModelScope-Confucius4TTS-purple"></a>
    &nbsp;&nbsp;&nbsp;&nbsp;
</div>
<br>

Confucius4-TTS is an advanced LLM-based text-to-speech (TTS) system designed for multilingual and cross-lingual speech synthesis. Built on a speech encoder + large language model (LLM) architecture, Confucius4-TTS enables high-quality speech generation while preserving speaker identity across languages. You can try our online demo at **[https://confucius4-tts.youdao.com/gradio](https://confucius4-tts.youdao.com/gradio)**.

**✨ Key Features**

- **14 Languages Supported**: Chinese, English, Japanese, Korean, German, French, Spanish, Indonesian, Italian, Thai, Portuguese, Russian, Malay and Vietnamese *(more coming soon)*
- **Unconstrained Voice Cloning**: No reference transcript required
- **Cross-Lingual Voice Transfer**: Unaccented speech synthesis across 14 languages
- **Zero-Shot Voice Transfer**: Clone voices without additional training
- **Seamless Emotion Transfer**: Clone the feeling, not just the voice
- **Robust Generalization**: Stable performance in real-world multilingual scenarios

With strong cross-lingual generalization, Confucius4-TTS allows users to seamlessly switch languages while keeping the same voice, delivering fluent, natural, and expressive speech.

<div align="center">

Video Demo

<table border="0">
  <tr>
    <td>
      <video src="https://github.com/user-attachments/assets/2e2a4fc2-c8ef-4f12-89f7-55a6221200f1" controls width="100%"></video>
    </td>
    <td>
      <video src="https://github.com/user-attachments/assets/dacd356d-3bf5-4b06-9a2c-6ad5c24eb035" controls width="100%"></video>
    </td>
  </tr>
  <tr>
    <td>
      <video src="https://github.com/user-attachments/assets/e00ae5e1-fbb0-4137-af13-dd53599196a5" controls width="100%"></video>
    </td>
    <td>
      <video src="https://github.com/user-attachments/assets/f6c1aabb-4258-40ba-b945-81c3eeed67c3" controls width="100%"></video>
    </td>
  </tr>
</table>
</div>

## Contents

- [Installation](#-installation)
- [Inference](#-inference)
- [Training](#-training)
- [Performance](#-performance)
- [Citation](#citation)

## 🛠 Installation

### Requirements

- Python 3.10
- CUDA 12.6

### Setup

1. Clone the repository:

```bash
git clone https://github.com/netease-youdao/Confucius4-TTS.git
cd Confucius4-TTS
```

2. Create and activate a conda environment:

```bash
conda create -n confuciustts python=3.10 -y
conda activate confuciustts
```

3. Install dependencies:

```bash
pip install -r requirements.txt
```

## 🚀 Inference

For environments with restricted access to HuggingFace, set a mirror endpoint before running:

```bash
export HF_ENDPOINT=https://hf-mirror.com
```

### Basic Usage

Use the provided `example.py` script for zero-shot TTS synthesis:

```bash
python example.py \
    --prompt_wav path/to/reference.wav \
    --text "Hello, this is a test of zero-shot voice cloning." \
    --lang en \
    --out output.wav \
    --config config/inference_config.yaml
```

You can also use the Python API directly:

```python
import torch
import torchaudio
from confuciustts.cli.inference import ConfuciusTTS

model = ConfuciusTTS(
    config_path="config/inference_config.yaml",
    device="cuda" if torch.cuda.is_available() else "cpu",
)

audio = model.generate(
    text="Hello, welcome to Confucius4-TTS.",
    lang="en",
    prompt_wav="path/to/reference.wav",
    verbose=True,
)

torchaudio.save("output.wav", audio.cpu(), model.sample_rate)
```

### vLLM Usage

The basic path above runs the Text2Semantic (T2S) autoregressive stage with HuggingFace Transformers. For faster T2S generation you can switch to the **vLLM** backend (`example_vllm.py`), which accelerates the LLM stage with PagedAttention.

We currently support **vLLM 0.16.0 (V1 engine)**. Older vLLM versions have not been tested and may not work, because the model registration and `GPUModelRunner` monkey-patches target the v1 engine internals.

> ⚠️ `vllm` has specific requirements on GPU architecture / driver / CUDA. It may not run on your hardware, and installing it into the base env can break other packages. We recommend creating a **separate conda environment** cloned from the basic one, then installing only the vLLM add-on dependencies.

```bash
# 1. Clone a dedicated env from the basic env (so confuciustts deps are inherited)
conda create -n confuciustts_vllm --clone confuciustts
conda activate confuciustts_vllm

# 2. Install the vLLM add-on dependencies
pip install -r requirements_vllm_add.txt

# 3. Run the vLLM example (models auto-download on first run)
python example_vllm.py \
    --prompt_wav path/to/reference.wav \
    --text "要合成的文本" \
    --lang zh \
    --out output_vllm.wav
```

Both non-streaming and streaming generation are supported via the `--stream` flag:

```bash
# Non-streaming: synthesize the whole utterance and save one .wav
python example_vllm.py \
    --prompt_wav path/to/reference.wav \
    --text "要合成的文本" \
    --lang zh \
    --out output_vllm.wav

# Streaming: generate chunk-by-chunk (model.generate_stream), concatenated into one .wav here
python example_vllm.py \
    --prompt_wav path/to/reference.wav \
    --text "要合成的文本" \
    --lang zh \
    --out output_vllm_stream.wav \
    --stream
```

### Web Demo

A Gradio web UI is provided for interactive zero-shot voice cloning in the browser. Upload a reference audio, type the text, pick a language, and click Generate.

```bash
python webui.py --port 7860
```

Then open `http://<server-ip>:7860` in your browser. The reference audio is uploaded to the server via HTTP, so the client browser does not need to share a filesystem with the server.

### Online Server

For programmatic access (e.g. calling TTS from another service/machine), use the FastAPI server. It exposes both non-streaming and streaming endpoints and accepts the reference audio as an HTTP file upload, so the client and server do not need to be on the same host.

```bash
python server.py --port 8000
```

| Endpoint | Method | Description |
|---|---|---|
| `/api/tts` | POST (multipart) | Non-streaming: returns a complete `.wav` (PCM 16-bit) |
| `/api/tts/stream` | POST (multipart) | Streaming: raw int16-LE PCM chunks (sample rate in `X-Sample-Rate` header) |

Request form fields: `text`, `lang` (e.g. `zh`, `en`, `ja`), `reference` (audio file upload).

```bash
# Non-streaming → out.wav
curl -F "text=要合成的文本" -F "lang=zh" -F "reference=@path/to/reference.wav" \
     http://localhost:8000/api/tts -o out.wav

# Streaming → raw int16 PCM (22050 Hz, mono), playable per X-Sample-Rate header
curl -F "text=要合成的文本" -F "lang=zh" -F "reference=@path/to/reference.wav" \
     http://localhost:8000/api/tts/stream -o out.pcm
```

## 🚀 Fine-Tuning

Confucius4-TTS follows a "speech encoder + LLM" architecture. The training pipeline covers two modules:
- **Text2Semantic (T2S)**: generates semantic token sequences from text and speaker conditioning.
- **Semantic2Acoustic (S2A)**: a flow-matching model that converts semantic tokens into mel spectrograms.

### 1. Prepare Pretrained Models

Download the two external models:

```bash
# Wav2Vec2-BERT (speaker conditioning & semantic feature extraction)
huggingface-cli download facebook/w2v-bert-2.0 \
    --local-dir pretrained/w2v-bert-2.0

# Amphion MaskGCT (semantic codec implementation)
git clone https://github.com/open-mmlab/Amphion.git external/Amphion
```

After downloading, your directory should look like:

```
checkpoints/
├── t2s_model.safetensors        # pretrained T2S weights
├── s2a_model.pt                 # pretrained S2A weights
├── wav2vec2bert_stats.pt        # semantic feature normalization statistics
├── special_tokens_map.json      # tokenizer files
├── tokenizer.json
├── tokenizer.model
└── tokenizer_config.json
pretrained/
├── w2v-bert-2.0/                # Wav2Vec2-BERT model
└── campplus/
    └── campplus_cn_common.bin   # CAMPPlus speaker encoder checkpoint
external/
└── Amphion/                     # MaskGCT semantic codec implementation
```

### 2. Prepare Training Data

Training data is provided as **TSV files** (tab-separated, no header) with the following 5 columns:

| Column | Description |
|---|---|
| `lang` | Language code (e.g. `zh`, `en`, `ja`) |
| `wav_path` | Path to the target audio |
| `norm_text` | Normalized text |
| `semantic_ids_path` | Pre-extracted semantic tokens (`.npy` file path) |
| `ref_audio_paths` | Reference audio path(s), comma-separated for multiple |

Configure the train/validation paths in `config/train_t2s.yaml`:

```yaml
data:
  train_data_path:
    - data/train.tsv
  val_data_path:
    - data/val.tsv
```

### 3. Launch T2S Training

Set the pretrained T2S checkpoint path in `config/train_t2s.yaml`:

```yaml
paths:
  t2s_checkpoint: checkpoints/t2s_model.safetensors
```

**Single-node training:**

```bash
python -m confuciustts.cli.train_t2s -c config/train_t2s.yaml
```

### 4. Launch S2A Training

Set the checkpoint paths in `config/train_s2a.yaml`. `t2s_checkpoint` points to the frozen T2S backbone; `s2a_checkpoint` is optional and can be used to resume from a pretrained S2A model:

```yaml
paths:
  t2s_checkpoint: checkpoints/t2s_model.safetensors
  s2a_checkpoint: checkpoints/s2a_model.pt   # optional: resume from pretrained S2A
```

**Single-node training:**

```bash
python -m confuciustts.cli.train_s2a -c config/train_s2a.yaml
```

During S2A training, the T2S model, speaker encoder (Wav2Vec2-BERT), and style encoder (CAMPPlus) are all frozen. Only the flow-matching S2A model is trained.

## 📊 Performance

Confucius4-TTS achieves competitive results on multilingual and cross-lingual zero-shot TTS benchmarks, with strong intelligibility and speaker similarity across multiple languages.

> Lower is better for WER/CER (↓), and higher is better for SIM (↑).

### CV3-eval Cross-lingual

<details>
<summary><b>CV3-eval Cross-lingual Results (click to expand)</b></summary>

| Direction | Metric | Confucius4-TTS | CosyVoice2† | CosyVoice3-0.5B† | CosyVoice3-1.5B† | OmniVoice† | VoxCPM2 |
|---|---|---:|---:|---:|---:|---:|---:|
| en→zh | CER↓ | **6.16** | 13.50 | 8.48 | 8.01 | 6.53 | 6.29 |
| ja→zh | CER↓ | 4.87 | 48.10 | 6.86 | 6.78 | 52.64 | **4.20** |
| ko→zh | CER↓ | 1.28 | 7.70 | 5.24 | 3.30 | 1.71 | **1.20** |
| zh→en | WER↓ | **3.19** | 17.10 | 6.83 | 5.39 | 3.72 | 3.84 |
| ja→en | WER↓ | **3.44** | 11.20 | 5.86 | 5.94 | 5.25 | 4.10 |
| ko→en | WER↓ | **3.42** | 13.10 | 18.30 | 13.70 | 3.91 | 5.69 |

† Requires reference text.

</details>

### X-Voice Benchmark

<details>
<summary><b>X-Voice Cross-lingual Results (click to expand)</b></summary>

| Direction | Metric | Confucius4-TTS | X-Voice | IndexTTS2 | OmniVoice† | VoxCPM2 |
|---|---|---:|---:|---:|---:|---:|
| de→zh | CER↓ | **2.86** | 3.07 | 3.46 | 7.79 | 3.62 |
| en→zh | CER↓ | 3.21 | **3.06** | 3.78 | 3.30 | 3.35 |
| fr→zh | CER↓ | **2.70** | 3.01 | 3.53 | 8.16 | 3.75 |
| ja→zh | CER↓ | 3.50 | **3.39** | 4.11 | 60.88 | 4.53 |
| ko→zh | CER↓ | **2.86** | 3.13 | 2.90 | 7.35 | 6.33 |
| th→zh | CER↓ | 2.82 | **2.79** | 3.08 | 2.85 | 5.96 |
| vi→zh | CER↓ | **2.75** | 2.78 | 2.98 | 6.59 | 3.65 |

† Requires reference text.

</details>

### Seed-TTS-eval

<details>
<summary><b>Seed-TTS-eval English & Chinese Zero-shot Results (click to expand)</b></summary>

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

† Requires reference text.

</details>

### MiniMax-MLS-Test

<details>
<summary><b>MiniMax-MLS-Test Results (click to expand)</b></summary>

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

† Requires reference text.

</details>

---

## Acknowledgements

Confucius4-TTS builds on the following open-source projects:

- **[Qwen3-TTS](https://github.com/QwenLM/Qwen3-TTS)** — Speaker encoder (ECAPA-TDNN) and text embedding projector architectures
- **[CosyVoice](https://github.com/FunAudioLLM/CosyVoice)** — Text normalization pipeline
- **[Amphion / MaskGCT](https://github.com/open-mmlab/Amphion)** — Semantic codec implementation
- **[w2v-BERT 2.0](https://huggingface.co/facebook/w2v-bert-2.0)** — Semantic feature extraction and speaker conditioning
- **[Seed-VC](https://github.com/Plachtaa/seed-vc)** — Flow matching architecture reference
- **[BigVGAN](https://github.com/NVIDIA/BigVGAN)** — High-fidelity neural vocoder for mel-spectrogram to waveform synthesis

---

## Citation

If you find Confucius4-TTS useful in your research or project, please consider citing:

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
