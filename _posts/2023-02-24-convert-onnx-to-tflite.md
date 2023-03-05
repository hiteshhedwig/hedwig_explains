---
layout: post
title: "How to convert Onnx to TFlite model?"
subtitle: "Model conversion"
date: 2023-02-24
author: "Hitesh Kumar"
header-img: "img/convert_to_tflite.jpg"
tags: [model, conversion, tflite, onnx, pytorch]
---

## Idea :

Converting onnx to tflite are crucial in many sense. When you want to ship models for android phone or embedded device. Generally, you might not be tensorflow inclined. You may use different framework like pytorch. But when time comes, you'd want to convert your existing pytorch model converted to tflite without retraining model in tensorflow. 

In this blogpost, i'll be citing out the steps. How to do it actually :

- Make sure you have model converted in ONNX.

## Process :

1. [Clone this repo](https://github.com/hiteshhedwig/to-tflite-edge) 
```
	git clone https://github.com/hiteshhedwig/to-tflite-edge
	git submodule update --init
	./build.sh
```


2. Convert `.onnx` to `.pb`. 
	`onnx-tf convert -i "mobilenetv2.onnx" -o  "mobilenetv2.pb"`

3. Finally run command below by providing path to converted PB model
	`python to_tflite.py "/path/to/pbmodel"


## Heads UP :
-   If you have an onnx model converted from pytorch model. Make sure you exported the onnx model in torch package <= 1.7
-   If you face some errors, try and see if you have non-simplified onnx model version. Simplified onnx version gives trouble
-  If you have custom model, and the post is not able to help you. Please know details of your model and try implementing the specific layer for conversion which is longer way to go.