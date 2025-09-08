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
const selected = ref("");   // placeholder por defecto
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
  <div class="page">
    <header class="topbar">
      <div class="brand">📊</div>
      <div>
        <h1>Tabular Funciones</h1>
        <p>Selecciona límites e incremento para generar la tabla</p>
      </div>
    </header>

    <div class="layout">
      <!-- Panel izquierdo (controles) -->
      <aside class="panel">
        <h2 class="panel-title">Parámetros</h2>

        <div v-if="errorMsg" class="notice">
          <span>⚠️</span> <strong>{{ errorMsg }}</strong>
          <button class="close" @click="errorMsg = ''" aria-label="Cerrar">✕</button>
        </div>

        <label class="field">
          <span>Función</span>
          <select v-model="selected">
            <option disabled value="">Selecciona función</option>
            <option v-for="o in fnOptions" :key="o.key" :value="o.key">{{ o.label }}</option>
          </select>
        </label>

        <div class="grid-2">
          <label class="field">
            <span>Límite inferior</span>
            <input v-model.number="xi" type="number" step="any" />
          </label>
          <label class="field">
            <span>Límite superior</span>
            <input v-model.number="xf" type="number" step="any" />
          </label>
        </div>

        <label class="field">
          <span>Incremento</span>
          <input v-model.number="inc" type="number" step="any" />
        </label>

        <button class="btn" @click="generar" :disabled="!isValid">Generar tabla</button>

        <p class="hint">Salida en notación fija con <strong>6 decimales</strong>.</p>
      </aside>

      <!-- Resultados -->
      <section class="results">
        <div class="ext-cards" v-if="extremos">
          <div class="card good">
            <div class="badge">Máximo</div>
            <div class="kv">
              <div class="k">Y</div><div class="v">{{ extremos.yMax }}</div>
              <div class="k">en X</div><div class="v">{{ extremos.xAtMax }}</div>
            </div>
          </div>
          <div class="card bad">
            <div class="badge">Mínimo</div>
            <div class="kv">
              <div class="k">Y</div><div class="v">{{ extremos.yMin }}</div>
              <div class="k">en X</div><div class="v">{{ extremos.xAtMin }}</div>
            </div>
          </div>
        </div>

        <div class="table-wrap" v-if="rows.length">
          <table>
            <thead>
              <tr>
                <th style="width:40%">X</th>
                <th>Y</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(r,i) in rows" :key="i">
                <td>{{ r.x }}</td>
                <td>{{ r.y }}</td>
              </tr>
            </tbody>
          </table>
        </div>

        <div v-else class="empty">
          <p>Elige una función y pulsa <strong>Generar tabla</strong>.</p>
        </div>
      </section>
    </div>
  </div>
</template>

<style>
/* ===== Paleta clara con acento teal/indigo ===== */
:root{
  --bg:#f6f7fb;
  --ink:#0f172a;         /* texto principal */
  --muted:#5b647a;       /* texto secundario */
  --panel:#ffffff;       /* panel izquierdo */
  --panel-border:#e6e9f2;
  --surface:#ffffff;     /* tarjetas/tabla */
  --surface-br:#e6e9f2;
  --accent:#16a34a;      /* teal/verde */
  --accent-2:#4f46e5;    /* indigo para detalles */
  --good:#059669;
  --bad:#dc2626;
}

/* Layout base */
*{box-sizing:border-box}
html,body,#app{height:100%}
body{margin:0; background:var(--bg); color:var(--ink);
  font-family:ui-sans-serif,system-ui,-apple-system,Segoe UI,Inter,Roboto,Arial}

/* Barra superior */
.topbar{
  display:flex; align-items:center; gap:14px;
  padding:22px 28px; background:linear-gradient(90deg,#eef3ff,#e9fff2);
  border-bottom:1px solid #e7ecfa;
}
.brand{font-size:34px}
.topbar h1{margin:0; font-size:clamp(22px,3.6vw,32px); color:var(--accent-2)}
.topbar p{margin:2px 0 0; color:var(--muted); font-weight:600}

/* Disposición principal */
.layout{
  max-width:1100px; margin:20px auto; padding:0 20px;
  display:grid; grid-template-columns:320px 1fr; gap:22px;
}
@media (max-width:900px){ .layout{ grid-template-columns:1fr; } }

/* Panel izquierdo */
.panel{
  background:var(--panel); border:1px solid var(--panel-border);
  border-radius:16px; padding:18px; position:sticky; top:16px; height:fit-content;
  box-shadow:0 6px 18px rgba(15,23,42,.06);
}
.panel-title{margin:0 0 10px; font-size:16px; color:var(--muted); text-transform:uppercase; letter-spacing:.06em}

.notice{
  display:flex; align-items:center; gap:10px; padding:10px 12px;
  background:#fff1f2; color:#991b1b; border:1px solid #fecaca; border-radius:12px; margin-bottom:10px;
}
.notice .close{margin-left:auto; background:transparent; border:0; color:#991b1b; cursor:pointer; font-size:16px}

.field{display:grid; gap:6px; margin-bottom:12px}
.field span{font-size:13px; color:var(--muted); font-weight:700; letter-spacing:.02em}
input,select{
  background:#fbfdff; border:1px solid #cfd6ea; border-radius:10px; padding:10px 12px;
  color:var(--ink); outline:none; font-weight:600;
}
input:focus,select:focus{border-color:#8aa2ff; box-shadow:0 0 0 3px rgba(138,162,255,.25)}
.grid-2 {
  display: grid;
  grid-template-columns: 1fr 1fr; 
  gap: 12px;
}

.grid-2 input {
  width: 100%;       
  max-width: 100%;    
  box-sizing: border-box;
}

/* Botón */
.btn{
  width:100%; padding:12px 14px; border:0; border-radius:12px;
  background:linear-gradient(90deg,var(--accent),#22c55e); color:#fff; font-weight:800;
  letter-spacing:.02em; cursor:pointer; box-shadow:0 8px 18px rgba(34,197,94,.25);
}
.btn:disabled{opacity:.5; cursor:not-allowed; box-shadow:none}

/* Resultados */
.results{min-height:300px}

.ext-cards{
  display:grid; grid-template-columns:1fr 1fr; gap:14px; margin-bottom:14px;
}
@media (max-width:600px){ .ext-cards{grid-template-columns:1fr} }

.card{
  background:var(--surface); border:1px solid var(--surface-br);
  border-radius:14px; padding:14px; box-shadow:0 6px 16px rgba(15,23,42,.06);
}
.card.good{border-color:#bbf7d0}
.card.bad{border-color:#fecaca}
.badge{
  display:inline-block; padding:4px 10px; border-radius:999px;
  background:#eef2ff; color:var(--accent-2); font-weight:800; letter-spacing:.04em; font-size:12px;
}
.kv{display:grid; grid-template-columns:auto 1fr; gap:6px 10px; margin-top:10px}
.k{color:var(--muted); font-weight:700}
.v{font-weight:900}
.card.good .v{color:var(--good)}
.card.bad  .v{color:var(--bad)}

/* Tabla */
.table-wrap{
  background:var(--surface); border:1px solid var(--surface-br);
  border-radius:14px; overflow:auto; box-shadow:0 8px 18px rgba(15,23,42,.06);
}
table{width:100%; border-collapse:separate; border-spacing:0}
thead th{
  position:sticky; top:0; background:#eef2ff; color:#1e293b; text-align:left;
  padding:12px 14px; font-size:13px; letter-spacing:.06em; text-transform:uppercase;
  border-bottom:1px solid var(--surface-br);
}
tbody td{
  padding:12px 14px; border-bottom:1px solid #eef1f7; font-variant-numeric:tabular-nums; font-weight:700;
}
tbody tr:nth-child(even){ background:#fafbff }
.empty{
  display:grid; place-items:center; padding:40px; color:var(--muted);
}
.hint{color:var(--muted); font-size:12px; margin:10px 2px 0}
</style>
