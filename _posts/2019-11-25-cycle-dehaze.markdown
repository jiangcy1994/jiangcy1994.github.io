---
layout: default
title:  "CycleDehaze日记"
date:   2019-11-25 22:00:00
categories: network
---

# CycleDehaze

CycleDehaze 是CycleDehaze改形，映入了预训练模型vgg16中的特征(第二块的池化层，第五块的池化层)。

文章： [Cycle-Dehaze: Enhanced CycleGAN for Single Image Dehazing](https://arxiv.org/abs/1805.05308)

[文章对应网站](https://github.com/engindeniz/Cycle-Dehaze)

# 理论模型

图片源域：$X$, 图片目标域$Y$

正映射$G$：$G:X \rightarrow Y$, 负映射$G$：$F:Y \rightarrow X$

VGG16特征提取函数：$\phi$

## 模型构成

该模型由两个判别器和生成器构成

两个判别器可标记为 $D_X$, $D_Y$，分别对图片是否属于A或者B进行判断

两个生成去可标记为 $G$， $F$，分别为从$X$到$Y$和从$Y$到$X$的映射

## 目标函数

$G^\star, F^\star=arg\, \underset {G,F}{\min} \underset{D_X,D_Y}{\max} L(G, F, D_X, D_Y)=L_{GAN}(G, D_Y, X, Y) + L_{GAN}(F, D_X, Y, X)+ \lambda L_{cyc}(G, F) + \gamma L_{Perceptual}(G, F)$
* $L_{GAN}(F, D_X, Y, X)=E_{x \sim p_{data}(x)}[\log D_X(x)]+E_{y\sim p_{data}(y)}[\log (1-D_X(F(y)))]$
* $L_{GAN}(G, D_Y, X, Y)=E_{y \sim p_{data}(y)}[\log D_Y(y)]+E_{x\sim p_{data}(x)}[\log (1-D_Y(G(x)))]$
* $L_{cyc}(G, F)=E_{y\sim p_{data}(y)}[\|\|F(G(x))-x\|\|]+E_{y\sim p_{data}(y)}[\|\|G(F(y))-y\|\|]$
* $L_{Perceptual}=\|\|\phi(x) - \phi(F(G(x)))\|\|+\|\|\phi(y) - \phi(G(F(y)))\|\|$


# 网络实现

## 判别器

同CycleGAN

## 生成器

同CycleGAN

## VGG16特征

```python
from keras.applications import vgg16
from keras.models import Model

__all__ = ['vgg16_feature_net']

vgg16_feature_net = lambda img_shape=(256, 256, 3): pretrained_net_feature(
    pretrained_net=vgg16.VGG16,
    output_layers=['block2_pool', 'block5_pool'],
    input_shape=img_shape
)

def pretrained_net_feature(pretrained_net, output_layers, input_shape=(128, 128, 3)):
    
    assert type(output_layers) in [int, list, str]
    if type(output_layers) in [int, str]:
        output_layers = [output_layers]
        
    pretrained_model = pretrained_net(include_top=False, 
                                          weights='imagenet', 
                                          input_shape=input_shape)
    layer_names = [layer.name for layer in pretrained_model.layers]
    output_list = []
    
    for output_layer in output_layers:
        if type(output_layer) is str:
            assert output_layer in layer_names
            output_layer = layer_names.index(output_layer)

        output_list.append(pretrained_model.layers[output_layer].output)
    
    model = Model(inputs=[pretrained_model.input], outputs=output_list)
    model.trainable = False
    return model

```
