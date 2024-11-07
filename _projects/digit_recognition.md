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

When I first started out as a data scientist, I really wanted to take a project from start to finish. 
I also wanted to show my friends and colleagues just how cool machine learning could be. 
So, I built an app with Flask and Canvas and hosted it on Heroku(until 2022)


The full code can be found [here](https://github.com/EkaterinaKasilina/written_digit_recognition?tab=readme-ov-file)
*The project is old and code is not very clean and good in general

# Models
For this project I use two models:  
1. Feed forward neural network (FNN) is written on numpy and is not very accurate.  
2. Convolutional neural network is created with PyTorch and is much more better.  
The models are trained on MNIST dataset.  

# FeedForward neural network
This model has very simple architecture:  
- Input layer size is 28*28  
- Hidden layer size is 100  
- Output layer size is 10  

## FNN  details
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