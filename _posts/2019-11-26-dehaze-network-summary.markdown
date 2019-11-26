---
layout: default
title:  "去雾神经网络汇总"
date:   2019-11-26 23:00:00
categories: network
---

# 去雾神经网络汇总

汇总中的网络：
* [Pix2pix](2019-11-26-pix2pix.markdown)
* [CycleGAN](2019-11-25-cycle-gan.markdown)
* [CycleDehaze](2019-11-25-cycle-dehaze.markdown)
* [DCPDN](2019-11-25-dcpdn.markdown)

# 去雾理论模型

## 物理模型

$I(z)=J(z)t(z)+A(z)(1−t(z))$,

$I$是观测到的有雾的图片，$J$是真实场景，$A$是全局大气光强，$t$是投射图，$z$是像素位置

$\hat{J}(z)=\frac{I(z)-\hat{A}(z)(1-\hat{t}(z))}{\hat{t}(z)}$

## 模型共通点

$D_X$, $D_Y$, 分别为$X$和$Y$的判别器
$G$ $F$，分别为从$X$到$Y$和从$Y$到$X$的映射

## 目标函数对比

| Object Function             | Pix2pix       | CycleGAN        | CycleDehaze     | DCPDN           |
| --------------------------- | ------------- | --------------- | --------------- | --------------- |
| $L_{GAN}(D_X,F)$            | or $L_{cGAN}$ | 1               | 1               | 0               |
| $L_{GAN}(D_Y,G)$            | 0             | 1               | 1               | 0               |
| $L_{cyc}(F\circ G)$         | 0             | $\lambda_{cyc}$ | $\lambda_{cyc}$ | 0               |
| $L_{cyc}(G\circ F)$         | 0             | $\lambda_{cyc}$ | $\lambda_{cyc}$ | 0               |
| $L_{id}(F, X)$              | 0             | $\lambda_{id}$  | $\lambda_{id}$  | 0               |
| $L_{id}(G, Y)$              | 0             | $\lambda_{id}$  | $\lambda_{id}$  | 0               |
| $L_{Perceptgual}(F\circ G)$ | 0             | 0               | $\gamma$        | 0               |
| $L_{Perceptgual}(G\circ F)$ | 0             | 0               | $\gamma$        | 0               |
| $L_{E,f}(G)$                | 0             | 0               | 0               | $\lambda_{E,f}$ |
| $L_{trans}$                 | 0             | 0               | 0               | $L_2$           |
| $L_{overall}$               | $L_1$         | 0               | 0               | $L_2$           |
| $L_{ato}$                   | 0             | 0               | 0               | $L_2$           |
| $L_{E,g}, H \And V$         | 0             | 0               | 0               | $L_2$           |
| $L_{E,f}, H \And V$         | 0             | 0               | 0               | $L_2$           |
| $L_{j}$                     | 0             | 0               | 0               | 1               |
