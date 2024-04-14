---
layout: post
title: "[Paper review] UniDistill: A Universal Cross-Modality Knowledge Distillation Framework...."
subtitle: "Knowledge Distillation on 3D object detection"
date: 2024-03-29
author: "Hitesh Kumar"
header-img: "img/unidistll/pexels-daniel-absi-952670.jpg"
tags: [paper, review, bev, kd]
---

# Description - 

Knowledge Distillation is developing technique for bridging the information gap between bigger and smaller model. Smaller models are essentially caters to industry specific needs due to cost and performance aspects. Knowledge Distillation in the context of 3D object detection is a powerful tool, although still developing.

_Disclaimer - No, this is not ChatGPT generated post. I truly want to learn and understand things. However, i have used chatgpt to comprehend some equations and graphs presented in the paper._

CHECK OUT THE PAPER - [Arxiv Link](https://arxiv.org/abs/2303.15083)

## Technical abbreviation - 
1. **BEV** - Bird's-Eye View
2. **MT** - Modality of Teacher
3. **MS** - Modality of Student
4. **KD** - Knowledge Distillation
5. **CNN** - Convolutional Neural Network
6. **NDS** - nuScenes Detection Score
7. **BEVEnc** - Bird's-Eye View Encoder
8. **DetHead** - Detection Head
9. **L_Fea**  - Feature Distillation Loss
10. **L_Rel** - Relation Distillation Loss
11. **L_Resp** - Response Distillation Loss
12. **L_Det**  - Detection Loss
13. **Adapt_1** - Adaptive Layer 1
14. **Adapt_2** - Adaptive Layer 2
15. **mAP** - mean Average Precision
16. **mATE** - mean Average Translation Error
17. **mASE** - mean Average Scale Error
18. **mAOE** - mean Average Orientation Error
19. **mAVE** - mean Average Velocity Error
20. **mAAE** - mean Average Attribute Error


# Paper discussion

## Introduction 

**Unidistill**, a novel method to present 3d object detection in BEV using knowledge distillation. 

- 3D detectors categorized into 2 parts 
	- single modality (based on camera or lidar)
	- multi modality   (combination of 2 modality)

The multi-modalities introduces extra network design and computation overload. Plus the break down of any single modality can breakdown whole system. **UniDistill** addresses these challenges by enabling a single-modality detector (student) to learn from a more complex multi-modality system (teacher) without incurring extra computational costs during inference.

## Knowledge distillation

One author proposed to transfer the depth knowledge of LiDAR points to a camera based student detector by training another camera based teacher with LiDAR projected to perspective view.

UniDistill projects the features of detectors to BEV, and supports LiDAR to camera, camera to LiDAR, fusion to LiDAR and Fusion to camera distillation paths. BEVDistill also performs this cross modality distillation but limits the application.

- UniDistill is CNN based, BEVDistill is Transformer based.
- UniDistill projects features of both the teacher and student detectors into a unified BEV domain. 
- It then calculates three distillation losses to align foreground features for knowledge transfer: feature distillation, relation distillation, and response distillation.
- When the teacher performs worse than the student, adaptive layers are introduced to avoid performance degradation
- Experiments on the **nuScenes dataset** show that UniDistill effectively improves the mAP and NDS metrics of the student detectors by 2.0% to 3.2% across the different distillation paths.

![photo](/hedwig_explains/img/unidistll/unidistill.png)

## Distillation Loss 

### Feature distillation -
- It uses low-level features to transfer semantic knowledge from the teacher detector to the student detector.
	- **feature alignment** : one way to straighaway align F_low_MT with F_low_MS. but its challenging due to different sensor modality. Paper uses "**Selective Feature Alignment**" to address the issues, the feature distillation process selectively aligns features corresponding to the foreground objects. 
	- specifically at nine crucial points that define the object's bounding box in BEV
		- The four corners of the bounding box
		- The midpoints of the four edges of the bounding box
		- The center of the bounding box.
	- These nine points are chosen because they represent key locations on the object that are important for understanding its shape and position

![photo](/hedwig_explains/img/unidistll/feature_distill.png)

 - When the teacher performs worse than the student (e.g., when the teacher is camera-based and the student is LiDAR-based), directly using feature distillation can degrade performance.
 - To prevent this, an adaptive layer is introduced when using feature distillation in such scenarios
 - To address this, they introduce an adaptive layer Adapt_1, which is a one-layer convolutional network, after F_MS^low to produce new features F̂_MS^low. The feature distillation loss is then calculated using F̂_MS^low and F_MT^low.

### Relation Distillation
- High-level BEV features `F^high` encapsulate more abstract information about the scene's structure, which is crucial for making accurate predictions about the locations and orientations of objects.
 - Relation distillation also focuses on the nine crucial points of a ground truth bounding box (its corners, midpoints of edges, and center)
- A relation matrix `RelMat` is computed to represent the relationship between the features of these crucial points.
- The matrix `RelMat_MS` for the student is calculated by applying a cosine similarity function `Φ` to the feature pairs of the nine points, resulting in a 9x9 matrix where each entry `i,j` represents the similarity between the `i`-th and `j`-th crucial points.
   ![photo](/hedwig_explains/img/unidistll/relmat_MS.jpg)

   - Loss (**`L_Rel`**): The relationship matrix `RelMat_MS` is then compared with the corresponding matrix `RelMat_MT` from the teacher
   ![photo](/hedwig_explains/img/unidistll/relation_dist.png)
   - In cases where the student's modality may provide higher performance than the teacher's, directly applying relation distillation could be detrimental. An adaptive layer is introduced after `F^high_MS` to adjust the student's features before the relation distillation process.

### Response Distillation
- Response distillation is part of the UniDistill framework and focuses on aligning the final predictions of a student detector with those of a teacher detector
- High level feature map :
   - The student and teacher detectors produce high-level feature maps.
   - `F^cls` is the classification heatmap with dimensions `HxWxC`, where `H` and `W` are the height and width of the heatmap, and `C` is the number of classes.
   - `F^reg` is the regression heatmap with dimensions `HxWxT`, where `T` is the number of regression targets (such as object dimensions or orientation).
   - A new heatmap, `F^cls_max`, is created by taking the maximum value across the `C` channels at each position `(i,j)` in the classification heatmap `F^cls`. This step simplifies the classification heatmap by focusing on the most confident class predictions.
   - The simplified classification heatmap `F^cls_max` is concatenated with the regression heatmap `F^reg` to form a set of response features `F^resp`. These combined features encapsulate all the necessary information for making final object predictions.
- **Loss** - To train the student to emulate the teacher's predictions, the response features of the student (`F^resp_MS`) are compared to those of the teacher (`F^resp_MT`).
![photo](/hedwig_explains/img/unidistll/relation_dist.png) 
  - However, instead of using all the values in the heatmaps, which might include misalignments especially in the background, the comparison is focused on the areas near the center of ground truth bounding boxes
  - A Gaussian-like mask is generated around the center of each ground truth bounding box, to highlight the region of interest.
  - The response distillation loss is calculated by summing the product of the absolute differences in response features and the values of the Gaussian mask over all positions `(i,j)` in the heatmap
- Masking ? - The Gaussian-like mask is key because it focuses the distillation process on areas relevant to object detection, rather than trying to match features over the entire image where there might be no objects


### Combined loss 

detection loss `L_Det` of the student with the distillation losses as the total loss `L_Total`
![photo](/hedwig_explains/img/unidistll/loss.png) 
where λ 1 , λ 2 and λ 3 are hyperparameters used to balance the scale of different losses.


### Experiments
- BEVDet + Resnet50 = Camera based detector
- CenterPoint  = LiDAR based detector
Training of detector follow some parameters
- Optimizer : AdamW ; 
  - Lr = 1e-4 (for Lidar or LiDAR-camera)
  - Lr = 2e-4 (for camera based detector)
- Batch size : 20
- Epochs     : 20

Unidistill evaluated on ability to transfer knowledge in 4 distillation paths :
1. From the LiDAR-camera based teacher detector to the LiDAR based student.
2. From the LiDAR-camera based teacher detector to the
camera based student.
3. From the camera based teacher detector to the LiDAR based student.
4. From the LiDAR based teacher detector to the camera based student.

The hyperparameters λ 1 ,λ 2 and λ 3 in each path are: 
1. 10, 1, 10.
2. 10, 5, 10. 
3. 10, 5, 1. 
4. 100, 40, 10


#### When compared with SOTA 
![photo](/hedwig_explains/img/unidistll/sota_table1.png) 
- UniDistill improves the performance (mAP and NDS) of single-modality detectors in all four distillation paths: LiDAR-to-camera (L-C), camera-to-LiDAR (C-L), fusion-to-LiDAR (L+C-L), and fusion-to-camera (L+C-C).
- The LiDAR-based student (L) achieves better performance with UniDistill (65.4% mAP, 70.6% NDS) compared to other state-of-the-art LiDAR-based detectors like CVCNet, Guided 3DOD, and AFDetV2.
- UniDistill outperforms S2M2-SSD, another cross-modality knowledge distillation method, in the LiDAR-to-LiDAR (L-L) distillation path.
- The camera-based student (C) generally shows lower performance compared to the LiDAR-based student (L), but still benefits from UniDistill in both the LiDAR-to-camera (L-C) and fusion-to-camera (L+C-C) distillation paths.

### Ablation Studies
Settings and Results: The study tested various combinations of three distillation losses: Feature Distillation (LFea), Relation Distillation (LRel), and Response Distillation (LResp).
![photo](/hedwig_explains/img/unidistll/ablation_studies_settings.png) 

![photo](/hedwig_explains/img/unidistll/albation_std.png) 

##### Effect of each distillation loss
- The feature distillation, relation distillation, and response distillation losses are analyzed individually and in combination.
- Each distillation loss contributes to the overall performance improvement of the student detector.
- The three distillation losses are complementary, and using them together yields the best results.

#### Rationale for selecting 9 crucial points
- For feature distillation, aligning features at 9 crucial points performs better than aligning features completely or inside a Gaussian-like mask.
- For relation distillation, aligning the relationship between 9 crucial points performs better than aligning the relationship between all high-level BEV features or features inside a Gaussian-like mask.

#### Effectiveness of adaptive layers
- Adaptive layers (Adapt_1 and Adapt_2) are introduced in the camera-to-LiDAR distillation path to avoid performance degradation.
- Ablation studies show that removing these adaptive layers leads to worse performance, confirming their indispensability in this scenario.

#### Effectiveness of aligning low-level and high-level BEV features
- For feature distillation, aligning low-level BEV features performs better than aligning high-level BEV features.
- For relation distillation, aligning the relationship between high-level BEV features yields better results than aligning the relationship between low-level BEV features.

# Conclusion

UniDistill projects the features of both the teacher and student into a unified BEV domain and then calculates three distillation losses to align the features for knowledge transfer. Furthermore, it enables UniDistill supports LiDAR-to-camera, camera-to-
LiDAR, fusion-to-LiDAR and fusion-to-camera distillation paths which increases the flexibility of the method proposed. 


