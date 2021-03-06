# Cách train
Chuẩn bị:
-Train trên windows 10:
cài thêm `colorama` để tqdm hoạt động đúng, không bị xuống nhiều dòng.
```
pip install colorama
```

1. Tạo TTS conda envs: 
```bash
conda install pytorch torchvision torchaudio cudatoolkit=10.2 -c pytorch
```
2. Download `github.com/tuananhktmt/FastSpeech2` về máy, giải nén, vào thư mục FastSpeech2-master, chạy:
```bash
pip install -r requirements.txt
```
3. Download dataset: LJSpeech 1.1 về máy, lưu trong `./data/ljspeech` :
[LJSpeech](https://keithito.com/LJ-Speech-Dataset/) 
```
[data/ljspeech/  - metadata.csv
                 - wavs: 
                         - LJ001-0003.wav
                         - LJ001-0004.wav
                         - LJ001-0005.wav
                         - LJ001-0001.wav
                         - LJ001-0002.wav
cụ thể: 
"D:\Code\TTS\FastSpeech2-master\data\ljspeech\metadata.csv"
"D:\Code\TTS\FastSpeech2-master\data\ljspeech\wavs\LJ001-0003.wav"
"D:\Code\TTS\FastSpeech2-master\data\ljspeech\wavs\LJ001-0004.wav"
"D:\Code\TTS\FastSpeech2-master\data\ljspeech\wavs\LJ001-0001.wav"
"D:\Code\TTS\FastSpeech2-master\data\ljspeech\wavs\LJ001-0002.wav"
..........
```

Xoá bớt data đi train cho nhanh
4. Sửa đường dẫn data trong file `config/LJSpeech/preprocess.yaml`, nếu có xoá file đi, thì tính lại `val_size:`, đổi thành số phù hợp với tổng số file trong data, rồi chạy:
```
python prepare_align.py config/LJSpeech/preprocess.yaml
```
5. Alignments for the LJSpeech and AISHELL-3 datasets are provided [here](https://drive.google.com/drive/folders/1DBRkALpPd6FL9gjHMmMEdHODmkgNIIK4?usp=sharing).
You have to unzip the files in `preprocessed_data/LJSpeech/TextGrid/`.
lúc này, cái TextGrid sẽ có thư mục kiểu như này:
```
"D:\Code\TTS\FastSpeech2-master\data\preprocessed_data\LJSpeech\TextGrid\LJSpeech\LJ001-0003.TextGrid
                                                                                 \LJ001-0004.TextGrid
                                                                                 \LJ001-0001.TextGrid
                                                                                 \LJ001-0002.TextGrid
                                                                                 ..........
```
Dòng số 67-69, sửa thành như này, để nó hiển thị đúng tiến độ đang xử lý: 
file: `D:\Code\TTS\FastSpeech2-master\preprocessor\preprocessor.py`
```python
        for i, speaker in enumerate( tqdm(os.listdir(self.in_dir)),  desc = 'Dataset'):
            speakers[speaker] = i
            for wav_name in tqdm(os.listdir(os.path.join(self.in_dir, speaker)), desc = 'speaker'):

```

After that, run the preprocessing script by
```
python preprocess.py config/LJSpeech/preprocess.yaml
``` 
vậy là đã xong bước này. Đến đây, bạn kiểm tra  xem có các thư mục sau đây không, và trong nó có nội dung không, nếu có là OK rồi:
```
"D:\Code\TTS\FastSpeech2-master\data\preprocessed_data\LJSpeech\energy"
"D:\Code\TTS\FastSpeech2-master\data\preprocessed_data\LJSpeech\mel"
"D:\Code\TTS\FastSpeech2-master\data\preprocessed_data\LJSpeech\pitch"
"D:\Code\TTS\FastSpeech2-master\data\preprocessed_data\LJSpeech\TextGrid"
"D:\Code\TTS\FastSpeech2-master\data\preprocessed_data\LJSpeech\duration"
```
Có cả thêm các file này nữa:
```
"D:\Code\TTS\FastSpeech2-master\data\preprocessed_data\LJSpeech\speakers.json"
"D:\Code\TTS\FastSpeech2-master\data\preprocessed_data\LJSpeech\stats.json"
"D:\Code\TTS\FastSpeech2-master\data\preprocessed_data\LJSpeech\train.txt"
"D:\Code\TTS\FastSpeech2-master\data\preprocessed_data\LJSpeech\val.txt"
```
trong thư mục data:
```
D:\Code\TTS\FastSpeech2-master\data\
├───ljspeech
│   └───wavs
├───preprocessed_data
│   └───LJSpeech
│       ├───duration
│       ├───energy
│       ├───mel
│       ├───pitch
│       └───TextGrid
│           └───LJSpeech
└───raw_data
    └───LJSpeech
        └───LJSpeech
```        
6. Train
Giải nén file này: `"D:\Code\TTS\FastSpeech2-master\hifigan\generator_LJSpeech.pth.tar.zip"` ra tại chỗ để có `hifigan\generator_LJSpeech.pth.tar`
giảm bớt thời gian train, bằng cách giảm tổng số epoch, và các bước liên quan. Sửa file `D:\Code\TTS\FastSpeech2-master\config\LJSpeech\train.yaml` như này:
```
step:
  total_step: 500 #900000
  log_step: 100
  synth_step: 200 #1000
  val_step: 100   #1000
  save_step: 100 # 100000
```  
rồi chạy:
```
python train.py -p config/LJSpeech/preprocess.yaml -m config/LJSpeech/model.yaml -t config/LJSpeech/train.yaml
```
tuy nhiên trên win, cái tqdm lồng nhau nó chạy không tốt lắm, nó cứ bị xuống dòng khi hết 100%, cần phải sửa lại code cho nó ngon

Cuối cùng, sau khi train xong, xem kết quả trong này:
```
PS D:\Code\TTS\FastSpeech2-master\output>
D:.
├───chkpoint
├───ckpt
│   └───LJSpeech
│       └───data
├───log
│   └───LJSpeech
│       ├───train
│       └───val
└───result
    └───LJSpeech
```
có thể chạy tensorboard để nhìn kết quả cho nó rõ nét hơn.


# FastSpeech 2 - PyTorch Implementation

This is a PyTorch implementation of Microsoft's text-to-speech system [**FastSpeech 2: Fast and High-Quality End-to-End Text to Speech**](https://arxiv.org/abs/2006.04558v1). 
This project is based on [xcmyz's implementation](https://github.com/xcmyz/FastSpeech) of FastSpeech. Feel free to use/modify the code.

There are several versions of FastSpeech 2.
This implementation is more similar to [version 1](https://arxiv.org/abs/2006.04558v1), which uses F0 values as the pitch features.
On the other hand, pitch spectrograms extracted by continuous wavelet transform are used as the pitch features in the [later versions](https://arxiv.org/abs/2006.04558).

![](./img/model.png)

# Updates
- 2021/2/26: Support English and Mandarin TTS
- 2021/2/26: Support multi-speaker TTS (AISHELL-3 and LibriTTS)
- 2021/2/26: Support MelGAN and HiFi-GAN vocoder

# Audio Samples
Audio samples generated by this implementation can be found [here](https://ming024.github.io/FastSpeech2/). 

# Quickstart

## Dependencies
You can install the Python dependencies with
```
pip3 install -r requirements.txt
```

## Inference

You have to download the [pretrained models](https://drive.google.com/drive/folders/1DOhZGlTLMbbAAFZmZGDdc77kz1PloS7F?usp=sharing) and put them in ``output/ckpt/LJSpeech/`` or ``output/ckpt/AISHELL3``.

For English single-speaker TTS, run
```
python3 synthesize.py --text "YOUR_DESIRED_TEXT" --restore_step 900000 --mode single -p config/LJSpeech/preprocess.yaml -m config/LJSpeech/model.yaml -t config/LJSpeech/train.yaml
```

For Mandarin multi-speaker TTS, try
```
python3 synthesize.py --text "大家好" --speaker_id SPEAKER_ID --restore_step 900000 --mode single -p config/LJSpeech/preprocess.yaml -m config/LJSpeech/model.yaml -t config/LJSpeech/train.yaml
```

The generated utterances will be put in ``output/result/``.

Here is an example of synthesized mel-spectrogram of the sentence "Printing, in the only sense with which we are at present concerned, differs from most if not from all the arts and crafts represented in the Exhibition", with the English single-speaker TTS model.  
![](./img/synthesized_melspectrogram.png)

## Batch Inference
Batch inference is also supported, try

```
python3 synthesize.py --source preprocessed_data/LJSpeech/val.txt --restore_step 900000 --mode batch -p config/LJSpeech/preprocess.yaml -m config/LJSpeech/model.yaml -t config/LJSpeech/train.yaml
```
to synthesize all utterances in ``preprocessed_data/LJSpeech/val.txt``

## Controllability
The pitch/volume/speaking rate of the synthesized utterances can be controlled by specifying the desired pitch/energy/duration ratios.
For example, one can increase the speaking rate by 20 % and decrease the volume by 20 % by

```
python3 synthesize.py --text "YOUR_DESIRED_TEXT" --restore_step 900000 --mode single -p config/LJSpeech/preprocess.yaml -m config/LJSpeech/model.yaml -t config/LJSpeech/train.yaml --duration_control 0.8 --energy_control 0.8
```

# Training

## Datasets

The supported datasets are

- [LJSpeech](https://keithito.com/LJ-Speech-Dataset/): a single-speaker English dataset consists of 13100 short audio clips of a female speaker reading passages from 7 non-fiction books, approximately 24 hours in total.
- [AISHELL-3](http://www.aishelltech.com/aishell_3): a Mandarin TTS dataset with 218 male and female speakers, roughly 85 hours in total.
- [LibriTTS](https://research.google/tools/datasets/libri-tts/): a multi-speaker English dataset containing 585 hours of speech by 2456 speakers.

We take LJSpeech as an example hereafter.

## Preprocessing
 
First, run 
```
python3 prepare_align.py config/LJSpeech/preprocess.yaml
```
for some preparations.

As described in the paper, [Montreal Forced Aligner](https://montreal-forced-aligner.readthedocs.io/en/latest/) (MFA) is used to obtain the alignments between the utterances and the phoneme sequences.
Alignments for the LJSpeech and AISHELL-3 datasets are provided [here](https://drive.google.com/drive/folders/1DBRkALpPd6FL9gjHMmMEdHODmkgNIIK4?usp=sharing).
You have to unzip the files in ``preprocessed_data/LJSpeech/TextGrid/``.

After that, run the preprocessing script by
```
python3 preprocess.py config/LJSpeech/preprocess.yaml
```

Alternately, you can align the corpus by yourself. 
Download the official MFA package and run
```
./montreal-forced-aligner/bin/mfa_align raw_data/LJSpeech/ lexicon/librispeech-lexicon.txt english preprocessed_data/LJSpeech
```
or
```
./montreal-forced-aligner/bin/mfa_train_and_align raw_data/LJSpeech/ lexicon/librispeech-lexicon.txt preprocessed_data/LJSpeech
```

to align the corpus and then run the preprocessing script.
```
python3 preprocess.py config/LJSpeech/preprocess.yaml
```

## Training

Train your model with
```
python3 train.py -p config/LJSpeech/preprocess.yaml -m config/LJSpeech/model.yaml -t config/LJSpeech/train.yaml
```

The model takes less than 10k steps (less than 1 hour on my GTX1080Ti GPU) of training to generate audio samples with acceptable quality, which is much more efficient than the autoregressive models such as Tacotron2.

# TensorBoard

Use
```
tensorboard --logdir output/log/LJSpeech
```

to serve TensorBoard on your localhost.
The loss curves, synthesized mel-spectrograms, and audios are shown.

![](./img/tensorboard_loss.png)
![](./img/tensorboard_spec.png)
![](./img/tensorboard_audio.png)

# Implementation Issues

- Following [xcmyz's implementation](https://github.com/xcmyz/FastSpeech), I use an additional Tacotron-2-styled Postnet after the decoder, which is not used in the original paper.
- Gradient clipping is used in the training.
- In my experience, using phoneme-level pitch and energy prediction instead of frame-level prediction results in much better prosody, and normalizing the pitch and energy features also helps. Please refer to ``config/README.md`` for more details.

Please inform me if you find any mistakes in this repo, or any useful tips to train the FastSpeech 2 model.

# References
- [FastSpeech 2: Fast and High-Quality End-to-End Text to Speech](https://arxiv.org/abs/2006.04558), Y. Ren, *et al*.
- [xcmyz's FastSpeech implementation](https://github.com/xcmyz/FastSpeech)
- [TensorSpeech's FastSpeech 2 implementation](https://github.com/TensorSpeech/TensorflowTTS)
- [rishikksh20's FastSpeech 2 implementation](https://github.com/rishikksh20/FastSpeech2)
