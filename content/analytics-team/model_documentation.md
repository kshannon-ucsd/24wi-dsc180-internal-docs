# X-Ray Classification Model Documentation
Date: 2024-01-21

## Overview
This document provides technical details about our CNN-based X-ray image classification model implementation. The model is designed to process medical X-ray images and provide binary classification results while incorporating positional data.

## Model Architecture

### Input Layers
- **Image Input**: (224, 224, 1) - Grayscale X-ray images
- **Position Input**: (2,) - Encoding for view position information

### CNN Architecture

#### Convolutional Layers
1. First Conv Block
   - Conv2D: 8 filters, 3x3 kernel, ReLU activation
   - BatchNormalization
   - MaxPooling2D
   - Dropout (0.3)

2. Second Conv Block
   - Conv2D: 16 filters, 3x3 kernel, ReLU activation
   - BatchNormalization
   - MaxPooling2D
   - Dropout (0.3)

3. Third Conv Block
   - Conv2D: 32 filters, 3x3 kernel, ReLU activation
   - BatchNormalization
   - GlobalAveragePooling2D
   - Dropout (0.3)

#### Feature Fusion
- Concatenation of CNN features with position data

#### Dense Layers
- Dense (32 units, ReLU, L2 regularization)
- Dropout (0.3)
- Output Dense (1 unit, sigmoid activation)

## Training Configuration

### Optimizer
- Adam optimizer
- Learning rate: 0.001

### Loss Function
- Binary Cross Entropy

### Metrics
- Accuracy
- AUC (Area Under Curve)

### Training Parameters
- Batch size: 32
- Maximum epochs: 50
- Early stopping patience: 5 epochs
- Model checkpointing: Saves best model based on validation accuracy

## Model Usage

### Creating the Model
```python
from src.models.train_model import create_model

# Initialize model
model = create_model(input_shape=(224, 224, 1))
```

### Training the Model
```python
from src.models.train_model import train_model

# Train model
history = train_model(
    model=model,
    train_data=train_dataset,
    val_data=val_dataset,
    epochs=50,
    batch_size=32,
    model_dir='results/models'
)
```

## Model Features

### Regularization Techniques
1. Dropout layers (0.3 rate) after each major block
2. L2 regularization in dense layer
3. Batch normalization
4. Early stopping to prevent overfitting

### Position Information Integration
- The model incorporates X-ray view position information
- Position data is concatenated with image features before final classification

## Model Output
- Binary classification (0 or 1)
- Probability score through sigmoid activation

## Checkpointing
- Best model weights saved to: `results/models/best_model.h5`
- Monitored metric: Validation accuracy
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