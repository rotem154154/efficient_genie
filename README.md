# efficient_genie
![Efficient Genie Demo](https://github.com/rotem154154/efficient_genie/blob/main/Untitled2.gif?raw=true)

I created a much smaller world model based on Google's **[Genie](https://sites.google.com/view/genie-2024/)**. Trained on an 8-hour video on a MacBook, it has 300x fewer parameters and runs in real time. Designed like Genie, it comprises 3 main parts:

**Video Tokenizer**: The purpose of this part is to extract features from the T previous frames. Genie uses a VQ-VAE based on the ST-transformer to encode effectively with temporal and spatial attention. I experimented with a much more efficient VQ-VAE based on convolutions, but the results were not good. Instead, I trained **[StyleGAN 2](https://github.com/rosinality/stylegan2-pytorch)** on frames from the video. Then, I distilled the model into a much lighter **[MobileStyleGAN](https://github.com/bes-dev/MobileStyleGAN.pytorch)**. After that, I created an inversion using **[EfficientVit](https://github.com/mit-han-lab/efficientvit)** to project a single frame into the w latent space that feeds into the MobileStyleGAN to recreate the frame.

**Latent Action Model**: Genie uses the ST-transformer for the action model as well, which is overkill. Instead, I created a simple MLP that takes the latents of the current and next frames and encodes them into a tiny vector. I then quantized this into 8 codes using **[vector-quantize-pytorch](https://github.com/lucidrains/vector-quantize-pytorch)**.

**Dynamics Model**: The dynamics model is similar to Genie's; a decoder-only **[Transformer](https://github.com/lucidrains/x-transformers)** takes as input 16 individually encoded frames and projected actions and predicts the next w that will generate the next frame using the decoder.

![Google Genie](https://dataconomy.com/wp-content/uploads/2024/02/What-is-Google-Genie-and-how-to-use-it_1.jpg)
