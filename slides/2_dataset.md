# Dataset and Validation Approach

<div class="grid grid-cols-[12fr_2fr] gap-10">
<div>

### **CIFAR-10** image classification dataset.

<br>

- Standard dataset for image classification known more than 15 years;
  -  $>95\%$ ($>98\%$) accuracy for modern (pretrained) DL models;
- 10-classes of low-resolution (32x32) images;
- 50k/10k train/test examples.

<br>

<center>
<figure>
    <img src="/validation_scheme.drawio.png" style="width: 550px !important;">
</figure>
</center>
</div>
<div>
<figure>
    <img src="/cifar10_example.jpg" style="width: 195px !important;">
</figure>
</div>
</div>

---

# Baseline Model(s)

- Baseline model:
    - [Benchopt](https://github.com/benchopt/benchopt)-optimized **ResNet-18** from [paperswithcode.com](https://paperswithcode.com) benchmark ([archived](http://web.archive.org/web/20250405043955/https://paperswithcode.com/paper/benchopt-reproducible-efficient-and#code));
    - Validation accuracy: 95.55% while trained on augmented sample of 50k examples.

- Model search space — **~600 models**:
  - **540 variations** of the base model with different hyperparameters:
    - Activation function / Optimizer / # of convolution filters;
    - Batch size / Peak learning rate / L2 regularization parameter.
  - **64 variations** of Compact Convolutional Transformer (ViT alternative for small datasets) with different hyperparameters:
    - Network depth / Batch size / Peak learning rate / Drop path rate.