# Autonomous Driving Lane Perception

A video-based lane perception pipeline for autonomous driving. It combines real-time lane segmentation with bounding-box extraction and object tracking to help a vehicle understand lane boundaries and maintain its position on the road.

## Overview

A vehicle-mounted camera captures the road from changing viewpoints and environmental conditions. This project processes video frame by frame to identify lane markings and track relevant objects.

The pipeline is:

```text
Video frame → Segmentation model → Predicted mask → Bounding boxes → BoT-SORT tracking
```

The predicted lane markings can support lane-departure awareness. For example, a downstream driving system could use lane proximity to help keep the vehicle within its lane. Solid and dashed markings may also inform different driving rules. These are potential applications; this repository describes a perception pipeline, not a complete vehicle control system.

## Model and tracking

* **Segmentation:** DeepLabV3+ with a ResNet-50 backbone
* **Tracking:** SORT and BoT-SORT
* **Input:** Video frames resized to 256 × 256 for inference
* **Output:** Predicted segmentation masks, with bounding boxes extracted from the masks for tracking

The original video frames are 1280 × 720. After inference, masks are resized to the video resolution using nearest-neighbor interpolation so class labels remain discrete. Resizing can introduce some visual noise. Training at a higher resolution may improve detail, at the cost of greater computational expense.

## Training

### Datasets

Dataset links and references are provided in the [`Dataset`](Dataset) file. For the Lane Segmentation dataset, this project uses only the **desktop** subset.

### Data augmentation

Augmentation is divided into two groups:

* **Photometric augmentations** change image pixel values, such as brightness and contrast. They are applied to the image only; the corresponding mask remains unchanged.
* **Geometric augmentations** change pixel positions, such as shifts and crops. The same transformation must be applied to both the image and its mask to preserve pixel-level alignment.

To keep geometric transformations synchronized, the mask is concatenated with the RGB image as an additional channel. The augmentation is applied to the combined image, then the mask channel is separated afterward.

### Loss and evaluation

The background class occupies substantially more pixels than the lane class, creating class imbalance. Class weights are used to address this imbalance.

Training uses a combined **Dice loss** and **cross-entropy loss**. The reported evaluation metrics include:

* Mean Intersection over Union (mIoU)
* Accuracy
* Loss

Accuracy is reported as a supplementary metric. Because the background dominates the masks, accuracy alone may not reflect lane-segmentation quality; mIoU and loss are more informative measures to monitor.

## Inference workflow

1. Read a frame from the video.
2. Resize it to the model's expected input size and apply the backbone's required preprocessing.
3. Run DeepLabV3+ to predict a segmentation mask.
4. Resize the mask to the original frame dimensions with nearest-neighbor interpolation.
5. Extract bounding boxes from the predicted mask.
6. Pass the boxes to SORT or BoT-SORT for object tracking.
7. Visualize or use the resulting perception outputs in a downstream system.

The best segmentation model is saved after training and loaded by the video-inference stage.

## Intended use

This project is intended for computer-vision experimentation and autonomous-driving perception research. Its outputs are perception results and should not be treated as a complete or safety-validated vehicle control system.

## Topics

`autonomous-driving` · `lane-detection` · `semantic-segmentation` · `real-time-vision` · `object-tracking` · `deeplabv3plus` · `resnet50` · `botsort` · `computer-vision` · `pytorch`

