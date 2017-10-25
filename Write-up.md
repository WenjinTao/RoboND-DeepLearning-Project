## Identification and Tracking of Human Target Using Fully Convolutional Networks

[image_fcn]: ./docs/misc/fcn_architecture.png
[image_follow]: ./docs/misc/result_1.png
[image_notarg]: ./docs/misc/result_2.png
[image_patrol]: ./docs/misc/result_3.png

### Introduction

Fully Convolutional Networks (FCNs) are powerful in the [Semantic Segmentation](https://people.eecs.berkeley.edu/~jonlong/long_shelhamer_fcn.pdf) tasks. This project is to identify and track a particular human target in the simulator using FCNs. The relevant techniques can be applied to a wide range of scenarios like advanced cruise control in autonomous vehicles or human-robot collaboration in industry.

### Network Architecture

![][image_fcn]

My model is illustrated in the above figure. The input is an 160x160x3 image. Three encoder blocks are used to extract the spatial feature map with the size getting smaller (160-80-40-20) and the depth getting deeper (3-32-64-128). Then, a 1x1 convolution layer is applied to preserve the spatial information extracted from the three encoders. This layer increases the depth to 256. After that, as the inverse process of encoding, three decoders are used to up-sample the feature map with the size getting larger (20-40-80-160) and the depth getting smaller (256-128-64-32). Finally, a convolution layer is used to generate a three channels mask for identifying the human target.

#### Encoder Block

The encoder block consists of  a separable convolution layer and a batch normalization layer. 

***Separable convolution (or Depthwise separable convolution)*** comprises of a convolution performed over each channel of an input layer and followed by a 1 x 1 convolution that takes the output channels from the previous step and then combines them into an output layer. It is used rather than a regular convolution because it reduces the number of parameters needed, which will increase the efficiency of the encoder network. 

***Batch normalization*** normalizes each layer's inputs by using the mean and variance of the values in the current mini-batch, which will accelerate the training process.

#### 1 x 1 Convolution

The 1 x 1 convolution layer is a regular convolution, which is used to preserve the spatial information.

#### Decoder Block

The decoder block consists of a up-sampling layer, a concatenation layer and two "separable convolution + batch normalization" layers.

***Bilinear interpolation*** is used in the up-sampling layer, which scale the input size up by 2.

***Layer concatenation*** is used to concatenate two layers, the up-sampled layer and a layer with more spatial information than the up-sampled on from the encoding process. It is a great way to retain some of the finer details from the previous layers as we decode. The sizes of the two layers need match up, e.g., 40x40x256 and 40x40x64, and the depths need not. 

***Separable convolution + batch normalization*** layers are used here to extract some more spatial information from the prior layer.

### Hyperparameters

The hyperparameters are chosen by trial and error:

- Epoch is set to 10 for exploration and 100 for more confident training
- Learning Rate is chosen from {0.001, 0.01} as experience
- Batch Size is chosen from {1, 8, 16, 32, 64}

Here's some experimental results:

| Batch Size | Learning Rate | Epoch   | Final Grade Score |
| ---------- | ------------- | ------- | ----------------- |
| 1          | 0.001         | 10      | 0.25              |
| 1          | 0.001         | 100     | 0.297             |
| 8          | 0.001         | 100     | 0.42              |
| 16         | 0.001         | 100     | 0.44              |
| **16**     | **0.01**      | **100** | **0.45**          |
| 32         | 0.001         | 10      | 0.33              |
| 64         | 0.001         | 10      | 0.36              |
| 64         | 0.001         | 100     | 0.40              |

As the training time increases with increasing of the batch size, I choose 16 finally. The learning rate of 0.01 is chosen because it shows better score in compared to 0.001.

### Results

The final grade score is 0.45 which is larger than the rubric limit 0.4. The following figures show the segmentation result, where the target and the other humans are highlighted in blue and green, respectively.

![][image_follow]

![][image_notarg]

![][image_patrol]

### Limitation of the Current Model

Now the model is able to segment humans from the drone's view and identify the target human, who is a woman dressed in red. I think it would not work well for following another object (dog, cat, car, etc.) instead of a human, because the feature of another object like dog are so different with that of a human, and the model did not learn from data including other objects.

If we want to extend the model to identify more objects. First of all, we need more data of those objects. Or, if the objects are already included in some open dataset like COCO, we can use some pre-trained models in the encoding process. After we solve the data issue, the FCN architecture may need to be modified as well.

### Future Enhancements

- Modifying the current FCN architecture is worth a try.
- Collecting more data should improve the model performance to some extent. 
- More hyperparameters tuning could also contribute to the improvement.