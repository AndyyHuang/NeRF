# Neural Radiance Field (NeRF) Implementation
<p align="center"><img width="35%" alt="Screenshot 2023-12-13 at 5 13 17 PM" src="https://github.com/AndyyHuang/NeRF/assets/76765795/e150fa28-4ec4-4e31-b04e-dcc53c0906eb"></p>
<p align="center">A novel rendering of a learned 3D scene.</p>

In this project, I implemented a Neural Radiance Field (NeRF) as outlined in the original paper by Ben Mildenhall et al.

## Implementation Details
<p align="center"><img width="880" alt="Screenshot 2023-12-13 at 5 13 17 PM" src="https://github.com/AndyyHuang/NeRF/assets/76765795/ac67dcbe-2d87-4b59-a1b1-80cfbc7d1264"></p>

### Ray Sampling
As mentioned in the paper, a NeRF is trained with 2D images taken from different perspectives of an object. As you can see in the diagram above, the inputs are randomly sampled 3D world coordinates along rays pointing in the direction that we want to render and the direction of the ray corresponding to the sampled points. During training, these rays are also randomly sampled, thus producing an input dimension of BxRxSx3 where B = number of training images (batch dim), R = number of rays sampled per image, S = Number of sampled points per ray, and 3 = each color channel (RGB). Anyways, without getting too deep into the details, I first had to implement a ray and point sampler which provides input to the model. To speed up the training process, preprocessing was done by precalculating and storing all ray directions and origins in memory, so my sampler would just have to index into the corresponding entry to sample from an image. This would allow me to avoid doing all camera to pixel and camera to world coordinate transforms during training time whenever sampling rays in world space. 

### Training
Again, the inputs to the model are randomly sampled 3D world coordinates and the corresponding ray directions from 2D images. The 3D world coordinates are positionally encoded using sinusoidal encoding. Positional encoding gives a higher dimensional representation of the world coordinates (3D to 63D), which attributes to better representation of high frequency variations in color and geometry.

<p align="center"><img width="722" alt="Screenshot 2023-12-13 at 5 02 24 PM" src="https://github.com/AndyyHuang/NeRF/assets/76765795/44cd2b8a-5867-4f36-a9db-efd5d9fbafdd"></p>
<p align="center">Sinusoidal Encoding</p>

The model yields two outputs: The predicted density and RGB value at that sampled 3D world coordinate. The loss was calculated over the groundtruth pixel RGB values and the output of the discrete approximation of the volumetric rendering equation (using the predicted density and RGB value as the function's inputs). 

<p align="center"><img width="595" alt="Screenshot 2023-12-13 at 5 02 53 PM" src="https://github.com/AndyyHuang/NeRF/assets/76765795/329c3877-75c9-458f-af98-f1d227451edc"></p>
<p align="center">Continuous Volumetric Rendering Equation</p>

<p align="center"><img width="523" alt="Screenshot 2023-12-13 at 5 03 18 PM" src="https://github.com/AndyyHuang/NeRF/assets/76765795/2aac573e-e1a1-4674-9e02-a91172321e0b"></p>
<p align="center">Discrete Volumetric Rendering Equation</p>

I used MSE as the loss function, however both MSE and Peak signal-to-noise ratio (PSNR) were used to evaluate the model's performance. Loss was calculated by taking MSE of the outputs and groundtruth colors.

<p align="center"><img width="254" alt="Screenshot 2023-12-13 at 5 02 32 PM" src="https://github.com/AndyyHuang/NeRF/assets/76765795/47242c64-78be-4b10-ab87-501bd5addc92"></p>
<p align="center">PSNR</p>

I used one image in my validation set to monitor the training progress of the NeRF. The optimizer I used was Adam.

## Results

Below are renders of an image from my validation set.

<p align="center">
  <img width="250" alt="" src="https://github.com/AndyyHuang/NeRF/assets/76765795/0cd527d0-4856-440e-99b2-82a5d2d390be">
  <img width="250" alt="" src="https://github.com/AndyyHuang/NeRF/assets/76765795/154e5e25-8824-4e98-a7f5-01c95dbd485b">
  <img width="250" alt="" src="https://github.com/AndyyHuang/NeRF/assets/76765795/6cff4016-bbde-4848-89b4-ccce09958b1e">
</p>
<p align="center">Epochs 1, 50, 100</p>

<p align="center">
  <img width="250" alt="" src="https://github.com/AndyyHuang/NeRF/assets/76765795/73035cb6-7da5-46a5-9457-6a5984d760b7">
  <img width="250" alt="" src="https://github.com/AndyyHuang/NeRF/assets/76765795/c7dd01c9-2ac1-4da7-bcfc-5fb06894a20d">
  <img width="250" alt="" src="https://github.com/AndyyHuang/NeRF/assets/76765795/adaaaad9-3781-40bf-9bcc-d1dd2c667e10">
</p>
<p align="center">Epochs 200, 400, 800</p>

<p align="center"><img width="250" alt="" src="https://github.com/AndyyHuang/NeRF/assets/76765795/9ac4f2d7-f88f-419a-86c2-2279accf61cd"></p>
<p align="center">Epoch 1000</p>

## Training and Validation Loss
Below are graphs of my learning/training and validation loss and PSNR over 1000 training epochs. As you can see, the model acheives roughly 22.5 PSNR for both the learning/training and validation image(s).

<p align="center"><img width="400" alt="L_loss" src="https://github.com/AndyyHuang/NeRF/assets/76765795/5eea5acb-06df-4c85-82a5-df7473ce42a8">
  <img width="400" alt="L_PSNR" src="https://github.com/AndyyHuang/NeRF/assets/76765795/50aeddaf-ac92-4721-ba23-2249e96b0414"></p>
<p align="center"></p>

<p align="center"><img width="400" alt="v_loss" src="https://github.com/AndyyHuang/NeRF/assets/76765795/694f9605-5ebb-47d0-af07-b4366d8d2b8d">
<img width="400" alt="v_PSNR" src="https://github.com/AndyyHuang/NeRF/assets/76765795/4f1fb160-8334-4c01-8d31-9dba82cc5a24"></p>
<p align="center"></p>

## Novel Rendering Of Test Cameras
Below is a rendering of novel views on the trained model (1000 epochs).

<p align="center"><img width="35%" alt="Screenshot 2023-12-13 at 5 13 17 PM" src="https://github.com/AndyyHuang/NeRF/assets/76765795/e150fa28-4ec4-4e31-b04e-dcc53c0906eb"></p>


