# LipVoicer: Generating Speech from Silent Videos Guided by Lip Reading

<div align="center">

[Paper](https://openreview.net/pdf?id=ZZCPSC5OgD) |
[Introduction](#Introduction) |
[Preparation](#Preparation) |
[Benchmark](#Benchmark-evaluation) |
[Inference](#Speech-prediction) |
[Model zoo](#Model-Zoo) |
</div>

## Authors
Yochai Yemini, Aviv Shamsian, Lior Bracha, Sharon Gannot and Ethan Fetaya

## Demo Page
The [demo page](https://lipvoicer.github.io/) includes many sample videos and comparisons to other baslines.

## Introduction
Official implementation of LipVoicer, a lip-to-speech method. Given a silent video, we first predict the spoken text using a pre-trained lip-reading network. We then condition a diffusion model on the video and use the extracted text through a classifier-guidance mechanism where a pre-trained ASR serves as the classifier. LipVoicer outperforms multiple lip-to-speech baselines on LRS2 and LRS3, which are in-the-wild datasets with hundreds of unique speakers in their test set and an unrestricted vocabulary.

The lip reading network used in LipVoicer is taken from the [Visual Speech Recognition for Multiple Languages](https://github.com/mpc001/Visual_Speech_Recognition_for_Multiple_Languages) repository.
The ASR system is adapted from [Audio-Visual Efficient Conformer for Robust Speech Recognition](https://github.com/burchim/AVEC/tree/master).

## Installation
1. Clone the repository:
```
git clone https://github.com/yochaiye/LipVoicer.git
cd LipVoicer
```
2. Install the required packages and ffmpeg
```
pip install -r requirements.txt
conda install -c conda-forge ffmpeg
cd ..
```
3. Install `ibug.face_detection`
```
git clone https://github.com/hhj1897/face_detection.git
cd face_detection
git lfs pull
pip install -e .
cd ..
```
4. Install `ibug.face_alignment`
```
git clone https://github.com/hhj1897/face_alignment.git
cd face_alignment
pip install -e .
cd ..
```
5. Install [RetinaFace](https://pypi.org/project/retina-face/) or [MediaPipe](https://pypi.org/project/mediapipe/) face tracker
<!--6. Install ctcdecode for the ASR beam search
```
git clone --recursive https://github.com/parlance/ctcdecode.git
cd ctcdecode
pip install .
cd ..
```-->

## Pretrained Models
We provide pretrained checkpoints for LipVoicer so you can kick-start generating speech for silent videos. You can download checkpoint for the following models

* MelGen trained on LRS2/LRS3
* ASR finetuned for LipVoicer on LRS2/LRS3
* Language model for the ASR (provided by [Audio-Visual Efficient Conformer for Robust Speech Recognition](https://github.com/burchim/AVEC/tree/master))
* Lip-reading network and its language model (provided by [Visual Speech Recognition for Multiple Languages](https://github.com/mpc001/Visual_Speech_Recognition_for_Multiple_Languages))
* HiFi-GAN trained on 16KHz audio signals. In the paper we used DiffWave as the vocoder, but since HiFi-GAN is faster it is used here as the vocoder.

The simplest and fastet way to download the models is to run
```
python download_checkpoints.py
```
which will download all pretrained checkpoints and put the in the right place in the repository. 

Alternatively, you can download individual checkpoint from [Google Drive](https://drive.google.com/drive/u/1/folders/1IdYw_jiKyfBdemiel6XD2rHkbIG12MbM)

## Inference forIn-the-Wild Videos
To generate a speech signal for your video, you first need to edit the following arguments in the hydra [config file](https://github.com/yochaiye/LipVoicer/blob/main/configs/config.yaml)

* `generate.ckpt_path`
* `generate.video_path`
* `generate.save_dir`

You can also play with the values of `w_video, w_asr, ast_start`. Then run the following command
```
python inference_real_video.py
```
which will also take care of converting the video fps rate to 25Hz if necessary, mouth cropping and lip-reading.

## Benchmarks Test Audio Signals Generated by LipVoicer
We provide the audio generated by LipVoicer for the test videos of LRS2 and LRS3. They were used to compute the metrics in the paper, and therefore it will hopefully facilitate future comparisons.

The links are given below:
* [LRS2](https://drive.google.com/uc?id=15BgeaU-j4B-o9WfP-d-BSnnjslAxOHkt)
* [LRS3](https://drive.google.com/uc?id=1y4O3grI7uwnjN5GIoydrGhyvK63ptLSy)

## Training
For training LipVoicer on the benchmark datasets, please download [LRS2](https://www.robots.ox.ac.uk/~vgg/data/lip_reading/lrs2.html) or [LRS3](https://mmai.io/datasets/lip_reading/). 

### Data Preparation
Perform the following steps inside the LipVoicer directory:
1. Extract the audio files from the videos (audio files will be saved in a WAV format)
```
python dataloaders/extract_audio_from_video.py --ds_dir <path_to_video_dir>
                                               --split <trainval/test/...>
                                               --out_dir <output_directory>
```
The `wav` files will be saved to `output_directory/split`

2. Compute the log mel-spectrograms and save them
   ```
   python dataloaders/wav2mel.py dataset.audios_dir=<path_to_directory_with_extracted_wav_files>
   ```
   It will save the mel-spectrograms with extension `.wav.spec`.
<!---
3. Crop the mouth regions of the videos, convert to greyscale and save to `<mouthrois_dir>`
Download option?

### Train
4. Train MelGen
```
CUDA_VISIBLE_DEVICES=<...> python train_melgen.py train.save_dir=<save_dir> \
                                                  dataset.dataset_path=<dataset_path> \
                                                  dataset.audio_dir=<audio_dir> \
                                                  dataset.mouthrois_dir=<mouthrois_dir>
```
5. Finetune the modified ASR, which now includes the diffusion time-step embedding.
For further details on how to carry out this step, please refer to [Audio-Visual Efficient Conformer for Robust Speech Recognition](https://github.com/burchim/AVEC/tree/master).
--->

## Inference
### Random (In-the-Wild) Video
### For Benchmark Datasets
If you wish to generate audio for all of the test videos of LRS2/LRS3, use the following
```
python generate_full_test_split.py generate.save_dir=<save_dir> \
                                   generate.lipread_text_dir=<lipread_text_dir> \
                                   dataset.dataset_path=<dataset_path> \
                                   dataset.audio_dir=<audio_dir> \
                                   dataset.mouthrois_dir=<mouthrois_dir
```
