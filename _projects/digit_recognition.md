---
layout: project
title: 'Written Digit Recognition'
caption: Written Digit Recognition application
description: >
  Written Digit Recognition application 
date: 1 Apr 2020
image: 
  path: /assets/img/projects/digit_v4.png
links:
  - title: Link
    url: https://github.com/EkaterinaKasilina/written_digit_recognition?tab=readme-ov-file
#accent_color: '#4fb1ba'
#accent_image:
#  background: /assets/img/projects/digit_v4.png
#theme_color: '#193747'
sitemap: false
---

Small project for classic problem Written Digit Recognition but with Flask and Canvas.   

# Models
I use two models in this project:  
1. Feed forward neural network (FNN) is written on numpy and is not very accurate.  
2. Convolutional neural network is created with PyTorch and is much more accurate.  
The models were trained on MNIST dataset.  

# FeedForward neural network
This model has the simplest architecture:  
- Input layer size is 28*28  
- Hidden layer size is 100  
- Output layer size is 10  

# FNN  details
- Weights are initialized using Xavier weight initialization;  
- Loss method performs forward and backward passes and computes loss with l2 regularization;  
- The model is trained using stochastic gradient descent.  

![](https://raw.githubusercontent.com/EkaterinaKasilina/written_digit_recognition/refs/heads/master/static/images/conf_matrix_fnn.png)
FNN Confusion matrix
{:.figcaption}

# Convolutional neural network with PyTorch
- Weights are initialized using Xavier weight initialization;  
- During training dropout is applied to conv layers and first fully connected layer;  

![](https://raw.githubusercontent.com/EkaterinaKasilina/written_digit_recognition/refs/heads/master/static/images/conf_matrix_fnn.png)
CNN Confusion matrix
{:.figcaption}