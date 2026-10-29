# YingMusic-Singer：基于免标注旋律引导的零样本歌声合成与编辑

## 项目概述 ✨

**YingMusic-Singer** 是一个基于**免标注旋律引导**的**零样本歌声合成（SVS）与编辑**统一框架。为满足实际应用中的规模化需求，该框架无需依赖成本高昂的音素级对齐和人工旋律标注，即可在零样本条件下，结合任意歌词与参考旋律进行歌声合成或编辑。

### 核心特性 🔧

- **合成与编辑一体化**：在同一框架中集成零样本歌声合成与编辑功能。
- **免标注旋律引导**：自动从参考音频中提取旋律，无需人工 MIDI 标注或音素对齐。
- **零样本生成**：无需针对目标音色进行微调，即可根据任意歌词和旋律生成高质量歌声。
- **灵活的旋律输入**：支持使用参考音频或直接输入 MIDI 文件作为旋律来源。

## 安装 🛠️

```bash
git clone https://github.com/xianyu925/AudioProject.git
cd AudioProject

conda create -n singer python=3.12
conda activate singer

# 可根据 GPU 的兼容情况调整 torch 和 flash_attn 的版本。
# 安装 PyTorch
pip3 install torch==2.9.1 torchaudio==2.9.1 --index-url https://download.pytorch.org/whl/cu126

# 安装 FlashAttention
pip3 install https://github.com/Dao-AILab/flash-attention/releases/download/v2.8.1/flash_attn-2.8.1+cu12torch2.9cxx11abiTRUE-cp312-cp312-linux_x86_64.whl

# 安装其他依赖
pip3 install -r requirements.txt
```

## 快速开始 🚀

### 下载模型权重

在项目根目录执行以下任一组命令，将模型权重及配套配置文件下载至 `./ckp` 目录，两种方式任选其一即可。

**方式一：从 [Hugging Face](https://huggingface.co/GiantAILab/YingMusic-Singer) 下载**

使用 Hugging Face 的 [`snapshot_download` 接口](https://huggingface.co/docs/huggingface_hub/guides/download)：

```bash
python -m pip install huggingface_hub
python - <<'PY'
from huggingface_hub import snapshot_download

snapshot_download(
    repo_id="GiantAILab/YingMusic-Singer",
    local_dir="./ckp",
)
PY
```

**方式二：从 [ModelScope](https://www.modelscope.cn/models/giantailab/YingMusic-Singer/) 下载**

使用 ModelScope 的 [`snapshot_download` 接口](https://github.com/modelscope/modelscope/blob/master/modelscope/hub/snapshot_download.py)：

```bash
python -m pip install modelscope
python - <<'PY'
from modelscope.hub.snapshot_download import snapshot_download

snapshot_download(
    model_id="giantailab/YingMusic-Singer",
    local_dir="./ckp",
)
PY
```

下方命令行示例通过 `--ckpt_path ./ckp` 加载已下载的模型；Gradio 界面默认从 `ckp` 目录加载模型。

> **提示**：如果音色参考音频与目标旋律（MIDI 或音频）的音高范围差异较大，建议手动调整调性，以获得更好的效果。

### 1. 命令行推理

建议将音色参考音频的时长控制在 5～7 秒，总时长不超过 45 秒。

```bash
unset PYTHONPATH
export PYTHONPATH=$(pwd)/src:$PYTHONPATH
```

使用 MIDI 文件作为旋律输入：

```bash
python src/singer/model.py --timbre_audio_path resources/audios/male.wav \
    --ckpt_path ./ckp \
    --timbre_audio_content "在爱的回归线，又期待相见。" \
    --midi_file "resources/audios/female__Rnb_Funk__下等马_clip_001.mid" \
    --lyrics "头抬起来，你表情别太奇怪，无大碍。没伤到脑袋，如果我下手太重，私密马赛。习武十载，没下山没谈恋爱，吃光后山七八亩菜，练就这套拳脚，莫以貌取人哉。暮色压台，擂鼓未衰，下一个谁还要来？速来领拜，别耽误我热蒸屉揭盖。" \
    --out_path "outputs/test_yingsinger_zs.wav" \
    --cfg_strength 4.0 \
    --nfe_steps 64 \
    --pitch_shift -1
```

使用音频作为旋律输入：

```bash
python src/singer/model.py --timbre_audio_path resources/audios/male.wav \
    --ckpt_path ./ckp \
    --timbre_audio_content "在爱的回归线，又期待相见。" \
    --melody_audio_path "resources/audios/female__Rnb_Funk__下等马_clip_001.wav" \
    --lyrics "头抬起来，你表情别太奇怪，无大碍。没伤到脑袋，如果我下手太重，私密马赛。习武十载，没下山没谈恋爱，吃光后山七八亩菜，练就这套拳脚，莫以貌取人哉。暮色压台，擂鼓未衰，下一个谁还要来？速来领拜，别耽误我热蒸屉揭盖。" \
    --out_path "outputs/test_yingsinger_zs.wav" \
    --cfg_strength 4.0 \
    --nfe_steps 64 \
    --pitch_shift -1
```

### 2. Gradio 界面推理

运行以下命令启动 Gradio 演示界面：

```bash
python app.py
```
