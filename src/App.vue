<script setup>
import { ref, computed } from "vue";

// --- funciones ---
const f  = (x) => 4 * x ** 3 - 6 * x ** 2 + 7 * x - 2.3;
const g2 = (x) => x ** 2 * Math.sqrt(Math.abs(Math.cos(x))) - 5;

const fnOptions = [
  { key: "f",  label: "f(x) = 4x³ − 6x² + 7x − 2.3", fn: f  },
  { key: "g2", label: "g(x) = x² · √(|cos(x)|) − 5",  fn: g2 },
];

// estado
const selected = ref("");   // ← placeholder por defecto
const xi = ref(0), xf = ref(1), inc = ref(0.1);
const rows = ref([]);       // [{x,y}]
const extremos = ref(null); // {yMax,xAtMax,yMin,xAtMin} | null
const errorMsg = ref("");   // string vacío == sin error

const toFixed6 = (n) => Number(n).toFixed(6);

// para desactivar el botón si algo está mal
const isValid = computed(() => {
  if (!selected.value) return false;
  const a = Number(xi.value), b = Number(xf.value), h = Number(inc.value);
  return [a,b,h].every(Number.isFinite) && h !== 0;
});

// iterador sin “drift” de flotantes
function forRange(xi, xf, inc, cb){
  const dir = Math.sign(xf - xi) || 1;
  const step = Math.abs(inc) * dir;
  const steps = Math.floor(((xf - xi) / step) + 1e-12);
  for(let k=0;k<=steps+1;k++){
    const x = xi + k*step;
    if ((dir>0 && x>xf+1e-12) || (dir<0 && x<xf-1e-12)) break;
    cb(x);
  }
}

function generar(){
  errorMsg.value = "";

  if (!selected.value){
    rows.value = [];
    extremos.value = null;
    errorMsg.value = "Campos vacíos o erróneos.";
    return;
  }

  const opt = fnOptions.find(o=>o.key===selected.value);
  const a = Number(xi.value), b = Number(xf.value), h = Number(inc.value);

  if (![a,b,h].every(Number.isFinite) || h===0){
    rows.value = [];
    extremos.value = null;
    errorMsg.value = "Campos vacíos o erróneos.";
    return;
  }

  const data = [];
  let yMax=-Infinity, xAtMax=null, yMin=Infinity, xAtMin=null;

  forRange(a,b,h,(x)=>{
    const y = opt.fn(x);
    data.push({ x: toFixed6(x), y: toFixed6(y) });
    if (y>yMax){ yMax=y; xAtMax=x; }
    if (y<yMin){ yMin=y; xAtMin=x; }
  });

  rows.value = data;
  extremos.value = {
    yMax: toFixed6(yMax), xAtMax: toFixed6(xAtMax??0),
    yMin: toFixed6(yMin), xAtMin: toFixed6(xAtMin??0),
  };
}
</script>

<template>
  <main class="card">
    <header class="header">
      <div class="logo">📊</div>
      <h1>Tabular Funciones</h1>
      <p class="subtitle">Selecciona los límites y el incremento para tabular la función</p>
    </header>

    <section v-if="errorMsg" class="alert">
      <span>⚠️</span> {{ errorMsg }}
      <button class="close" @click="errorMsg = ''">✕</button>
    </section>

    <section class="controls">
      <label class="full">
        <span>Función a tabular</span>
        <select v-model="selected">
          <option disabled value="">Selecciona función</option>
          <option v-for="o in fnOptions" :key="o.key" :value="o.key">{{ o.label }}</option>
        </select>
      </label>

      <label>
        <span>Límite inferior</span>
        <input v-model.number="xi" type="number" step="any">
      </label>
      <label>
        <span>Límite superior</span>
        <input v-model.number="xf" type="number" step="any">
      </label>
      <label>
        <span>Incremento</span>
        <input v-model.number="inc" type="number" step="any">
      </label>

      <button class="primary" @click="generar" :disabled="!isValid">Generar Tabla</button>
    </section>

    <section class="table-wrap" v-if="rows.length">
      <table>
        <thead><tr><th>X</th><th>Y</th></tr></thead>
        <tbody>
          <tr v-for="(r,i) in rows" :key="i">
            <td>{{ r.x }}</td>
            <td>{{ r.y }}</td>
          </tr>
        </tbody>
      </table>
    </section>

    <section class="extremos" v-if="extremos">
      <h2>Puntos máximo y mínimo</h2>
      <div class="cards">
        <div class="ext-card ok">
          <h3>Valor Máximo</h3>
          <p class="val">Y = {{ extremos.yMax }}</p>
          <p class="at">en X = {{ extremos.xAtMax }}</p>
        </div>
        <div class="ext-card bad">
          <h3>Valor Mínimo</h3>
          <p class="val">Y = {{ extremos.yMin }}</p>
          <p class="at">en X = {{ extremos.xAtMin }}</p>
        </div>
      </div>
    </section>

    <footer class="foot">
      <small>Salida en notación fija con seis cifras decimales.</small>
    </footer>
  </main>
</template>

<style>
:root{ --bg:#0f1115; --card:#171923; --muted:#9aa4b2; --text:#e7ecf3;
  --accent:#60a5fa; --ok:#22c55e; --bad:#ef4444; --ring:#2a2f3a; }
*{box-sizing:border-box}
main.card{
  width:min(920px,92vw); background:var(--card); color:var(--text);
  border:1px solid var(--ring); border-radius:18px; padding:24px 24px 12px;
  margin:24px auto; box-shadow:0 10px 30px #00000055;
  font-family:ui-sans-serif,system-ui,-apple-system,Segoe UI,Roboto,Inter,Arial;
}
.header{display:grid; gap:6px; justify-items:center; text-align:center; margin-bottom:10px}
.logo{font-size:40px}
h1{margin:0; font-size:clamp(24px,4vw,40px); color:#a7c4ff}
.subtitle{margin:0; color:var(--muted)}

.alert{
  display:flex; align-items:center; gap:8px;
  background:#7f1d1d; border:1px solid #450a0a; color:#fff; padding:10px 12px;
  border-radius:10px; margin:10px 0;
}
.alert .close{margin-left:auto; background:transparent; border:0; color:#fff; cursor:pointer}

.controls{display:grid; grid-template-columns:repeat(4,1fr); gap:12px; margin:18px 0 8px}
.controls .full{grid-column:1/-1}
label{display:grid; gap:6px; font-size:14px}
input,select{
  background:#0d0f14; color:var(--text); border:1px solid #2a3140; border-radius:10px;
  padding:10px 12px; outline:none;
}
input:focus,select:focus{border-color:var(--accent); box-shadow:0 0 0 3px #60a5fa22}
.primary{
  grid-column:1/-1; padding:12px 16px; border-radius:12px; border:1px solid #2a3140;
  background:linear-gradient(180deg,#1e293b,#111827); color:#cde1ff; font-weight:600;
  cursor:pointer; transition:transform .05s ease, box-shadow .15s ease;
}
.primary:disabled{opacity:.5; cursor:not-allowed}
.primary:hover:not(:disabled){box-shadow:0 6px 20px #00000050}
.primary:active:not(:disabled){transform:translateY(1px)}

.table-wrap{max-height:360px; overflow:auto; border:1px solid #2a3140; border-radius:12px}
table{width:100%; border-collapse:collapse}
thead th{
  position:sticky; top:0; background:#0d0f14; color:#a7c4ff; letter-spacing:.03em;
  padding:12px; border-bottom:1px solid #2a3140; text-align:left;
}
tbody td{padding:12px; border-bottom:1px dashed #2a314033; font-variant-numeric:tabular-nums}
tbody tr:nth-child(even){background:#0f1118}

.extremos{margin:16px 0 10px}
.extremos h2{color:#a7c4ff; text-align:center; margin:6px 0 14px}
.cards{display:grid; grid-template-columns:1fr 1fr; gap:14px}
.ext-card{border:1px solid #2a3140; border-radius:14px; padding:14px; background:#0d0f14}
.ext-card h3{margin:0 0 8px; color:var(--muted)}
.ext-card.ok .val{color:var(--ok)} .ext-card.bad .val{color:var(--bad)}
.val{font-size:22px; font-weight:700; margin:4px 0}
.at{color:var(--muted); margin:0}
.foot{display:flex; justify-content:center; padding:6px 0 0}
small{color:var(--muted)}
@media (max-width:720px){ .controls{grid-template-columns:1fr 1fr} .cards{grid-template-columns:1fr}}
</style>
