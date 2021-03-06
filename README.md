This is the code of **Team \WM/** to reproduce our solution for the *Deepfake Detection Challenge* (DFDC).

**Members** (alphabetical order):

- Hanqing Zhao (zhq2015@mail.ustc.edu.cn)
- Hao Cui (cuihao.leo@gmail.com)
- Wenbo Zhou (welbeckz@ustc.edu.cn)

## Content

- `make_dataset.sh` and `make_dataset.py`: Script to extract faces from videos. For dataset processing.
- `train-wsdan.py`: Script to train WS-DAN models.
- `train-xception.py`: Script to train the Xception model.

- `best-submission.py`: Our best submission on Kaggle.

## Environment

We trained models on our lab's Linux cluster. The environment listed below reflects a typical software / hardware configuration in this cluster.

Hardware:

- CPU: Xeon Gold 5120
- GPU: 2080Ti or 1080Ti
- Mem: > 64GB
- Data is stored in SSD.

Software:

- System: Ubuntu 16.04.6 with Linux 4.4.0 kernel.
- Python: 3.6 or 3.7 distributed by Anaconda.
- CUDA: 10.0

## Reproduce Guide

### Code & Data Dependency

- For dependent python packages, please refer to `requirements.txt`.
- Other external code dependencies are provided as git submodules in the  `external/` folder.
  - Run `git submodule init && git submodule update` to fetch these dependencies.
- Other data dependencies can be downloaded from [Google Drive](https://drive.google.com/drive/folders/1ttye8HFwYJA-P8WANGSOLPtC787xGZta?usp=sharing):
  - `RetinaFace-Resnet50-fixed.pth`: Pretrained RetinaFace model.
  - `ckpt_x.pth`: Pretrained weight files for WS-DAN w/ Xception.
  - `ckpt_e.pth`: Pretrained weight files for WS-DAN w/ EfficientNet-b3.
  - `xception-hg-2.pth`: Pretrained Xception weight files.

### Dataset Processing

The script `make_dataset.py` extracts aligned faces from a video file and save as images. It works like this:

```
$ mkdir /path/to/output_frames/
$ python make_dataset.py /path/to/video.mp4 /path/to/output_frames/
```

The script `make_dataset.sh` finds all mp4 files recursively in a directory and calls `make_dataset.py` on each mp4 file.

Supposing that you have downloaded DFDC datasets and extracted all zip files to `videos/`, run the following command to process the whole dataset and save face images to `/mnt/ssd0/dfdc/` for training:

```
$ bash make_dataset.sh videos/ /mnt/ssd0/dfdc/
```

### Training: WS-DAN

WS-DAN is the core part of our final solution. We have trained two variants of WS-DAN: one with Xception and another with EfficientNet-b3 as feature extractors.

Training configs for both variants are provided in `wsdan-conf/` folder. You should check `save_dir`, `datapath` and `pretrained` settings before training.

Note the `pretrained` setting:

- For WS-DAN w/Xception, we used our previously trained Xception model (see the next section) to initialize feature extractor. This should be the path to Xception weight files.
- For WS-DAN w/EfficientNet-b3, we used weight files downloaded by the EfficientNet-PyTorch code which is pretrained on ImageNet. Set it with any non-empty string is fine.

To train WS-DAN w/Xception:

```
$ python train-wsdan.py wsdan-conf/xception.py
```

To train WS-DAN w/EfficientNet:

```
$ python train-wsdan.py wsdan-conf/efb3.py 
```

**Time estimation:** We trained WS-DAN w/ Xception with 6 GPUs for almost a week (50 epoches).

### Training: Xception

Xception part is not of much interest. We had been using two-class Xception as per-face classifier before we found powerful WS-DAN.

Check `xception-conf.py` for various path settings. Then run:

```
$ python train-xception.py xception-conf.py
```

**Time estimation:** With 4 GPUs, 92% or more validation accuracy should be observed in around 12h. We typically trained for more than 1 day (20+ epoches).

### Validation

`submission.py` is the code of our best submission on Kaggle. We got 0.42842 (private) and 0.28680 (public).

## Reference

- RetinaFace implementation: [biubug6/Pytorch_Retinaface](https://github.com/biubug6/Pytorch_Retinaface)
- WS-DAN implementation: [GuYuc/WS-DAN.PyTorch](https://github.com/GuYuc/WS-DAN.PyTorch).
- EfficientNet implementation: [lukemelas/EfficientNet-PyTorch](https://github.com/lukemelas/EfficientNet-PyTorch).
- Face alignment code is from: [deepinsight/insightface](https://github.com/deepinsight/insightface/blob/master/common/face_align.py).



