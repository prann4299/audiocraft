# Audiocraft
![docs badge](https://github.com/facebookresearch/audiocraft/workflows/audiocraft_docs/badge.svg)
![linter badge](https://github.com/facebookresearch/audiocraft/workflows/audiocraft_linter/badge.svg)
![tests badge](https://github.com/facebookresearch/audiocraft/workflows/audiocraft_tests/badge.svg)

Audiocraft is a PyTorch library for deep learning research on audio generation. At the moment, it contains the code for MusicGen, a state-of-the-art controllable text-to-music model.

## MusicGen

Audiocraft provides the code and models for MusicGen, [a simple and controllable model for music generation][arxiv]. MusicGen is a single stage auto-regressive
Transformer model trained over a 32kHz <a href="https://github.com/facebookresearch/encodec">EnCodec tokenizer</a> with 4 codebooks sampled at 50 Hz. Unlike existing methods like [MusicLM](https://arxiv.org/abs/2301.11325), MusicGen doesn't not require a self-supervised semantic representation, and it generates
all 4 codebooks in one pass. By introducing a small delay between the codebooks, we show we can predict
them in parallel, thus having only 50 auto-regressive steps per second of audio.
Check out our [sample page][musicgen_samples] or test the available demo!

<a target="_blank" href="https://colab.research.google.com/drive/1fxGqfg96RBUvGxZ1XXN07s3DthrKUl4-?usp=sharing">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/>
</a>
<a target="_blank" href="https://huggingface.co/spaces/facebook/MusicGen">
  <img src="https://huggingface.co/datasets/huggingface/badges/raw/main/open-in-hf-spaces-sm.svg" alt="Open in HugginFace"/>
</a>
<br>

## Installation
Audiocraft requires Python 3.9, PyTorch 2.0.0, and a GPU with at least 16 GB of memory (for the medium-sized model). To install Audiocraft, you can run the following:

```shell
# Best to make sure you have torch installed first, in particular before installing xformers.
# Don't run this if you already have PyTorch installed.
pip install 'torch>=2.0'
# Then proceed to one of the following
pip install -U audiocraft  # stable release
pip install -U git+https://git@github.com/facebookresearch/audiocraft#egg=audiocraft  # bleeding edge
pip install -e .  # or if you cloned the repo locally
```

## Usage
You can play with MusicGen by running the jupyter notebook at [`demo.ipynb`](./demo.ipynb) locally, or use the provided [colab notebook](https://colab.research.google.com/drive/1fxGqfg96RBUvGxZ1XXN07s3DthrKUl4-?usp=sharing). Finally, a demo is also available on the [`facebook/MusiGen`  HugginFace Space](https://huggingface.co/spaces/facebook/MusicGen) (huge thanks to all the HF team for their support).

## API

We provide a simple API and 4 pre-trained models. The pre trained models are:
- `small`: 300M model, text to music only,
- `medium`: 1.5B model, text to music only,
- `melody`: 1.5B model, text to music and text+melody to music,
- `large`: 3.3B model, text to music only.

We observe the best trade-off between quality and compute with the `medium` or `melody` model.
In order to use MusicGen locally **you must have a GPU**. We recommend 16GB of memory, but smaller
GPUs will be able to generate short sequences, or longer sequences with the `small` model.

See after a quick example for using the API.

```python
import torchaudio
from audiocraft.models import MusicGen
from audiocraft.data.audio import audio_write

model = MusicGen.get_pretrained('melody')
model.set_generation_params(duration=8)  # generate 8 seconds.
wav = model.generate_unconditional(4)    # generates 4 unconditional audio samples
descriptions = ['happy rock', 'energetic EDM', 'sad jazz']
wav = model.generate(descriptions)  # generates 3 samples.

melody, sr = torchaudio.load('./assets/bach.mp3')
# generates using the melody from the given audio and the provided descriptions.
wav = model.generate_with_chroma(descriptions, melody[None].expand(3, -1, -1), sr)

for idx, one_wav in enumerate(wav):
    # Will save under {idx}.wav, with loudness normalization at -14 db LUFS.
    audio_write(f'{idx}', one_wav, model.sample_rate, strategy="loudness")
```


## Model Card

See [the model card page](./MODEL_CARD.md).

## FAQ

#### Will the training code be released?

Yes. We will soon release the training code for MusicLM and EnCodec.


## Citation
```
bib here
```

## License
* The code in this repository is released under the MIT license as found in the [LICENSE file](LICENSE).
* The weights in this repository are released under the CC-BY-NC 4.0 license as found in the [LICENSE_weights file](LICENSE_weights).

[arxiv]: https://arxiv.org/abs/2306.05284
[musicgen_samples]: https://ai.honu.io/papers/musicgen/
