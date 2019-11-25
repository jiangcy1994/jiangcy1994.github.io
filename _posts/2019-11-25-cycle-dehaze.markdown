---
layout: default
title:  "CycleDehaze日记"
date:   2019-11-25 22:00:00
categories: network
---

# CycleDehaze

CycleDehaze 是CycleDehaze改形，映入了预训练模型vgg16中的特征(第二块的池化层，第五块的池化层)。

文章： [Cycle-Dehaze: Enhanced CycleGAN for Single Image Dehazing](https://arxiv.org/abs/1805.05308)

# CycleDehaze理论

图片源域：$X$, 图片目标域$Y$

正映射$G$：$G:X \rightarrow Y$, 负映射$G$：$F:Y \rightarrow X$

## 模型构成

该模型由两个判别器和生成器构成

两个判别器可标记为 $D_X$, $D_Y$，分别对图片是否属于A或者B进行判断

两个生成去可标记为 $G$， $F$，分别为从$X$到$Y$和从$Y$到$X$的映射

## 目标函数

$L_{GAN}(F, D_X, Y, X)=E_{x \sim p_{data}(x)}[\log D_X(x)]+E_{y\sim p_{data}(y)}[\log (1-D_X(F(y)))]$

$L_{GAN}(G, D_Y, X, Y)=E_{y \sim p_{data}(y)}[\log D_Y(y)]+E_{x\sim p_{data}(x)}[\log (1-D_Y(G(x)))]$

$L_{cyc}(G, F)=E_{y\sim p_{data}(y)}[\|\|F(G(x))-x\|\|]+E_{y\sim p_{data}(y)}[\|\|G(F(y))-y\|\|]$

$L_{Perceptual}=\|\|\phi(x) - \phi(F(G(x)))\|\|+\|\|\phi(y) - \phi(G(F(y)))\|\|$

$L(G, F, D_X, D_Y) =L_{GAN}(G, D_Y, X, Y) + L_{GAN}(F, D_X, Y, X)+ \lambda L_{cyc}(G, F) + \gamma L_{Perceptual}(G, F)$

$G^\star, F^\star=arg\, \underset {G,F}{\min} \underset{D_X,D_Y}{\max} L(G, F, D_X, D_Y)$

# CycleGAN实现

## 工具函数

``` python
from functools import reduce

def compose(*funcs):
    if funcs:
        return reduce(lambda f, g: lambda *a, **kw: g(f(*a, **kw)), funcs)
    else:
        raise ValueError('Composition of empty sequence not supported.')

```

## CycleDehaze判别器

同CycleGAN

``` python
num_channels_in
num_discriminator_filter
max_layers=3
use_sigmoid=True

img_input = Input(shape=(None, None, num_channels_in), name='img_input')

layers = compose(
    Conv2D(num_discriminator_filter, kernel_size=4, strides=2, padding="same", name='first_conv'),
    LeakyReLU(alpha=0.2, name='first_lrelu')
)

for layer_idx in range(1, max_layers):
    conv_filter = num_discriminator_filter * min(2**layer_idx, 8)
    name_prefix = 'pyramid.{0}'.format(layer_idx)
    layers = compose(
        layers,
        Conv2D(conv_filter, kernel_size=4, strides=2, padding="same", use_bias=False, name=name_prefix + '_conv'),
        BatchNormalization(momentum=0.9, epsilon=1.01e-5, name=name_prefix + '_bn'),
        LeakyReLU(alpha=0.2, name=name_prefix + '_lrelu')
    )

conv_filter = num_discriminator_filter * min(2**max_layers, 8)
layers = compose(
    layers,
    Conv2D(conv_filter, kernel_size=4, strides=2, padding="same", use_bias=False, name='pyramid_last_conv'),
    BatchNormalization(momentum=0.9, epsilon=1.01e-5, training=1, name='pyramid_last_bn'),
    LeakyReLU(alpha=0.2, name='pyramid_last_lrelu')
)

# final layer
layers = compose(
    layers,
    Conv2D(1, kernel_size=4, name='final_conv', padding='same', activation='sigmoid' if use_sigmoid else None)
) 

discriminator_model = Model(inputs=[img_input], outputs=[layers(img_input)])
```

## CycleDehaze生成器

同CycleGAN

``` python
img_size
num_channel_in=3
num_channel_out=3
num_generator_filter=64
fixed_input_size=True
max_num_filter = 8 * num_generator_filter

def block(x, size, num_filter_in, use_batchnorm=True, num_filter_out=None, num_filter_next=None):
    assert size >= 2 and size % 2 == 0
    if num_filter_next is None:
        num_filter_next = min(num_filter_in*2, max_num_filter)
    if num_filter_out is None:
        num_filter_out = num_filter_in
    x = Conv2D(num_filter_next, kernel_size=4, strides=2, use_bias=(not (use_batchnorm and size > 2)),
                padding='same', name='conv_{0}'.format(size)
                )(x)
    if size > 2:
        if use_batchnorm:
            x = BatchNormalization(momentum=0.9, epsilon=1.01e-5)(x, training=1)
        x2 = LeakyReLU(alpha=0.2)(x)
        x2 = block(x2, size//2, num_filter_next)
        x = Concatenate()([x, x2])            
    x = Activation("relu")(x)
    x = Conv2DTranspose(num_filter_out, kernel_size=4, strides=2, use_bias=not use_batchnorm,
                        name='convt.{0}'.format(size))(x)        
    x = Cropping2D(1)(x)
    if use_batchnorm:
        x = BatchNormalization(momentum=0.9, epsilon=1.01e-5)(x, training=1)
    if size <= 8:
        x = Dropout(0.5)(x, training=1)
    return x

size = img_size if fixed_input_size else None
t = inputs = Input(shape=(size, size, num_channel_in))        
t = block(t, img_size, num_channel_in, False, num_filter_out=num_channel_out, num_filter_next=num_generator_filter)
t = Activation('tanh')(t)
generator_model = Model(inputs=inputs, outputs=[t])
```

## VGG16 Feature

```python
from keras.models import Model

__all__ = ['pretrained_net_feature']

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

``` python
from keras.applications import vgg16
from pretrained_net_feature import *

__all__ = ['vgg16_feature_net']

vgg16_feature_net = lambda img_shape=(256, 256, 3): pretrained_net_feature(
    pretrained_net=vgg16.VGG16,
    output_layers=['block2_pool', 'block5_pool'],
    input_shape=img_shape
)

```
