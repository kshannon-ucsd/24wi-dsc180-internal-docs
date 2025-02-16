---
title: "Analytics Team Documentation"
date: 2025-02-16
---

# X-Ray Classification Model Documentation
## Overview
This document provides technical details about our ResNet50 Transfer Learning approach for extracting meaningful features from chest X-Rays and provide a binary classification to find lungs with pulmonary issues. 

## Model Architecture

### Input Layers
- **Image Input**: (224, 224, 3) - 3 Channel X-ray images

### CNN Architecture

#### ResNet50 Layers
1. Stage 1
   - Conv1 (7x7, 64 filters, stride = 2)
   - Batch Norm
   - ReLU
   - Max Pooling

2. Stage 2
- 1st Bottleneck Block:  
  - 1x1 Conv (64 filters)  
  - Batch Norm, ReLU  
  - 3x3 Conv (64 filters, stride = 1)  
  - Batch Norm, ReLU  
  - 1x1 Conv (256 filters)  
  - Batch Norm  
  - Residual Connection  
  - ReLU  
- 2 more identical bottleneck blocks (without stride change)  

3. Stage 3
- 1st Bottleneck Block:  
  - 1x1 Conv (128 filters, stride = 2)  
  - Batch Norm, ReLU  
  - 3x3 Conv (128 filters)  
  - Batch Norm, ReLU  
  - 1x1 Conv (512 filters)  
  - Batch Norm  
  - Residual Connection  
  - ReLU  
- 3 more identical bottleneck blocks  

4. Stage 4
- 1st Bottleneck Block:  
  - 1x1 Conv (256 filters, stride = 2)  
  - Batch Norm, ReLU  
  - 3x3 Conv (256 filters)  
  - Batch Norm, ReLU  
  - 1x1 Conv (1024 filters)  
  - Batch Norm  
  - Residual Connection  
  - ReLU  
- 5 more identical bottleneck blocks  

5. Stage 5
- 1st Bottleneck Block:  
  - 1x1 Conv (512 filters, stride = 2)  
  - Batch Norm, ReLU  
  - 3x3 Conv (512 filters)  
  - Batch Norm, ReLU
  - 1x1 Conv (2048 filters)  
  - Batch Norm  
  - Residual Connection  
  - ReLU  
- 2 more identical bottleneck blocks  

6. Final Layers
- Global Average Pooling  
- Fully Connected (1000 units for ImageNet)

#### Added Layers
1. Flatten Layer
2. ReLU
 - Added l2 regularization
3. Dropout (0.3)
4. ReLU
5. Dropout (0.2)
6. Sigmoid Activation

## Training Configuration

### Optimizer
- Adam optimizer
- Cosine Annealing Scheduler
- Initial Learning rate: 0.0001

### Loss Function
- Binary Cross Entropy

### Metrics
- Accuracy
- Validation Loss

### Training Parameters
- Batch size: 16
- Maximum epochs: 50
- Early stopping patience: 10 epochs
- Model checkpointing: Saves best model based on validation loss

## Model Usage

### Creating the Model
```python
import numpy as np
from tensorflow.keras.layers import Dense, GlobalAveragePooling2D, Dropout, Flatten
from tensorflow.keras.regularizers import l2
from tensorflow.keras.models import Sequential
from tensorflow.keras.optimizers import Adam
from tensorflow.keras.callbacks import EarlyStopping, ModelCheckpoint
from tensorflow.keras.applications import ResNet50V2
from tensorflow.keras.optimizers.schedules import CosineDecayRestarts

# Compute class weights
from sklearn.utils.class_weight import compute_class_weight

class_weights = compute_class_weight('balanced', classes=np.unique(y_train), y=y_train)
class_weights_dict = dict(enumerate(class_weights))

# Load ResNet50V2
resnet50 = ResNet50V2(include_top=False, weights='imagenet', input_shape=(224, 224, 3))

# Unfreeze deeper layers for fine-tuning
for layer in resnet50.layers[-10:]:
    layer.trainable = True

# Define the model
model = Sequential([
    resnet50,
    Flatten(),
    Dense(256, activation="relu", kernel_regularizer=l2(1e-4)), 
    Dropout(0.3),
    Dense(128, activation="relu"),
    Dropout(0.2),
    Dense(1, activation="sigmoid")
])

# Implement Cosine Annealing
initial_lr = 1e-4
cosine_decay_restarts = CosineDecayRestarts(initial_learning_rate=initial_lr,
                                            first_decay_steps=5000,
                                            t_mul=2.0,  # Increases period after each restart
                                            m_mul=0.8,  # Reduce max learning rate after each restart
                                            alpha=1e-6)

# Compile model with Cosine Annealing scheduler
model.compile(optimizer=Adam(learning_rate=cosine_decay_restarts),
              loss='binary_crossentropy',
              metrics=['accuracy'])

# Callbacks
early_stopping = EarlyStopping(monitor='val_loss', patience=10, restore_best_weights=True, verbose=1)
model_checkpoint = ModelCheckpoint('best_model.keras', monitor='val_loss', save_best_only=True, verbose=1)
```

## Model Features

### Regularization Techniques
1. Dropout layers (0.3, 0.2 rate)
2. L2 regularization in dense layer
3. Batch normalization
4. Early stopping to prevent overfitting

## Model Output
- Binary classification (0 or 1)
- Probability score through sigmoid activation

## Checkpointing
- Best model weights saved to: `results/models/best_model.h5`
- Monitored metric: Validation loss
- Save best only: True

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
- Other requirements as listed in project's requirements.txt

## Notes
- The model uses a relatively shallow architecture optimized for medical image classification
- Global Average Pooling reduces parameters and helps prevent overfitting
- Position information integration improves model context awareness
- Early stopping and checkpointing ensure optimal model selection