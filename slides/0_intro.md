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

<div class="grid grid-cols-[4fr_5fr] gap-20">
<div>
<v-click at="1">
<figure>
    <img src="/loss_1_trial.png" style="width: 360px !important;">
</figure>
</v-click>
<br>
<br>
<br>
<v-click at="2">
<Arrow x1="240" y1="180" x2="240" y2="230" width="3" color="grey" />
<figure>
    <img src="/loss_1000_trials.png" style="width: 360px !important;">
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
<center>
<figure>
    <img src="/randomization_sources.png" style="width: 335px !important;">
    <figcaption style="color:#b3b3b3ff; font-size: 10px"><br>Image credit: Our paper
      <a href="https://ieeexplore.ieee.org/document/11030460">doi:10.1109/ACCESS.2025.3578926</a>
    </figcaption>
</figure>
</center>
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

<div class="grid grid-cols-[4fr_5fr] gap-10">
<div>
<v-click at="1">
<figure>
    <img src="/loss_2_models.png" style="width: 360px !important;">
</figure>
</v-click>
<br>
<br>
<br>
<v-click at="2">
<Arrow x1="240" y1="180" x2="240" y2="230" width="3" color="grey" />
<figure>
    <img src="/loss_2_models_1000_trials.png" style="width: 360px !important;">
</figure>
</v-click>
</div>
<div>
<br>
<v-click at="3">

* In machine learning, a standard **hypeparameter optimization** is used to find the best model for the task;
    * Typically, Grid search or Random search;
        * In both approaches, the model is trained **no more than once**.
    * The best selected model is not guaranteed to have a smaller variability in prediction quality than the other models considered.
</v-click>
</div>
</div>

---

# Robust Selection Algorithm

<div class="grid grid-cols-[4fr_5fr] gap-15">
<div>
<figure>
    <img src="/loss_2_models.png" style="width: 360px !important;">
</figure>
<br>
<br>
<br>
<Arrow x1="240" y1="180" x2="240" y2="230" width="3" color="grey" />
<figure>
    <img src="/loss_2_models_1000_trials.png" style="width: 360px !important;">
</figure>
</div>
<div>

<center>

#### An alternative: `robust_select`
<figure>
    <img src="/model_selection_algorithm.png" style="width: 335px !important;">
    <figcaption style="color:#b3b3b3ff; font-size: 10px"><br>See our paper
      <a href="https://ieeexplore.ieee.org/document/11030460">doi:10.1109/ACCESS.2025.3578926</a>
    </figcaption>
</figure>
</center>
</div>
</div>

---

# Model Selection Example: Warmup Steps

<div class="grid grid-cols-[3fr_4fr] gap-30">
<div>
<br>
<div style="position: relative; display: inline-block;">
<figure>
    <img src="/model_selection_algorithm.png" style="width: 335px !important;">
    <figcaption style="color:#b3b3b3ff; font-size: 10px"><br>
      <a href="https://ieeexplore.ieee.org/document/11030460">doi:10.1109/ACCESS.2025.3578926</a>
    </figcaption>
</figure>
<div style="position: absolute; top: 6%; left: 0; width: 325px; height: 39%; background: #c3b5fdb0; border-radius: 4px;"></div>
</div>
</div>
<div>
<br>
<center>
<v-click at="1">
<div class="absolute top-16% left-58% bg-#999999 text-white px-1 py-0 rounded">M = 4</div>
<div class="absolute top-16% left-72% bg-#ff2894 text-white px-1 py-0 rounded">Ranking metric: mean</div>
</v-click>
<v-click at="2">
<figure>
    <img src="/acc_1_trial.png" style="width: 430px !important;">
</figure>
<div class="absolute top-29.1% left-48% bg-violet-300 text-white px-1 py-0 rounded">1</div>
</v-click>
<v-click at="3">
<figure>
    <img src="/acc_2_trial.png" style="width: 415px !important;">
</figure>
<div class="absolute top-37.5% left-48% bg-violet-300 text-white px-1 py-0 rounded">2</div>
</v-click>
<v-click at="4">
<figure>
    <img src="/acc_3_trial.png" style="width: 415px !important;">
</figure>
<div class="absolute top-45.8% left-48% bg-violet-300 text-white px-1 py-0 rounded">3</div>
</v-click>
<v-click at="5">
<figure>
    <img src="/acc_4_trial.png" style="width: 415px !important;">
<div class="absolute top-53.7% left-46.8% bg-violet-300 text-white px-1 py-0 rounded">k=4</div>
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

# Model Selection Example: Selection Steps

<div class="grid grid-cols-[3fr_4fr] gap-30">
<div>
<br>
<div style="position: relative; display: inline-block;">
<figure>
    <img src="/model_selection_algorithm.png" style="width: 335px !important;">
    <figcaption style="color:#b3b3b3ff; font-size: 10px"><br>See our paper
      <a href="https://ieeexplore.ieee.org/document/11030460">doi:10.1109/ACCESS.2025.3578926</a>
    </figcaption>
</figure>
<div style="position: absolute; top: 45%; left: 0; width: 325px; height: 46%; background: #db96009c; border-radius: 4px;"></div>
</div>
</div>
<div>
<br>
<center>
<v-click at="1">
<div class="absolute top-16% left-58% bg-#999999 text-white px-1 py-0 rounded">M = 4</div>
</v-click>
<v-click at="2">
<figure>
    <img src="/acc_4_trials_density.png" style="width: 430px !important;">
</figure>
<div class="absolute top-27% left-43% bg-#db96009c text-white px-1 py-0 rounded">after k<br>steps</div>
</v-click>
<br>
<br>
<v-click at="3">
<figure>
    <img src="/acc_5_trial.png" style="width: 415px !important;">
</figure>
<div class="absolute top-48% left-72.5% bg-#999999 text-white px-1 py-0 rounded">+</div>
<div class="absolute top-58.2% left-45.5% bg-#db96009c text-white px-1 py-0 rounded">k+1</div>
</v-click>
<br>
<br>
<v-click at="4">
<Arrow x1="720" y1="365" x2="720" y2="405" width="3" color="grey" />
<div style="position: relative; top: 10px">
<figure>
    <img src="/acc_5_trials_density.png" style="width: 430px !important;">
</figure>
</div>
<div class="absolute top-67.5% left-58% bg-#999999 text-white px-1 py-0 rounded">M = 2</div>
</v-click>
</center>
</div>
</div>

---

# Discarding Half of the Models per Iteration

<br>

* The robust model selection algorithm selects robust model(s) from among $M$ models;
    * It requires that each model be trained at least $k$ times;

<br>

* At each iteration, **half** of the worst models are discarded;

<br>

* Total number of model training runs to select **one model** is <span style="background: rgba(66,133,244,0.15); border: 2px solid #4285f4; border-radius: 8px; padding: 6px 6px;">$(k + 2) \cdot M - 2$</span>
    * In comparison:
        * Grid search (<span style="color: #ff1414;">non-robust</span> selection): $M$
        * Exhaustive search (<span style="color: #02aa10;">robust</span> selection): $M \cdot \log_2 M$
