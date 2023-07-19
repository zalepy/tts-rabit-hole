# tts-rabit-hole
just notes on tts


So commercial solutions exist [ElevenLabs](https://docs.elevenlabs.io/welcome/introduction), but what is the opensource state?

Top seems to be https://github.com/neonbjb/tortoise-tts, and the paper https://arxiv.org/abs/2305.07243

Overall architecture requires 3 NNs:
1. **autoregressive decoder** - text &rarr; speech tokens (many)
2. contrastive model (**CLIP**) pick the best speech tokens (few)
3. **DDPM** - speech tokens &rarr; speech **MEL spectrograms**

* autoregressive decoder is described in [Zero-Shot Text-to-Image Generation](https://arxiv.org/abs/2102.12092) (a.k.a. [DALL-E(Ramesh et al., 2021)](https://github.com/openai/DALL-E))
  * partially based on older paper on autoregressive transformers: [Attention Is All You Need](https://arxiv.org/abs/1706.03762)
  * dVAE = discrete variational autoencoder - have a large, but limited number of small fixed images. Replace your big image with grid of the small images. The small images are the image tokens.\
  \
  The way I understand is: compress (encode) image via dVAE. Concatenate text tokens with the image tokens, then train on that. Resulting model is a decoder-only. Input text &rarr; text tokens, attach the const dVAE image tokens, on all that infer image.
  
  
* CLIP stands for [Constastive Language-Image Pretraining](https://openai.com/research/clip)\
    "_Given an image and text descriptions, the model can predict the most relevant text description for that image, without optimizing for a particular task._"\
    paper: [Learning Transferable Visual Models From Natural Language Supervision](https://arxiv.org/abs/2103.00020)\
    github: https://github.com/openai/CLIP
* DDPM stands for [Denoising Diffusion Probabilistic Models](https://hojonathanho.github.io/diffusion/) ([a.k.a. DDPMs(Ho et al., 2020)](https://arxiv.org/abs/2006.11239))
  * there is older paper - [Generative Modeling by Estimating Gradients of the Data Distribution](https://arxiv.org/abs/1907.05600)\
  \
  The way I understand is: we train on samples that are noised up to different levels, and extract vectors representing the denoise direction. All these vectors become latent space. That space also gives possibility for interpolation.

* MEL spectrograms - like normal audio spectrograms - time on x axis, frequency on y, color heat for intensity (dBFS), but frequency in Mel, rather than kHz. 1 Mel = 1 kHz but Mel is logarithmic, so has better resolution on low frequencies, and also matches how we hear.  

---

So the equivalent for dVAE image tokens in here is VQ-VAE. It's described in paper [Neural Discrete Representation Learning](https://arxiv.org/abs/1711.00937). 
VQVAE differs from VAEs in two key ways: 
1. The encoder network outputs discrete, rather than continuous, codes
2. The prior is learnt rather than static. 

In order to learn a discrete latent representation, we incorporate ideas from vector quantisation (VQ).
\
The way I understand is: In other VAE we produce vectors, that define a latent space. 
In here we **construct** a latent space, and instead of having preset of fixed tokens to encode vectors,
we train an encoder (and decoder) that produces vectors for the constructed latent space. 
During training the error is computed between original and decoded sample, 
so error trains both the encoder and the decoder at the same time.  
