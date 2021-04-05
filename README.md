

# End-to-End Image Stitching Network via Multi-Homography Estimation

Official project page **"End-to-End Image Stitching Network via Multi-Homography Estimation"**

Accepted at ***IEEE Signal Processing Letters 2021***

https://ieeexplore.ieee.org/document/9393563

![Title](./assets/title.gif)

Dae-Young Song<sup>[1]</sup> , Gi-Mun Um<sup>[2]</sup>, Hee Kyung Lee<sup>[2]</sup> and Donghyeon Cho<sup>[1]</sup>

This paper is one of the outcomes of an industry-academic cooperation project with **ETRI** (Electronics and Telecommunications Research Institute).

The dataset and source code are ETRI's assets.

[1] The Department of Electronics Engineering, Chungnam National University

[2] Communication Media Research Laboratory, ETRI



## I. Abstract

In this paper, we propose an end-to-end stitching network, which takes two images with a narrow field of view (FOV) as inputs, and produces a single image with a wide FOV. Our method estimates multiple homographies to cover the depth differences in the scene and is therefore robust against parallax distortion. In particular, global warping maps are generated using estimated multiple homographies and adjusted by local displacement maps. The final result is made by warping input images multiple times using the warping maps and then merging warped images with the weight maps. Multiple homographies, local displacement maps, and weight maps are generated simultaneously by our stitching network. To train the stitching network, we construct a dataset using the CARLA simulator. Then, using this dataset, our network is trained by end-to-end supervised learning based on appearance matching loss and depth layer loss. In experiments, we show that our method is superior to existing methods both qualitatively and quantitatively. Also, we provide various empirical studies for in-depth analysis as well as the result of the expansion to 360 degree panoramas.



## II. Network Architecture



![Title](./assets/Figure2.JPG)

Notations in this section:

- 🖸 :  Width dimension concatenation
- \+ : Channel dimension concatenation
- \* : Immediately followed by batch normalization and ELU(Exponential Linear Unit).
- (L) : Features from left image
- (R) : Features from right image
- ***K*** : Number of homography (left + right)



### 1. Encoder

| layer   | kernel | stride | in/out chns | scale | pad  | input   |
| ------- | ------ | ------ | ----------- | ----- | ---- | ------- |
| conv1*  | 7      | 1      | 3/32        | 1     | 3    | image   |
| conv1b* | 7      | 2      | 32/32       | 1/2   | 3    | conv1*  |
| conv2*  | 5      | 1      | 32/64       | 1/2   | 2    | conv1b* |
| conv2b* | 5      | 2      | 64/64       | 1/4   | 2    | conv2*  |
| conv3b  | 3      | 1      | 64/128      | 1/4   | 1    | conv2b* |
| conv3b* | 3      | 2      | 128/128     | 1/8   | 1    | conv3*  |
| conv4*  | 3      | 1      | 128/256     | 1/8   | 1    | conv3b* |
| conv4b* | 3      | 2      | 256/256     | 1/16  | 1    | conv4*  |
| conv5*  | 3      | 1      | 256/512     | 1/16  | 1    | conv4b* |
| conv5b* | 3      | 2      | 512/512     | 1/32  | 1    | conv5*  |
| conv6*  | 3      | 1      | 512/512     | 1/32  | 1    | conv5b* |
| conv6b* | 3      | 2      | 512/512     | 1/64  | 1    | conv6*  |
| conv7*  | 3      | 1      | 512/512     | 1/64  | 1    | conv6b* |
| conv7b* | 3      | 2      | 512/512     | 1/128 | 1    | conv7*  |



### 2. Regressor

| layer    | kernel | stride | in/out chns     | scale | pad  | input                      |
| -------- | ------ | ------ | --------------- | ----- | ---- | -------------------------- |
| conv45*  | 3      | 2      | 1024/512        | 1/256 | 1    | conv7b\*(L) \+ conv7b\*(R) |
| conv5a*  | 3      | 1      | 512/512         | 1/256 | 1    | conv45*                    |
| conv5b*  | 3      | 1      | 512/512         | 1/256 | 1    | conv5a*                    |
| conv56*  | 1      | 2      | 512/512         | 1/512 | 1    | conv5b*                    |
| conv6a*  | 1      | 1      | 512/512         | 1/512 | 1    | conv56*                    |
| conv6b*  | 1      | 1      | 512/512         | 1/512 | 1    | conv6a*                    |
| avg_pool |        |        |                 |       |      |                            |
| FC       |        |        | 512/(6x***K***) |       |      | avgpool(conv6b*)           |

 ***K*** homography Estimation : FC reshape -> [Batch X ***K*** X 2 X 3]



### 3. Decoder

| layer    | kernel | stride | in/out chns | scale | pad  | input                                |
| -------- | ------ | ------ | ----------- | ----- | ---- | ------------------------------------ |
| upconv7* | 3      | 1      | 512/512     | 1/64  | 1    | conv7b\*(L) 🖸 conv7b\*(R)            |
| iconv7*  | 3      | 1      | 1024/512    | 1/64  | 1    | upconv7* + conv6b\*(L) 🖸 conv6b\*(R) |
| upconv6* | 3      | 1      | 512/512     | 1/32  | 1    | iconv7*                              |
| iconv6*  | 3      | 1      | 1024/512    | 1/32  | 1    | upconv6* + conv5b\*(L) 🖸 conv5b\*(R) |
| upconv5* | 3      | 1      | 512/256     | 1/16  | 1    | iconv6*                              |
| iconv5*  | 3      | 1      | 512/256     | 1/16  | 1    | upconv5* + conv4b\*(L) 🖸 conv4b\*(R) |
| upconv4* | 3      | 1      | 256/128     | 1/8   | 1    | iconv5*                              |
| iconv4*  | 3      | 1      | 256/128     | 1/8   | 1    | upconv4* + conv3b\*(L) 🖸 conv3b\*(R) |
| upconv3* | 3      | 1      | 128/64      | 1/4   | 1    | iconv4*                              |
| iconv3*  | 3      | 1      | 128/64      | 1/4   | 1    | upconv3* + conv2b\*(L) 🖸 conv2b\*(R) |
| upconv2* | 3      | 1      | 64/32       | 1/2   | 1    | iconv3*                              |
| iconv2*  | 3      | 1      | 64/32       | 1/2   | 1    | upconv2* + conv1b\*(L) 🖸 conv1b\*(R) |
| upconv1* | 3      | 1      | 32/16       | 1     | 1    | iconv2*                              |
| iconv1*  | 3      | 1      | 16/16       | 1     | 1    | upconv1*                             |
| weight   | 3      | 1      | 16/***N***  | 1     | 1    | iconv1*                              |
| softmax  |        |        |             |       |      | weight                               |
| disp     | 3      | 1      | 16/***N***  | 1     | 1    | iconv1*                              |
| Tanh     |        |        |             |       |      | disp                                 |

***Weight maps*** : softmax(weight)

***Displacement maps*** : 0.02 x Tanh(disp)



## III. Training Dataset in the Paper

all datasets are produced using ***CARLA***<sup>[3]</sup> simulator and autopilot.

- total 42,777 sets

- Input FoV : 90 degree (rotated ±30 degree from ground-truth)

- Ground-truth FoV : 120 degree

- Direction : front (17,599 sets) + right (25,178 sets)

- input image configuration

  ![Title](./assets/input_figure.JPG)

- Examples

  | left (front dir.)                         | right (front dir.)                         |
  | ----------------------------------------- | ------------------------------------------ |
  | ![Title](./assets/config1/front_left.png) | ![Title](./assets/config1/front_right.png) |
  | **ground-truth (front dir.)**             | **depth (front dir.)**                     |
  | ![Title](./assets/config1/front_gt.png)   | ![Title](./assets/config1/front_depth.png) |
  | **left (right dir.)**                     | **right (right dir.)**                     |
  | ![Title](./assets/config1/right_left.png) | ![Title](./assets/config1/right_right.png) |
  | **ground-truth (right dir.)**             | **depth (right dir.)**                     |
  | ![Title](./assets/config1/right_gt.png)   | ![Title](./assets/config1/right_depth.png) |

[3] : https://github.com/carla-simulator/carla





# Bibtex

