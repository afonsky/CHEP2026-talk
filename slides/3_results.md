# Discarding Half of the Models per Iter.: Model Path

<center>
<figure>
    <img src="/model_path_mean_30k_k5_72_new.png" style="width: 790px !important;">
</figure>
</center>

<br>

<div class="grid grid-cols-[4fr_3fr_3fr] gap-5">
<div>

#### Ranking metric: Mean accuracy
</div>
<div>

#### Warmup steps $k = 3$
</div>
<div>

#### ⭐ - baseline model 
</div>
</div>

##### Note: The visualization is based on a subset of 72 models.

---

# Discarding Half of the Models per Iter.: Model Path

<center>
<figure>
    <img src="/model_path_std_30k_k5_72.png" style="width: 790px !important;">
</figure>
</center>

<br>

<div class="grid grid-cols-[4fr_3fr_3fr] gap-5">
<div>

#### Ranking metric: Std of accuracy
</div>
<div>

#### Warmup steps $k = 3$
</div>
<div>

#### ⭐ - baseline model 
</div>
</div>

##### Note: The visualization is based on a subset of 72 models.

---

# Discarding Half of the Models per Iter.: Calculations

<center>
<figure>
    <img src="/.full_sweep_vs_halving_scatter2.png" style="width: 790px !important;">
</figure>
</center>

<br>

<v-click at="1">

### Still computationally demanding. Can we do better?
</v-click>

---
zoom: 0.9
---

<style>
table { font-size: 17px; }
</style>

# Improved Model Selection Algorithm

* Idea: to calculate similarities among model performance distributions;
* Caveats:
    * Extremely low statistics (just a few values);
    * Comparable distributions might not be Gaussian.

| Statistical test   | Used for   | Is parametric? / Normality assumption | SciPy realization                                                                                             |   |
|--------------------|------------|---------------------------------------|---------------------------------------------------------------------------------------------------------------|---|
| Kolmogorov–Smirnov | Mean/Width | No                                    | <a href="https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.kstest.html">kstest</a>             |   |
| Mann–Whitney U     | Mean       | No                                    | <a href="https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.mannwhitneyu.html">mannwhitneyu</a> |   |
| Welch's t          | Mean       | Yes                                   | <a href="https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.ttest_ind.html">ttest_ind</a>       |   |
| Bartlett           | Width      | Yes                                   | <a href="https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.bartlett.html">bartlett</a>         |   |
| Levene             | Width      | Both                                  | <a href="https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.levene.html">levene</a>             |   |
| Fligner–Killeen    | Width      | No                                    | <a href="https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.fligner.html">fligner</a>           |   |        |   |

---

# Robust Selection Monitoring

<figure>
    <img src="/selection_steps_step02.png.jpg" style="width: 900px !important;">
</figure>

<v-click at="1">
<figure>
    <img src="/selection_steps_step09.png.jpg" style="width: 900px !important;">
</figure>
</v-click>

---

<br>
<br>
<br>
<br>
<br>
<br>

# <center><huge>`robust_select` playground</huge></center>

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