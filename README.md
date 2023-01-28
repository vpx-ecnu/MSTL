# Multi-source Templates Learning for Real-time Aerial Object Tracking

This is the official code for the paper "Multi-source Templates Learning for Real-time Aerial Object Tracking".In this work, we present an efficient Aerial Object Tracking method via Multi-source Templates named MSTL. 

## Highlights

### Real-Time Speed on edge platform.

Our tracker can run **~200fps on GPU, ~100fps on CPU, and ~20** on Nvidia Jetson Xavier NX platform. After tensorRT to accelerate, the speed can reach , **~60fps on  Jetson Xavier NX, ~19 fps on Jetson Nano**.

Opposing to previous aerial trackers which evaluate on high-end platform(like Jetson AGX/Orin Series), the proposed tracker can run on extremely cheap edge platform: Jetson Nano and  Jetson Xavier NX.

###  Competitive performance.

|         | Year      | Speed(fps) | UAV123(Prec.) | UAV123@10fpsUAV123(Prec.) | UAV20LUAV123(Prec.) |
| ------- | --------- | ---------- | ------------- | ------------------------- | ------------------- |
| Ours    |           | 209        | 82.35         | 83.50                     | 83.59               |
| TCTrack | CVPR 2022 | 128        | 80.05         | 77.39                     | 67.20               |
| HIFT    | ICCV 2021 | 137        | 78.70         | 74.87                     | 76.32               |







### Demo

![demo_gif](demo_gif.gif)

## Quick Start

### Environment Preparing

```
python 3.7.3
pytorch 1.11.0
opencv-python 4.5.5.64
```

### Training

First, you need to set paths for training datasets in lib/train/admin/local.py.

Then, run the following commands for training.

```bash
python lib/train/run_training.py
```

### Evaluation

First, you need to set paths for this project in lib/test/evaluation/local.py.

 Then, run the following commands for evaluation on four datasets.

- UAV123

```bash
python tracking/test.py MSTL MSTL --dataset uav
```

- UAV20L

```bash
python tracking/test.py MSTL MSTL --dataset uavl
```

- UAV@10fps

```
python tracking/test.py MSTL MSTL --dataset uav10
```

- UAV -x

```
python tracking/test.py MSTL MSTL --dataset uavd
```



### Trained model and Row results

The trained models, the training logs, and the raw tracking results are provided in the [model zoo](MODEL_ZOO.md)





## About joint-optimization for transformer-based trackers.

Typically, there are two main paradigms for transformer tracking:

**Paradigm1:** The template features are processed by the Transformer Encoder whereas the Transformer Decoder fuses search and template features using cross attention layers to compute a response map.  The map is then input into prediction heads to locate the target. (CVPR2021 TrDiMP,  CVPR2021 TransT etc)

As an example, we will use TransT with 4 feature integration layers (Each layer with 2 self-Attention and 2 Cross-Attention) to demonstrate how to implement the proposed decoupling strategy.

- During training:
  - Step 1: Feed the outputs of the second layers into an additional cross-attention mechanism to fuse the search and template features.(only for Paradigm1)
  - Step 2: Use the outputs of the cross-attention as input for prediction heads to locate the target.
  - Step 3: Train the original model together with the additional cross-attention layer and prediction heads.
- During inference:
  - We use the outputs of the second layer locate the target.

![TransT](TransT.png)

| Tracker | Original Succ. (UAV123) | Original params | Succ. (After decoupling) | params(After decoupling) |
| ------- | ----------------------------- | --------------- | -------------------- | ------ |
| TransT  | 69.1                          | 23.0M           | -                    | 16.7M  |



**Paradigm2:** Search features and template features are stacked and processed jointly by the full Transformer. Then, Both encoder and decoder outputs are then further processed to predict the bounding box of the target.(ICCV2021 STARK,  CVPR2022 ToMP, ECCV2022 OSTrack etc)

Here, we take STARK with 6 Encoder-Decoder layers as an example to illustrate how to use the proposed decoupling strategy. 

- During the training phase:
  - Step 1: The encoder outputs are fed into a Corner-prediction network, which is responsible for predicting the bounding box of the target directly.
  - Step 2: The original model and the additional Corner-prediction network are jointly trained.
- During the inference phase:
  - Step 1: The transformer decoder is removed.
  - Step 2: The outputs of the Corner-prediction network are utilized as the final tracking results.

| Tracker | Original Succ.(UAV123) | Original params  | Succ. (After decoupling) | params(After decoupling)  | 
| ------- | ---------------------- | ---------------  | -------------- | ------- | 
| STARK-S | 68.35                  | 28.079M            | **68.55**      | 18.616M | 





## About the UAV platform

The platform mainly consists of four parts, i.e., a [Pixhawk](https://pixhawk.org/) flight controller, a Figure Number transmissions, a visual camera and a [Jetson Xavier NX](https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/jetson-xavier-nx) onboard computer. The onboard computer can obtain the video flow through the USB port. The ground station computer can remotely access the onboard computer and select the target to be tracked through data transmission.



![Hardware](Hardware.png)



## Acknowledgement

-  Thanks for the [PyTracking](https://github.com/visionml/pytracking)  and [stark](https://github.com/researchmm/Stark) Library, which helps us to quickly implement our ideas. We would like to thank their authors for providing great frameworks and toolkits.

