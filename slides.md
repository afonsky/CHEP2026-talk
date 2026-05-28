---
theme: seriph
addons:
  - "@twitwi/slidev-addon-ultracharger"
addonsConfig:
  ultracharger:
    inlineSvg:
      markersWorkaround: false
    disable:
      - metaFooter
      - tocFooter
background: /funnel.jpg
highlighter: shiki
routerMode: hash
lineNumbers: false
aspectRatio: 16/9

css: unocss
title: Robust by Design
subtitle: A Meta‑Algorithm for Stable Deep Learning
date: 28/05/2026
venue: CHEP 2026
author: Alexey Boldyrev, Fedor Ratnikov, Andrey Shevelev
---

<br>
<br>

# <span style="font-size:28.0pt" v-html="$slidev.configs.title?.replaceAll(' ', '<br/>')"></span>
## <span style="font-size:26.0pt" v-html="$slidev.configs.subtitle?.replaceAll(' ', '<br/>')"></span>
# <span style="font-size:18.0pt"><u>Alexey Boldyrev</u>, Fedor Ratnikov, Andrey Shevelev</span>
#### (HSE University, Lambda lab)
<br>
<br>

### <span style="font-size:18.0pt">CHEP 2026</span>
### <span style="font-size:18.0pt">Bangkok Thailand</span>

<div>

<span style="color:#b3b3b3ff; font-size: 11px; float: right;">Background image: algorithm's depiction</span>
</div>

<style>
  :deep(footer) { padding-bottom: 3em !important; }
</style>


---
src: ./slides/0_intro.md
---

---
src: ./slides/2_dataset.md
---

---
src: ./slides/3_results.md
---

---
src: ./slides/0_end.md
---