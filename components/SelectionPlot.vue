<template>
  <div class="selection-plot">
    <div v-if="showControls" class="sp-controls">
      <label>
        metric
        <select v-model="metricLocal">
          <option v-for="o in metricOptions" :key="o" :value="o">{{ o }}</option>
        </select>
      </label>
      <label>
        sample size
        <select v-model.number="dtLenLocal">
          <option v-for="o in dtLenOptions" :key="o" :value="o">{{ o }}</option>
        </select>
      </label>
      <label>
        method
        <select v-model="methodLocal">
          <option v-for="o in methodOptionsForMetric" :key="o" :value="o">{{ o }}</option>
        </select>
      </label>
      <label>
        warmup
        <select v-model.number="warmupLocal">
          <option v-for="o in warmupOptions" :key="o" :value="o">{{ o }}</option>
        </select>
      </label>
      <label>
        p-value
        <select v-model="pvalueLocal">
          <option v-for="o in pvalueOptions" :key="o" :value="o">{{ pvalueLabel(o) }}</option>
        </select>
      </label>
      <span v-if="status" class="sp-status">{{ status }}</span>
    </div>
    <div :id="plotId" :style="{ width: width, height: height }"></div>
  </div>
</template>

<script setup>
import { onMounted, onBeforeUnmount, ref, computed, watch } from 'vue'
import Plotly from 'plotly.js-dist'
import Papa from 'papaparse'

const METHOD_OPTIONS_BY_METRIC = {
  mean: ['welch', 'ks'],
  std: ['bartlett', 'fligner'],
}

const props = defineProps({
  metric: { type: String, default: 'mean' },
  dtLen: { type: Number, default: 30000 },
  method: { type: String, default: 'welch' },
  warmup: { type: Number, default: 2 },
  pvalue: { type: String, default: '005' },
  width: { type: String, default: '900px' },
  height: { type: String, default: '500px' },
  showBaseline: { type: Boolean, default: true },
  showControls: { type: Boolean, default: true },
  metricOptions: { type: Array, default: () => ['mean', 'std'] },
  dtLenOptions: { type: Array, default: () => [10000, 20000, 30000] },
  warmupOptions: { type: Array, default: () => [2, 3, 4, 5] },
  pvalueOptions: { type: Array, default: () => ['001', '005', '010', '020'] },
  resnetCsv: { type: String, default: '/data/resnet_540_models_val_only.csv' },
  cctCsv: {
    type: String,
    default: '/data/cct_64_models_val_only.csv',
  },
  stepsTemplate: {
    type: String,
    default:
      '/data/model_selection_steps/{resdir}/selstep_{lenpath}_{method}_step{warmup}_pv{pvalue}.json',
  },
})

const metricLocal = ref(props.metric)
const dtLenLocal = ref(props.dtLen)
const methodLocal = ref(props.method)
const warmupLocal = ref(props.warmup)
const pvalueLocal = ref(props.pvalue)
const status = ref('')

const methodOptionsForMetric = computed(
  () => METHOD_OPTIONS_BY_METRIC[metricLocal.value] || [],
)

const plotId = `selplot-${Math.random().toString(36).slice(2)}`
const resizeObserver = ref(null)
let resnetCache = null
let cctCache = null

const HOVER_COLS = [
  'model',
  'batch_size',
  'peak_lr',
  'l2_reg',
  'activation_function',
  'num_filters',
]

const HOVER_TMPL =
  '<b>%{customdata[0]}</b><br>' +
  'batch_size=%{customdata[1]}<br>' +
  'peak_lr=%{customdata[2]}<br>' +
  'l2_reg=%{customdata[3]}<br>' +
  'activation=%{customdata[4]}<br>' +
  'num_filters=%{customdata[5]}' +
  '<extra></extra>'

const BASELINE_BY_DTLEN = {
  10000: 'model_ddb802e5',
  20000: 'model_13469335',
  30000: 'model_e169bb1c',
}

function pvalueLabel(s) {
  return `0.${s}`
}

// Prepend the Vite base URL so absolute "/data/..." paths still resolve when
// the site is deployed under a subpath (e.g. GitHub Pages /repo-name/).
const BASE = (import.meta.env?.BASE_URL || '/').replace(/\/+$/, '/')
function withBase(url) {
  if (!url) return url
  if (/^[a-z][a-z0-9+.-]*:\/\//i.test(url)) return url
  if (url.startsWith(BASE)) return url
  return BASE + url.replace(/^\/+/, '')
}

async function fetchCsv(url) {
  const full = withBase(url)
  const resp = await fetch(full)
  if (!resp.ok) throw new Error(`Failed to fetch ${full}: ${resp.status}`)
  const text = await resp.text()
  return new Promise((resolve, reject) => {
    Papa.parse(text, {
      header: true,
      dynamicTyping: true,
      skipEmptyLines: true,
      complete: (res) => resolve(res.data),
      error: reject,
    })
  })
}

async function fetchCsvOrNull(url) {
  try {
    return await fetchCsv(url)
  } catch (e) {
    console.warn('[SelectionPlot] optional CSV not available:', url)
    return null
  }
}

function suffixNum(col) {
  const m = /_(\d+)$/.exec(col)
  return m ? parseInt(m[1], 10) : -1
}

function pickCol(row, col) {
  const v = row[col]
  return v === undefined || v === null || v === '' || Number.isNaN(v) ? null : v
}

function rowsToTraceArrays(rows, xCol, yCol) {
  const x = new Array(rows.length)
  const y = new Array(rows.length)
  const customdata = new Array(rows.length)
  for (let i = 0; i < rows.length; i++) {
    x[i] = pickCol(rows[i], xCol)
    y[i] = pickCol(rows[i], yCol)
    customdata[i] = HOVER_COLS.map((c) => rows[i][c] ?? null)
  }
  return { x, y, customdata }
}

// Plotly drops the legend swatch when a trace's data is a 0-length array.
// Keep the trace "alive" by feeding it a single null point — invisible but
// preserves the legend entry.
const NULL_CD = HOVER_COLS.map(() => null)
function ensureNonEmpty(x, y, customdata) {
  if (x.length === 0) {
    return { x: [null], y: [null], customdata: [NULL_CD] }
  }
  return { x, y, customdata }
}

function minMax(values) {
  let mn = Infinity
  let mx = -Infinity
  for (const v of values) {
    if (v == null || Number.isNaN(v)) continue
    if (v < mn) mn = v
    if (v > mx) mx = v
  }
  return [mn, mx]
}

function buildFrames(dfPlot, selData) {
  const ttColsS = Object.keys(dfPlot[0] || {})
    .filter((c) => c.startsWith('TrainingTime_'))
    .sort((a, b) => suffixNum(a) - suffixNum(b))
  const valColsS = Object.keys(dfPlot[0] || {})
    .filter((c) => c.startsWith('ValAccuracy_'))
    .sort((a, b) => suffixNum(a) - suffixNum(b))

  const allTt = []
  const allVal = []
  for (const r of dfPlot) {
    for (const c of ttColsS) {
      const v = r[c]
      if (v != null && !Number.isNaN(v)) allTt.push(v)
    }
    for (const c of valColsS) {
      const v = r[c]
      if (v != null && !Number.isNaN(v)) allVal.push(v)
    }
  }
  const [ttMin, ttMax] = minMax(allTt)
  const [valMin, valMax] = minMax(allVal)
  const xRange = [ttMin * 0.98, ttMax * 1.02]
  const yRange = [valMin * 0.98, valMax * 1.02]

  const step1Virtual = {
    step: 1,
    kept_models: Array.from({ length: dfPlot.length }, (_, i) => i),
    removed_models: [],
  }
  const allSteps = [step1Virtual, ...selData.steps]

  const frames = []
  for (let j = 0; j < allSteps.length; j++) {
    const stepInfo = allSteps[j]
    const stepNum = stepInfo.step
    const ttCol = ttColsS[stepNum - 1]
    const valCol = valColsS[stepNum - 1]
    const keptIdx = stepInfo.kept_models
    const removedIdx = stepInfo.removed_models
    const currentKept = new Set(keptIdx)
    const removedSet = new Set(removedIdx)

    const blackRows = []
    const redRows = []
    const blackXs = []
    const blackYs = []
    const redXs = []
    const redYs = []

    for (let k = 0; k < j; k++) {
      const prev = allSteps[k]
      const prevTtCol = ttColsS[prev.step - 1]
      const prevValCol = valColsS[prev.step - 1]
      for (const m of prev.kept_models) {
        if (currentKept.has(m)) {
          const row = dfPlot[m]
          blackXs.push(pickCol(row, prevTtCol))
          blackYs.push(pickCol(row, prevValCol))
          blackRows.push(row)
        } else if (removedSet.has(m)) {
          const row = dfPlot[m]
          redXs.push(pickCol(row, prevTtCol))
          redYs.push(pickCol(row, prevValCol))
          redRows.push(row)
        }
      }
    }

    const greenRows = keptIdx.map((i) => dfPlot[i])
    const green = rowsToTraceArrays(greenRows, ttCol, valCol)

    const blackCd = blackRows.map((r) => HOVER_COLS.map((c) => r[c] ?? null))
    const redCd = redRows.map((r) => HOVER_COLS.map((c) => r[c] ?? null))
    const black = ensureNonEmpty(blackXs, blackYs, blackCd)
    const greenSafe = ensureNonEmpty(green.x, green.y, green.customdata)
    const red = ensureNonEmpty(redXs, redYs, redCd)

    frames.push({
      name: String(stepNum),
      data: [
        {
          x: black.x,
          y: black.y,
          customdata: black.customdata,
          mode: 'markers',
          type: 'scatter',
          marker: { color: 'black', size: 5, opacity: 0.9 },
          hovertemplate: HOVER_TMPL,
          name: 'prev kept',
          showlegend: true,
          visible: true,
          legendrank: 1,
        },
        {
          x: greenSafe.x,
          y: greenSafe.y,
          customdata: greenSafe.customdata,
          mode: 'markers',
          type: 'scatter',
          marker: { color: 'green', size: 5, opacity: 0.7 },
          hovertemplate: HOVER_TMPL,
          name: 'kept',
          showlegend: true,
          visible: true,
          legendrank: 2,
        },
        {
          x: red.x,
          y: red.y,
          customdata: red.customdata,
          mode: 'markers',
          type: 'scatter',
          marker: { color: 'red', size: 5, opacity: 0.7 },
          hovertemplate: HOVER_TMPL,
          name: 'removed',
          showlegend: true,
          visible: true,
          legendrank: 3,
        },
      ],
    })
  }

  const s0 = allSteps[0]
  const initial = rowsToTraceArrays(
    dfPlot,
    ttColsS[s0.step - 1],
    valColsS[s0.step - 1],
  )

  return { frames, xRange, yRange, initial }
}

function fmtPath(tmpl, params) {
  return tmpl.replace(/\{(\w+)\}/g, (_, k) => params[k])
}

let renderToken = 0

async function render() {
  const token = ++renderToken
  const metric = metricLocal.value
  const dtLen = dtLenLocal.value
  const method = methodLocal.value
  const warmup = warmupLocal.value
  const pvalue = pvalueLocal.value
  const lenpath = `${Math.round(dtLen / 1000)}k`
  const resdir = metric === 'std' ? `results_std_${lenpath}` : `results_${lenpath}`
  const params = { lenpath, method, warmup, pvalue, resdir }

  if (!METHOD_OPTIONS_BY_METRIC[metric]?.includes(method)) {
    status.value = `method "${method}" is not valid for metric "${metric}"`
    return
  }

  status.value = 'loading…'

  if (!resnetCache) {
    resnetCache = await fetchCsv(props.resnetCsv)
  }
  if (cctCache === null) {
    cctCache = await fetchCsvOrNull(props.cctCsv)
  }
  const cctRows = cctCache

  const stepsUrl = withBase(fmtPath(props.stepsTemplate, params))
  const selResp = await fetch(stepsUrl)
  if (!selResp.ok) {
    status.value = `no data for ${method}/step${warmup}/pv${pvalue}@${lenpath}`
    return
  }
  const selData = await selResp.json()

  if (token !== renderToken) return

  const resnet = resnetCache.map((r) => ({ ...r, model_type: 'resnet' }))
  const cct = (cctRows || []).map((r) => ({ ...r, model_type: 'cct' }))
  const df = [...resnet, ...cct]

  const dfPlot = df.filter((r) => r.dt_len === dtLen)
  const baselineModel = BASELINE_BY_DTLEN[dtLen]
  const baselineRow = df.find(
    (r) => r.dt_len === dtLen && r.model === baselineModel,
  )

  const ttCols = baselineRow
    ? Object.keys(baselineRow).filter((c) => c.startsWith('TrainingTime_'))
    : []
  const valCols = baselineRow
    ? Object.keys(baselineRow).filter((c) => c.startsWith('ValAccuracy_'))
    : []
  const ttBase = baselineRow ? ttCols.map((c) => baselineRow[c]) : []
  const valBase = baselineRow ? valCols.map((c) => baselineRow[c]) : []

  const { frames, xRange, yRange, initial } = buildFrames(dfPlot, selData)

  const initialBlack = ensureNonEmpty([], [], [])
  const initialKept = ensureNonEmpty(initial.x, initial.y, initial.customdata)
  const initialRed = ensureNonEmpty([], [], [])

  const initialData = [
    {
      x: initialBlack.x,
      y: initialBlack.y,
      customdata: initialBlack.customdata,
      mode: 'markers',
      type: 'scatter',
      marker: { color: 'black', size: 5, opacity: 0.9 },
      hovertemplate: HOVER_TMPL,
      name: 'prev kept',
      showlegend: true,
      visible: true,
      legendrank: 1,
    },
    {
      x: initialKept.x,
      y: initialKept.y,
      customdata: initialKept.customdata,
      mode: 'markers',
      type: 'scatter',
      marker: { color: 'green', size: 5, opacity: 0.7 },
      hovertemplate: HOVER_TMPL,
      name: 'kept',
      showlegend: true,
      visible: true,
      legendrank: 2,
    },
    {
      x: initialRed.x,
      y: initialRed.y,
      customdata: initialRed.customdata,
      mode: 'markers',
      type: 'scatter',
      marker: { color: 'red', size: 5, opacity: 0.7 },
      hovertemplate: HOVER_TMPL,
      name: 'removed',
      showlegend: true,
      visible: true,
      legendrank: 3,
    },
  ]

  if (props.showBaseline && baselineRow) {
    const ttValid = ttBase.filter((v) => v != null && !Number.isNaN(v))
    const valValid = valBase.filter((v) => v != null && !Number.isNaN(v))
    const mean = (a) => a.reduce((s, v) => s + v, 0) / a.length
    const std = (a) => {
      const m = mean(a)
      return Math.sqrt(a.reduce((s, v) => s + (v - m) * (v - m), 0) / a.length)
    }
    const ttMu = ttValid.length ? mean(ttValid) : null
    const valMu = valValid.length ? mean(valValid) : null
    const ttSd = ttValid.length ? std(ttValid) : 0
    const valSd = valValid.length ? std(valValid) : 0

    initialData.push({
      x: [ttMu],
      y: [valMu],
      mode: 'markers',
      type: 'scatter',
      marker: {
        symbol: 'star',
        color: 'gold',
        size: 16,
        opacity: 0.95,
        line: { color: 'black', width: 0.8 },
      },
      hovertemplate:
        `<b>baseline</b><br>` +
        `mean tt=${ttMu != null ? ttMu.toFixed(1) : 'n/a'}<br>` +
        `mean acc=${valMu != null ? valMu.toFixed(4) : 'n/a'}` +
        `<extra></extra>`,
      name: 'baseline',
      showlegend: true,
      visible: true,
      legendrank: 4,
    })
  }

  const sliderSteps = frames.map((f) => ({
    label: f.name,
    method: 'animate',
    args: [
      [f.name],
      {
        frame: { duration: 0, redraw: true },
        mode: 'immediate',
        transition: { duration: 0 },
      },
    ],
  }))

  // Attach the default range to the active frame so the slider's auto-animate
  // (active: 0) re-asserts it after every newPlot — survives any prior user
  // zoom because frame.layout is applied during animate().
  if (frames.length > 0) {
    frames[0].layout = {
      'xaxis.range': xRange.slice(),
      'yaxis.range': yRange.slice(),
      'xaxis.autorange': false,
      'yaxis.autorange': false,
    }
  }

  const layout = {
    xaxis: {
      title: { text: 'Model Training Time, s' },
      range: xRange.slice(),
      autorange: false,
    },
    yaxis: {
      title: { text: 'Validation Accuracy' },
      range: yRange.slice(),
      autorange: false,
    },
    paper_bgcolor: 'white',
    margin: { l: 60, r: 20, t: 20, b: 80 },
    legend: { x: 1, y: 0, xanchor: 'right', yanchor: 'bottom' },
    uirevision: `${metric}-${dtLen}-${method}-${warmup}-${pvalue}`,
    sliders: [
      {
        active: 0,
        steps: sliderSteps,
        currentvalue: { prefix: 'Step: ', font: { size: 14 } },
        pad: { t: 40, b: 10 },
        len: 0.95,
        x: 0.025,
      },
    ],
  }

  const config = { displayModeBar: false, responsive: true }

  const el = document.getElementById(plotId)
  if (!el || token !== renderToken) return
  Plotly.purge(el)
  await Plotly.newPlot(el, initialData, layout, config)
  await Plotly.addFrames(el, frames)
  await Plotly.relayout(el, {
    'xaxis.range': xRange.slice(),
    'yaxis.range': yRange.slice(),
    'xaxis.autorange': false,
    'yaxis.autorange': false,
  })
  // Force the first frame so its layout (with the default range) takes effect,
  // overriding any leftover user zoom from the previous selection.
  if (frames.length > 0) {
    await Plotly.animate(el, [frames[0].name], {
      frame: { duration: 0, redraw: true },
      transition: { duration: 0 },
      mode: 'immediate',
    })
  }
  status.value = ''
}

onMounted(() => {
  render().catch((e) => {
    console.error('[SelectionPlot]', e)
    status.value = String(e.message || e)
  })
  resizeObserver.value = new ResizeObserver(() => {
    const el = document.getElementById(plotId)
    if (el) Plotly.Plots.resize(el)
  })
  const el = document.getElementById(plotId)
  if (el) resizeObserver.value.observe(el)
})

onBeforeUnmount(() => {
  if (resizeObserver.value) resizeObserver.value.disconnect()
  const el = document.getElementById(plotId)
  if (el) Plotly.purge(el)
})

// When the metric switches, the previously-selected method may not be valid
// (e.g. "welch" makes no sense under "std") — snap to the first option for
// the new metric.
watch(metricLocal, (m) => {
  const opts = METHOD_OPTIONS_BY_METRIC[m] || []
  if (!opts.includes(methodLocal.value)) {
    methodLocal.value = opts[0]
  }
})

watch(
  () => [
    metricLocal.value,
    dtLenLocal.value,
    methodLocal.value,
    warmupLocal.value,
    pvalueLocal.value,
  ],
  () =>
    render().catch((e) => {
      console.error('[SelectionPlot]', e)
      status.value = String(e.message || e)
    }),
)

watch(
  () => [props.metric, props.dtLen, props.method, props.warmup, props.pvalue],
  ([mt, d, m, w, p]) => {
    metricLocal.value = mt
    dtLenLocal.value = d
    methodLocal.value = m
    warmupLocal.value = w
    pvalueLocal.value = p
  },
)
</script>

<style scoped>
.selection-plot {
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
}
.sp-controls {
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem;
  align-items: center;
  font-size: 0.85rem;
  color: #333;
}
.sp-controls label {
  display: inline-flex;
  align-items: center;
  gap: 0.3rem;
}
.sp-controls select {
  font-size: 0.85rem;
  padding: 1px 4px;
  border: 1px solid #bbb;
  border-radius: 3px;
  background: white;
}
.sp-status {
  color: #888;
  font-style: italic;
}
</style>
