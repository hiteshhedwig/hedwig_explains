---
layout: post
title: "[Paper review] WinCLIP: Zero-/Few-Shot Anomaly Classification and Segmentation"
subtitle: "Zero-/Few-Shot Anomaly Classification and Segmentation"
date: 2024-04-10
author: "Hitesh Kumar"
header-img: "img/winclip-shot/pexels-joshsorenson-1154510.jpg"
tags: [paper, review, anomaly, zeroshot, fewshot]
---

# Description - 

The idea of anomaly detection is pretty known thoughout the industry and research community. Yet, it remains one of the toughest problems to solve. Particularly in the context of manufacturing defects, where the defects variation is too much and data points are less.

_Disclaimer - No, this is not ChatGPT generated post. I truly want to learn and understand things. However, i have used chatgpt to comprehend some equations and graphs presented in the paper._

CHECK OUT THE PAPER - [Arxiv Link](https://arxiv.org/abs/2303.14814)

## Technical abbreviation - 
1. **AC** - Anomaly Classification
2. **AS** - Anomaly Segmentation
3. **AUROC** - Area Under the Receiver Operating Characteristic
4. **AUPR** - Area Under the Precision-Recall curve
5. **CPE** - Compositional Prompt Ensemble
6. **CLS** - Class Token
7. **CNN** - Convolutional Neural Network
8. **CVPR** - Conference on Computer Vision and Pattern Recognition
9. **FP** - Feature Map before Pooling
10. **FW** - Feature Map from WinCLIP
11. **LAION-400M** - Dataset of 400 million image-text pairs used for CLIP pre-training
12. **MVTec-AD** - MVTec Anomaly Detection Dataset
13. **PRO** - Per-Region Overlap Score
14. **ViT** - Vision Transformer
15. **WinCLIP** - Window-based CLIP for anomaly segmentation


# Problem statement 
- In factories, quality inspection role in finding defects can be long and tedious. Particularly due to wide range of variations makes difficult to specify anomaly. 
- Lack of scalable visual inspection systems.


# Paper discussion

## Introduction 
The paper is dealing in the context of anomaly segmentation and classification using zero/few shot with CLIP vision language model. The Vision language models have shown promise in zero shot classification tasks. 
- "Normal" and "anomalous" are context dependent. For example, a hole in a dress is desirable depending upon the fashion choice.
- Language provides information on the particular type of defect and context to define defect.
- With the pretrained CLIP as a base model, paper shows and verify the hypothesis of language models aids zero/few shot classification and segmentation.
- Paper mentions a challenge "CLIP is trained to enforce cross-modal alignment only on the global embeddings of image and text". it means that CLIP processes whole images and corresponding text descriptions to learn a general or "global" representation of each. 
	- Anomaly segmentation needs pixel level segmentation. For which paper produces WinCLIP. Uses multiple scales.
- Anomaly classification is related to state classification that predicts if an object is normal or anomalous.


## WinCLIP & WinCLIP+

**WinCLIP** : Effective Window based CLIP for efficient zero shot anomaly segmentation.

**WinCLIP++** : Benefit from a few normal reference images, with the context provided by benefits of language guided prediction.

### Anomaly Classification (AC)
Introduce binary zero shot anomaly classification framework CLIP-AC.
- Two class prompts; $o$ is object level label for example "bottle" 
	- normal  [ o ]
	- anomalous [ o ]
- **One-Class Design** : the model uses a text prompt representing the normal state of an object in the image to compute similarity scores.
- **Two-Class Design** : this easily outperforms their one class design. which can make sense because more information to process. 
    - CLIP pretrained by large web dataset provides a powerful representation with good alignment b/w text and images.specific definition about anomaly is necessary for good performance. 
    - for this we use "**Compositional Prompt Ensemble (CPE)**"  basically, multiple descriptions that encapsulate different states of an object (e.g., normal, damaged) and composes them to better capture the various possible anomalies.
    - The prompts for CPE can include lists of state words for all objects and/or specific states for specific objects.
    - The prompts for CPE can include lists of state words for all objects and/or specific states for specific objects. This allows the model to better understand and classify images based on the context provided by the prompts.
- **Compositional Prompt Ensemble** - CPE is different from CLIP prompt ensemble that does not explain object labels (e.g., “cat”) and only augments templates selected by trial-and-error for object classification, including the ones unsuitable for anomaly tasks, e.g., “a cartoon [c]"

### WinCLIP for zero-shot AS


Window-based CLIP (WinCLIP) for zero-shot anomaly segmentation to predict pixel-level anomalies. WinCLIP extracts dense visual features with good language alignment and local details for x, followed by applying ascore_0 spatially to obtain the anomaly segmentation map. 
- **Creating Sliding Windows**: Imagine dividing the image into overlapping square areas (windows). 
- **Extracting Features**: For each highlighted section, WinCLIP uses its image understanding capabilities (the encoder `f`) to extract important features. 
- This is like summarizing what's important or notable in each window, which might include shapes, textures, or patterns.

![photo](/hedwig_explains/img/winclip-shot/feature_extraction.png)

- the idea of Harmonic aggregation of windows, Each local window in an image is assigned an anomaly score, which is the similarity between the window's feature and the text embeddings from a compositional prompt ensemble.
   - each pixel in an image gets an anomaly score from various overlapping windows. A lower score (close to zero) suggests that the pixel is likely to be normal (not anomalous), while a higher score suggests an anomaly.
- Harmonic averaging is particularly effective because it gives more importance to lower anomaly scores, which are closer to zero and indicate normality, helping to refine the segmentation result .


![photo](/hedwig_explains/img/winclip-shot/harmonic_mean.png)

In simpler terms, harmonic averaging helps ensure that a few high anomaly scores don't overshadow many low scores, which indicate that a pixel is normal, thus improving the accuracy of identifying truly anomalous areas in the image.

### WinCLIP+ with few-normal-shots

In manycases, the zero shot method does not really work because of the context dependent where the normal and anomaly. For example, for metal nut, an anomaly type labeled as “flipped upside-down”, which can only be identified relatively from a normal image.

WinCLIP+, an extension of WinCLIP. For better and more precise anomaly detections by incorporating normal reference images. 

Reference association, this component in WinCLIP+ uses the reference normal images to generate memory features. These features are then used to enhance anomaly detection by comparing query images against these memories, looking for deviations that might indicate anomalies. 

![photo](/hedwig_explains/img/winclip-shot/winclip-workflow.png)

Three sets of reference memories denoted as R_s, R_m, and R_p are introduced as small-scale, mid-scale, and penultimate features. 
- These are used to help the model differentiate between normal and anomalous patterns

For anomaly segmentation, the model averages the multi-scale predictions from the three types of reference memories.

![photo](/hedwig_explains/img/winclip-shot/avg_multiscale_pred.png)

producing a combined memory feature that takes into account information from all scales.

To perform anomaly classification, we combine the maximum value of M_w and the WinCLIP zero-shot classification score.

- maximum value of M_w score will attribute to spatial features of few-shot references.
- Other score is CLIP knowledge retrieved via language

The overall anomaly score is given by 
![photo](/hedwig_explains/img/winclip-shot/clip_mw_avg.png)

## Experiments

- **Dataset** : MVTec-AD, VisA
- **Evaluation metrics** : 
  - **Classification** : 
    - AUROC
    - AUPR
    - F1 -score at optimal threshold
  - **Segmentation** :
    - pAUROC
    - PRO
    - F1 -score at optimal threshold pixel wise

### Zero-/few-shot anomaly classification

In the table below, paper compare zero-shot and few-normal-shot anomaly classification results with prior works on MVTec-AD and VisA benchmarks.
![photo](/hedwig_explains/img/winclip-shot/table_1_few_zeroshots.png)

#### Zero Shot 
- Model : CLIP-AC
  - labels : {“normal [c]”, “anomalous [c]”}
  - with the prompt ensemble

#### Few-normal-shot
- WinCLIP+ outperforms prior works.


### Zero-/few-shot anomaly segmentation
![photo](/hedwig_explains/img/winclip-shot/table_4_ZS_AS.png)

- No prior works on zero-shot anomaly segmentation.
- WinCLIP+ outperforms in many cases.


### Conclusion 
1. Introduction of a new framework that combines textual descriptions and image data to more accurately identify anomalies.
2. Superior performance of WinCLIP and WinCLIP+ models in anomaly detection tasks using minimal training samples.
3. Potential future improvements through pre-training on industry-specific data.


