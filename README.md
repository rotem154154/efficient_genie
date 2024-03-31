
# efficient_genie
![Efficient Genie Demo](https://github.com/rotem154154/efficient_genie/blob/main/Untitled2.gif?raw=true)

I developed a significantly smaller world model inspired by Google's **[Genie](https://sites.google.com/view/genie-2024/)**. Trained on an 8-hour video using a MacBook, this model boasts 300 times fewer parameters than Genie and operates in real-time. Mimicking Genie's structure, it is divided into three main components:

**Video Tokenizer**: This module's goal is to extract features from the T preceding frames. Unlike Genie, which employs a VQ-VAE based on the ST-transformer for effective encoding with temporal and spatial attention, my approach initially faced challenges with a more efficient convolution-based VQ-VAE. To address this, I employed a two-stage strategy: firstly, training a decoder to generate images from the video frames' distribution, and secondly, training an encoder to map a single frame into the w latent space. This vector feeds into the previously trained decoder to accurately recreate the frame. For this, I utilized **[StyleGAN 2](https://github.com/rosinality/stylegan2-pytorch)**, subsequently distilling it into a lighter and faster **[MobileStyleGAN](https://github.com/bes-dev/MobileStyleGAN.pytorch)**, which is compatible with MPS devices on Mac. The encoder of choice was **[EfficientVit](https://github.com/mit-han-lab/efficientvit)**, selected for its proven effectiveness in classification, segmentation, and segment anything. The inversion stage aims to minimize the mse and LPIPS distance between the generated image and the actual frame.

**Latent Action Model**: The purpose here is to infer latent actions from pairs of frames, which the dynamics model then uses to predict the video's subsequent frame. Moving away from Genie's use of an ST-transformer, which seemed excessive, I opted for a simpler MLP. This MLP processes the latent representations of current and subsequent frames, encoding them into a compact vector. To achieve discrete values, I quantized this vector into 8 codes with **[vector-quantize-pytorch](https://github.com/lucidrains/vector-quantize-pytorch)**.

**Dynamics Model**: Mirroring the structure of Genie's dynamics model, my version employs a decoder-only **[Transformer](https://github.com/lucidrains/x-transformers)**. This transformer accepts 16 individually encoded frames and their associated actions as input, subsequently predicting the next latent w. This prediction facilitates the generation of the upcoming frame by the decoder.

### 
![genie training](https://github.com/rotem154154/efficient_genie/blob/main/genie%20training.png?raw=true)

----------

While this approach yields excellent results for the first generated frame, subsequent frames—conditioned on these generated frames—do not perfectly match the distribution of the original frames. This discrepancy leads to progressively poorer generations, as each one serves as the input for future frames. To address this issue, I adjusted the training process of the StyleGAN inversion to use generated images as inputs instead of the original video frames. Additionally, I introduced a new component to the loss function: minimizing the cosine similarity between the encoded generated frame and the target frame. This adjustment helps align the distribution of the generated frames more closely with that of the original frames, improving the coherence and quality of sequential frame generation.

![Inversion](https://github.com/rotem154154/efficient_genie/blob/main/inversion.png?raw=true)
