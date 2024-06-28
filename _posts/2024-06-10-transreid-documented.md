---
layout: post
title: "[Code review] Dig into TransReID official code repo"
subtitle: "vision transformer for reid"
date: 2024-06-16
author: "Hitesh Kumar"
header-img: "img/transreid-documented/pexels-eberhardgross-691668.jpg"
tags: [paper, review, clip, 3d, scene]
---


# Description 

This blog post is to revive my blogging habit and post some bits of transreid code that i found interesting. TransReid is a popular paper focused on the concept of reidentification of 
objects using vision transformer. In this post, we'll go into the code review. 

CHECK OUT THE PAPER - [Arxiv Link](https://arxiv.org/abs/2102.04378)


I have created this TransReID documented [repo](https://github.com/hiteshhedwig/TransReID-documented)  

If you look into the repository. You might see structure something like this
```git
.
.
.
.
├── loss
│   ├── arcface.py
│   ├── center_loss.py
│   ├── __init__.py
│   ├── make_loss.py
│   ├── metric_learning.py
│   ├── softmax_loss.py
│   └── triplet_loss.py
├── model
│   ├── backbones
│   │   ├── __init__.py
│   │   ├── resnet.py
│   │   └── vit_pytorch.py
│   ├── __init__.py
│   └── make_model.py
├── processor
│   ├── __init__.py
│   └── processor.py
├── solver
│   ├── cosine_lr.py
│   ├── __init__.py
│   ├── lr_scheduler.py
│   ├── make_optimizer.py
│   ├── scheduler_factory.py
│   └── scheduler.py
├── utils
│   ├── __init__.py
│   ├── iotools.py
│   ├── logger.py
│   ├── meter.py
│   ├── metrics.py
│   └── reranking.py
├── dist_train.sh
├── LICENSE
├── README.md
├── requirements.txt
├── test.py
└── train.py

```

There's varity of dataset to pick from, since `Market1501` dataset was the smallest, i picked it. 

If you are interested, run the following command
```
python train.py --config_file configs/Market/vit_jpm.yml MODEL.DEVICE_ID "('0')" 
```

## Code exploration

Let's understand how `train.py` works out.

basically code is majorly divided into several parts.

1) `make_dataloader` 

2) `make_model`

3) `make_loss`

4) `make_optimizer`

5) `do_train` - main training loop


Where i will be focusing on `make_model` and `make_loss` segments. Which i think is the core to their code structure.

## make_model 

This function before calling does a important step that is creating `TransReID` model baseline.

`make_model` will go like this :

```python
def func()
    # intializes transreid baseline
    # adds 4 classifier layer for scores 
    # adds 4 transformer blocks to get global feature
    # and local feature
    # later, return scores and features
```

### TransReID

We will look into it in two ways. First is `Init`, Second is `Inference`.

#### Setting up

```python
class TransReID(nn.Module):
    """ Transformer-based Object Re-Identification
    """
    def __init__(self, img_size=224, patch_size=16, stride_size=16, in_chans=3,             num_classes=1000, embed_dim=768, depth=12,
                 num_heads=12, mlp_ratio=4., qkv_bias=False, qk_scale=None, drop_rate=0., attn_drop_rate=0., camera=0, view=0,
                 drop_path_rate=0., hybrid_backbone=None, norm_layer=nn.LayerNorm, local_feature=False, sie_xishu =1.0):
```

According to the paper, image is converted into patch embedding with OVERLAPPING patches. Paper mentions it helps with feature extraction which is essential for reidentification.

Simple conv2d layer is used for the purpose. So basically 
- Stride = Patch Size: No overlap
- Stride < Patch Size: Overlap between patches

For a 224x224 image with 16x16 patches, stride = 8 (50% overlap) results in 27x27 patches

```python
self.proj = nn.Conv2d(in_chans, embed_dim, kernel_size=patch_size, stride=stride_size)
```

Later, SIE embeddings based on the number of cameras and viewpoints are intialized.

Now, most importantly. Transformers blocks are stacked.

```python

self.blocks = nn.ModuleList([
            Block(
                dim=embed_dim, num_heads=num_heads, mlp_ratio=mlp_ratio, qkv_bias=qkv_bias, qk_scale=qk_scale,
                drop=drop_rate, attn_drop=attn_drop_rate, drop_path=dpr[i], norm_layer=norm_layer)
            for i in range(depth)])
```


And weights are initialized with normal distribution function - 
```python
def _init_weights(self, m):
        if isinstance(m, nn.Linear):
            trunc_normal_(m.weight, std=.02)
            if isinstance(m, nn.Linear) and m.bias is not None:
                nn.init.constant_(m.bias, 0)
        elif isinstance(m, nn.LayerNorm):
            nn.init.constant_(m.bias, 0)
            nn.init.constant_(m.weight, 1.0)
```

#### Inference

For inference, lets look at the `forward_features`.  

```python
def forward_features(self, x, camera_id, view_id):
    # x image
    # x = patch embedding(x)
    # append class token to x
    # append camera and view to x
    # dropout
    # run down to transformer blocks
```

Considering `x` to be image with tensorsize :

```
torch.Size([16, 3, 256, 128])
```
Next, when you do the patch embedding. the output is `torch.Size([16, 128, 768]`.

Now, after adding class token - `torch.Size([16, 129, 768])`

the overall vector should look something like this -
```
[cls_token, patch_1, patch_2, ..., patch_n]
```

Final step is transformer blocks. If local_feature is enabled in TransReID. The last transformer block is avoided. 

```python
        if self.local_feature:
            # For tasks requiring local feature context, 
            # except last block, all the blocks are considered
            for blk in self.blocks[:-1]:
                # torch.Size([16, 129, 768])
                x = blk(x)
            return x
        else:
            # For tasks requiring global context, 
            # the entire sequence of blocks is processed to leverage the final
            print("all features ")
            for blk in self.blocks:
                x = blk(x)
            x = self.norm(x)
            return x[:, 0]
```

### TransReID extention

Now with TransReID the baseline is defined. Additional layers are added in `build_transformer_local`.

Last block(global feature provider) is taken out from the baseline(`TransReID`).

```python

block = self.base.blocks[-1]
layer_norm = self.base.norm
self.b1 = nn.Sequential(
    copy.deepcopy(block),
    copy.deepcopy(layer_norm)
)
self.b2 = nn.Sequential(
    copy.deepcopy(block),
    copy.deepcopy(layer_norm)
)
```

Further, `bottleneck_1`, `bottleneck_2` ... `bottleneck_4` is defined. Which are basically
`nn.BatchNorm1d(self.in_planes)` layers.

And to get the scores from several layers of classifier initialized.  

```python

self.classifier = nn.Linear(self.in_planes, self.num_classes, bias=False)
self.classifier.apply(weights_init_classifier)
self.classifier_1 = nn.Linear(self.in_planes, self.num_classes, bias=False)
self.classifier_1.apply(weights_init_classifier)
self.classifier_2 = nn.Linear(self.in_planes, self.num_classes, bias=False)
self.classifier_2.apply(weights_init_classifier)
self.classifier_3 = nn.Linear(self.in_planes, self.num_classes, bias=False)
self.classifier_3.apply(weights_init_classifier)
self.classifier_4 = nn.Linear(self.in_planes, self.num_classes, bias=False)
self.classifier_4.apply(weights_init_classifier)
```


#### Inference 

The forward function of `build_transformer_local` is as follows.

```python
def forward(self, x, label=None, cam_label= None, view_label=None):
    # inference through transreid baseline
    # get the global branch feature using `self.b1`
    # apply JPM branch and shuffling
    # infer local features from `self.b2` like `b1_local_feat`, `b2_local_feat`...
    # run these down through `self.bottleneck_1`, 2,3,4 respectively,
    # now get classifier scores.
```

`x` is the image inferes through `self.base` TransReID model to get features.

Later when features are applied through JPM branch. According to the paper,
this basically shuffles the patches for better reidentification.

4 different set of local features are created with the patch length based on `feature_length // self.divide_length`.

Finally, classifier class score is created. 

```python
# cls_score - torch.Size([16, 751]
cls_score = self.classifier(feat)
# Classification scores for the local features
# global_feat - torch.Size([16, 768]
# cls_score_1,2,3,4 - torch.Size([16, 751])
cls_score_1 = self.classifier_1(local_feat_1_bn)
cls_score_2 = self.classifier_2(local_feat_2_bn)
cls_score_3 = self.classifier_3(local_feat_3_bn)
cls_score_4 = self.classifier_4(local_feat_4_bn)
```

And return the classifier score and features in the specific structure.


```python

return [cls_score, cls_score_1, cls_score_2, cls_score_3,
            cls_score_4], 
            [global_feat, local_feat_1, local_feat_2, local_feat_3,
            local_feat_4]  # global feature for triplet loss
```

## make_loss

Now, lets understand how does this `make_loss` function works. If you look at `loss/make_loss.py` file. You would be able to recognize the mess.

Market1501 `yml` config file specifies, where loss function is combination of cross entropy and triplet loss.  
Their `def make_loss` function goes like :

```python

def make_loss(cfg, num_classes) :
   # if config mentions triplet loss. Initialize triplet loss
   # if config mentions cross entropy. Initialize cross entropy
 
   # if config mentions `cross_entropy`
   # create and return inline function def loss_func(...)
   
   # if sampler config mentions `softmax_triplet`
   # create inline function def loss_func(...)
   # lots of things going on in def loss_func(...)
```

#### what are things going on in def loss_func(...) when sampler is "softmax_triplet"

- Checks if config asks for labelsmoothing on cross entropy or not.
- Applies relevant cross entropy loss also known as ID_LOSS.
- Applies triplet loss
- Takes weighted average of ID_LOSS and Triplet_loss

lets try and understand this part of code snippet -
```python

if isinstance(score, list):
    print("Score is a list. Calculating ID loss without label smoothing.")
    ID_LOSS = [F.cross_entropy(scor, target) for scor in score[1:]]
    ID_LOSS = sum(ID_LOSS) / len(ID_LOSS)
    ID_LOSS = 0.5 * ID_LOSS + 0.5 * F.cross_entropy(score[0], target)
else:
    print("Score is not a list. Calculating ID loss without label smoothing.")
    ID_LOSS = F.cross_entropy(score, target)

if isinstance(feat, list):
    print("Feat is a list. Calculating triplet loss.")
    TRI_LOSS = [triplet(feats, target)[0] for feats in feat[1:]]
    TRI_LOSS = sum(TRI_LOSS) / len(TRI_LOSS)
    TRI_LOSS = 0.5 * TRI_LOSS + 0.5 * triplet(feat[0], target)[0]
else:
    print("Feat is not a list. Calculating triplet loss.")
    TRI_LOSS = triplet(feat, target)[0]
```

If you closely look, you'd be like what are they doing with this - 
`F.cross_entropy(scor, target) for scor in score[1:]` and `F.cross_entropy(score[0], target)`. Similarly in triplet loss.


In our model, we are returning. 

```python

return [cls_score, cls_score_1, cls_score_2, cls_score_3,
            cls_score_4], 
            [global_feat, local_feat_1, local_feat_2, local_feat_3,
            local_feat_4]  # global feature for triplet loss
```

So the loss function, seperately calculates specific loss for `score[0]` and `score[1:]`. Which basically means first we calculate loss for global feature and then loss for local features. Then averaging it.

That's all.

Why do they do it? Probably to balance out the influence of local and global feature during training equally.

