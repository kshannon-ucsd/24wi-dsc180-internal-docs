---
title: "Analytics Team Documentation"
date: 2025-02-16
---

# X-Ray Classification Model Documentation

## Overview
This document provides technical details about our ResNet50 Transfer Learning approach for extracting meaningful features from chest X-rays and performing binary classification to detect pulmonary issues. The model processes X-ray images from a preprocessed directory, where images have been enhanced to improve distinguishable traits. Preprocessing includes image resizing, contrast boosting, and slight angle adjustments to enhance generalizability across variations in patient positioning.

## Model Architecture

### Input Layers
- **Image Input**: (224, 224, 3) - Three-channel X-ray images

### CNN Architecture

#### ResNet50 Layers

1. **Stage 1**
   - **Conv1 (7x7, 64 filters, stride = 2)**
     - Captures broad features across the image
     - 64 filters detect various edges and textures
     - Stride of 2 downsamples the feature map for faster convergence
   - **Batch Normalization**
     - Speeds up convergence by normalizing activations
   - **ReLU Activation**
     - Mitigates vanishing gradients by zeroing out negative values
   - **Max Pooling**
     - Further downsamples the image without losing valuable information

2. **Stage 2**
   - **1st Bottleneck Block:**  
     - 1x1 Conv (64 filters)  
     - Batch Norm, ReLU  
     - 3x3 Conv (64 filters, stride = 1)  
     - Batch Norm, ReLU  
     - 1x1 Conv (256 filters)  
     - Batch Norm  
     - Residual Connection
       - Skips layer weights when weights become too small or too large
       - Allows gradient flow, combating vanishing and exploding gradients  
     - ReLU  
   - **2 more identical bottleneck blocks**  

3. **Stage 3**
   - **1st Bottleneck Block:**  
     - 1x1 Conv (128 filters, stride = 2)  
     - Batch Norm, ReLU  
     - 3x3 Conv (128 filters)  
     - Batch Norm, ReLU  
     - 1x1 Conv (512 filters)  
     - Batch Norm  
     - Residual Connection  
     - ReLU  
   - **3 more identical bottleneck blocks**  

4. **Stage 4**
   - **1st Bottleneck Block:**  
     - 1x1 Conv (256 filters, stride = 2)  
     - Batch Norm, ReLU  
     - 3x3 Conv (256 filters)  
     - Batch Norm, ReLU  
     - 1x1 Conv (1024 filters)  
     - Batch Norm  
     - Residual Connection  
     - ReLU  
   - **5 more identical bottleneck blocks**  

5. **Stage 5**
   - **1st Bottleneck Block:**  
     - 1x1 Conv (512 filters, stride = 2)  
     - Batch Norm, ReLU  
     - 3x3 Conv (512 filters)  
     - Batch Norm, ReLU  
     - 1x1 Conv (2048 filters)  
     - Batch Norm  
     - Residual Connection  
     - ReLU  
   - **2 more identical bottleneck blocks**  

6. **Final Layers**
   - Global Average Pooling  
   - Fully Connected Layer (1000 units for ImageNet)  

#### Added Layers
1. Flatten Layer
2. ReLU Activation
   - L2 regularization applied
3. Dropout (0.3)
   - Reduces learned dependencies by randomly deactivating nodes
4. ReLU Activation
5. Dropout (0.2)
6. Sigmoid Activation
   - Outputs a probability between 0 and 1 for binary classification
   - Most common output for binary classification

## Training Configuration

### Optimizer
- **Adam Optimizer**
  - Adaptive moment estimation accelerates learning when the gradient direction remains consistent
- **Cosine Annealing Scheduler**
  - Modulates the learning rate per epoch along a cosine curve, promoting faster convergence
- **Initial Learning Rate:** 0.0001

### Loss Function
- **Binary Cross Entropy**
  - Optimized for binary classification problems, providing efficient convergence

### Metrics
- Accuracy
- Validation Loss
  - Tracks model performance across epochs

### Training Parameters
- **Batch Size:** 16
- **Maximum Epochs:** 50
- **Early Stopping Patience:** 10 epochs
- **Model Checkpointing:** Saves the best model based on validation loss

## Model Features

### Regularization Techniques
1. Dropout layers (0.3, 0.2 rate)
2. L2 regularization in the dense layer
3. Batch normalization
4. Early stopping to prevent overfitting

## Model Output
- Binary classification (0 or 1)
- Probability score via sigmoid activation

## Checkpointing
- **Best model weights saved to:** `results/models/best_model.h5`
- **Monitored metric:** Validation loss
- **Save best only:** True

## Performance Monitoring
- Training history includes:
  - Loss
  - Accuracy
  - AUC
  - Validation metrics

## Dependencies
- TensorFlow 2.x
- Python 3.x
- NumPy

## Notes
- The model employs a relatively shallow architecture optimized for medical image classification
- Global Average Pooling reduces parameters and mitigates overfitting
- Position information integration enhances model context awareness
- Early stopping and checkpointing ensure optimal model selection
