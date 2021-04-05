

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

