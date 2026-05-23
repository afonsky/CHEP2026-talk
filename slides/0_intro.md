# Simplified Data Processing Workflow in HEP

<br>
<br>

<center>
<figure>
    <img src="/HEP_data_workflow.drawio.svg" style="width: 960px !important;">
</figure>
</center>

<br>
<br>

<v-clicks at="1" depth="2">

* A machine learning model can be integrated into most of these stages;
    * What is the impact of each ML model on the physics results?
</v-clicks> 

<!--
A simplified diagram illustrating typical stages in experimental particle physics data analyses. After data acquisition that is using a multi-tiered trigger filtering step, the experimental collision data are further reduced in computing processes before they are ready for physics analyses. Events generated following theoretical models undergo a detector simulation step and are subsequently subject to the same reconstruction and processing steps as the collision data. The individual analysts then compare collision and simulated data using statistical analysis techniques. Our approach focuses mostly on the reproducibility challenges of ML models incorporated in the workflow.
-->

---

# Systematic Uncertainties of ML Model

<div class="grid grid-cols-[3fr_4fr] gap-30">
<div>
<v-click at="1">
<figure>
    <img src="/loss_1_trial.png" style="width: 335px !important;">
</figure>
</v-click>
<br>
<br>
<v-click at="2">
<Arrow x1="220" y1="280" x2="220" y2="340" width="3" color="orange" />
<figure>
    <img src="/loss_1000_trials.png" style="width: 335px !important;">
</figure>
</v-click>
</div>
<div>
<v-click at="3">

* The following factors affect the spread:
    * The choice of training and test sets;
    * Intrinsic stochasticity of the model.
</v-click>
<br>
<v-click at="4">
<figure>
    <img src="/randomization_sources.png" style="width: 335px !important;">
    <figcaption style="color:#b3b3b3ff; font-size: 10px"><br>Image credit. Our paper
      <a href="https://ieeexplore.ieee.org/document/11030460">doi:10.1109/ACCESS.2025.3578926</a>
    </figcaption>
</figure>
</v-click>
</div>
</div>

<!--
* Why not apply the standard analysis of systematic uncertainties used in HEP to a machine learning model?
* Suppose an ML model distinguishes reconstructed signal events from background ones.
* By training the model on a training set, we can obtain its average performance on a test set.
    * We can also assess the model’s performance as a function of, for example, the pT of the particle in the event.
* But what is the systematic uncertanty of a ML model?
    * Many mahine-learners will say: use cross validation, and look on the variability among folds
    * Some would say: use the bootstrap instead
-->

---

# Model Comparison

<div class="grid grid-cols-[3fr_4fr] gap-30">
<div>
<v-click at="1">
<figure>
    <img src="/loss_2_models.png" style="width: 335px !important;">
</figure>
</v-click>
<br>
<br>
<v-click at="2">
<Arrow x1="220" y1="280" x2="220" y2="340" width="3" color="orange" />
<figure>
    <img src="/loss_2_models_1000_trials.png" style="width: 335px !important;">
</figure>
</v-click>
</div>
<div>
<br>
<v-click at="3">
<figure>
    <img src="/model_selection_algorithm.png" style="width: 335px !important;">
    <figcaption style="color:#b3b3b3ff; font-size: 10px"><br>See our paper
      <a href="https://ieeexplore.ieee.org/document/11030460">doi:10.1109/ACCESS.2025.3578926</a>
    </figcaption>
</figure>
</v-click>
</div>
</div>

---

# Model Comparison Example: Best Mean Accuracy

<div class="grid grid-cols-[3fr_4fr] gap-30">
<div>
<br>
<figure>
    <img src="/model_selection_algorithm.png" style="width: 335px !important;">
    <figcaption style="color:#b3b3b3ff; font-size: 10px"><br>See our paper
      <a href="https://ieeexplore.ieee.org/document/11030460">doi:10.1109/ACCESS.2025.3578926</a>
    </figcaption>
</figure>
</div>
<div>
<br>
<center>
<v-click at="1">
<figure>
    <img src="/acc_1_trial.png" style="width: 430px !important;">
</figure>
</v-click>
<v-click at="2">
<figure>
    <img src="/acc_2_trial.png" style="width: 415px !important;">
</figure>
</v-click>
<v-click at="3">
<figure>
    <img src="/acc_3_trial.png" style="width: 415px !important;">
</figure>
</v-click>
<v-click at="4">
<figure>
    <img src="/acc_4_trial.png" style="width: 415px !important;">
</figure>
</v-click>
<br>
<br>
<v-click at="5">
<Arrow x1="720" y1="350" x2="720" y2="390" width="3" color="grey" />
<figure>
    <img src="/acc_4_trials_density.png" style="width: 430px !important;">
</figure>
</v-click>
</center>
</div>
</div>

---

# Model Path in the Algorithm: Mean Accuracy
<center>
<figure>
    <img src="/model_path_mean_30k_k5_72_new.png" style="width: 760px !important;">
</figure>
</center>

<br>
<div class="grid grid-cols-[2fr_2fr] gap-5">
<div>

* Warmup steps = 3
</div>
<div>

* ⭐ - baseline model 
</div>
</div>

---

# Computational Comparison with Exhaustive Search

<div class="grid grid-cols-[2fr_2fr] gap-18" style="align-items: center;">
<div>
<figure>
    <img src="/robust_select_72_models_computations.png" style="width: 460px !important;">
</figure>
</div>
<div style="padding-left: 20px;">

- Models: 72
- Instances: 20
- ~40-60% less computations
</div>
</div>

<br>
<br>

<v-click at="1">

#### Still computationally demanding approach.
</v-click>

---

<SelectionPlot
  :dt-len="30000"
  method="welch"
  :warmup="2"
  pvalue="005"
  :show-controls="true"
  width="900px"
  height="450px"
/>