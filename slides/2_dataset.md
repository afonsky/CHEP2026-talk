---
zoom: 0.75 
---

# Dataset

<div class="flex gap-8">
  <div class="flex-1">

- Dataset **CIFAR-10**:
    - 50k/10k train/test samples.


- Base model:
    - [Benchopt](https://github.com/benchopt/benchopt)-optimized **ResNet-18** from [paperswithcode.com](https://paperswithcode.com) benchmark ([archived version](http://web.archive.org/web/20250405043955/https://paperswithcode.com/paper/benchopt-reproducible-efficient-and#code));
    - Validation accuracy: 95.55% while trained on augmented sample of 50k examples.


- Model search space generation:
  - Created **500 variations** of the base model by adjusting model hyperparameters:
    - Architecture:
        - Activation function / Optimizer / Convolution filter number;
    - Training:
        - Batch size / Maximum learning rate / L2 regularization parameter.


- Evaluation procedure:
  - Applied the **Robust Model Selection Algorithm** to all 500 model variations<br> on **three different subsets** of the training data of sizes: 10k, 20k, and 30k examples.

</div>
  <div class="w-40 flex items-center">
    <img src="/cifar10_example.jpg" class="h-full object-contain" />
  </div>
</div>