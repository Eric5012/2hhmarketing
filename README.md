<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>2HH Marketing Dashboard</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
  <script>
    tailwind.config = {
      darkMode: 'class',
      theme: {
        extend: {
          colors: {
            brand: {
              50:  '#f0f4ff',
              100: '#dde6ff',
              200: '#c3d1ff',
              300: '#9db1ff',
              400: '#7086fb',
              500: '#4f60f5',
              600: '#3a42e9',
              700: '#3033ce',
              800: '#2b2fa7',
              900: '#292d84',
            }
          },
          fontFamily: { sans: ['Inter', 'ui-sans-serif', 'system-ui'] }
        }
      }
    }
  </script>
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&display=swap" rel="stylesheet" />
  <style>
    * { box-sizing: border-box; }
    body { font-family: 'Inter', sans-serif; }
    .card {
      background: rgba(255,255,255,0.05);
      backdrop-filter: blur(12px);
      border: 1px solid rgba(255,255,255,0.1);
      border-radius: 16px;
      padding: 24px;
      transition: transform .2s, box-shadow .2s;
    }
    .card:hover { transform: translateY(-2px); box-shadow: 0 12px 40px rgba(0,0,0,0.3); }
    .badge {
      display: inline-flex; align-items: center; gap: 4px;
      padding: 2px 10px; border-radius: 999px; font-size: 12px; font-weight: 600;
    }
    .badge-up   { background: rgba(52,211,153,.15); color: #34d399; }
    .badge-down { background: rgba(248,113,113,.15); color: #f87171; }
    .select-styled {
      background: rgba(255,255,255,0.08);
      border: 1px solid rgba(255,255,255,0.15);
      border-radius: 8px; color: #e2e8f0;
      padding: 6px 12px; font-size: 13px; cursor: pointer; outline: none;
      transition: border-color .2s;
    }
    .select-styled:focus { border-color: #7086fb; }
    .select-styled option { background: #1e2035; color: #e2e8f0; }
    .btn-primary {
      background: linear-gradient(135deg,#4f60f5,#7086fb);
      color: #fff; border: none; border-radius: 8px;
      padding: 8px 18px; font-size: 13px; font-weight: 600;
      cursor: pointer; transition: opacity .2s, transform .1s;
    }
    .btn-primary:hover  { opacity: .9; transform: translateY(-1px); }
    .btn-primary:active { transform: translateY(0); }
    .btn-danger {
      background: rgba(248,113,113,.15); color: #f87171;
      border: 1px solid rgba(248,113,113,.3); border-radius: 6px;
      padding: 3px 10px; font-size: 12px; cursor: pointer; transition: background .2s;
    }
    .btn-danger:hover { background: rgba(248,113,113,.3); }
    .input-styled {
      background: rgba(255,255,255,0.08);
      border: 1px solid rgba(255,255,255,0.15);
      border-radius: 8px; color: #e2e8f0;
      padding: 8px 12px; font-size: 13px; outline: none; width: 100%;
      transition: border-color .2s;
    }
    .input-styled::placeholder { color: rgba(255,255,255,0.3); }
    .input-styled:focus { border-color: #7086fb; }
    .modal-overlay {
      position: fixed; inset: 0; z-index: 50;
      background: rgba(0,0,0,.6); backdrop-filter: blur(4px);
      display: flex; align-items: center; justify-content: center;
      opacity: 0; pointer-events: none; transition: opacity .25s;
    }
    .modal-overlay.open { opacity: 1; pointer-events: all; }
    .modal-box {
      background: #1a1d30; border: 1px solid rgba(255,255,255,.12);
      border-radius: 20px; padding: 32px; width: min(96vw, 520px);
      transform: scale(.95); transition: transform .25s;
    }
    .modal-overlay.open .modal-box { transform: scale(1); }
    .tab-btn {
      padding: 6px 16px; border-radius: 8px; font-size: 13px; font-weight: 500;
      cursor: pointer; border: none; transition: background .2s, color .2s;
      background: transparent; color: #94a3b8;
    }
    .tab-btn.active { background: rgba(79,96,245,.25); color: #7086fb; }
    .tab-btn:hover:not(.active) { background: rgba(255,255,255,.05); color: #e2e8f0; }
    .channel-tag {
      display: flex; align-items: center; gap: 8px;
      background: rgba(255,255,255,.06); border: 1px solid rgba(255,255,255,.1);
      border-radius: 8px; padding: 8px 12px; margin-bottom: 8px;
      font-size: 13px; color: #e2e8f0;
    }
    .channel-dot { width:10px; height:10px; border-radius:50%; flex-shrink:0; }
    .sparkline { height: 48px; }
    ::-webkit-scrollbar { width: 6px; height: 6px; }
    ::-webkit-scrollbar-track { background: transparent; }
    ::-webkit-scrollbar-thumb { background: rgba(255,255,255,.15); border-radius: 3px; }
    canvas { max-width: 100%; }
    @keyframes fadeIn { from{opacity:0;transform:translateY(8px)} to{opacity:1;transform:none} }
    .fade-in { animation: fadeIn .4s ease forwards; }
  </style>
</head>
<body class="bg-[#0f1020] text-slate-100 min-h-screen">

<!-- ───────── NAV ───────── -->
<nav class="sticky top-0 z-40 bg-[#0f1020]/80 backdrop-blur border-b border-white/5 px-4 sm:px-8 py-3 flex items-center justify-between">
  <div class="flex items-center gap-3">
    <div class="w-8 h-8 rounded-lg bg-gradient-to-br from-brand-500 to-brand-300 flex items-center justify-center text-white font-bold text-sm">2H</div>
    <span class="font-700 text-base tracking-tight">2HH <span class="text-brand-400">Marketing</span></span>
    <span class="hidden sm:block text-slate-500 text-xs ml-1">Data Dashboard</span>
  </div>
  <div class="flex items-center gap-3">
    <span id="live-clock" class="text-slate-400 text-xs font-mono hidden sm:block"></span>
    <button onclick="openModal()" class="btn-primary flex items-center gap-1.5">
      <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4"/></svg>
      <span class="hidden sm:inline">Admin Settings</span>
      <span class="sm:hidden">設定</span>
    </button>
  </div>
</nav>

<!-- ───────── DATE BAR ───────── -->
<div class="px-4 sm:px-8 pt-6 pb-2 flex flex-wrap items-center justify-between gap-3">
  <div>
    <h1 class="text-xl sm:text-2xl font-800 text-white">業務數據總覽</h1>
    <p class="text-slate-400 text-sm mt-0.5">三分店即時數據 · <span id="date-label"></span></p>
  </div>
  <div class="flex items-center gap-2">
    <div class="w-2 h-2 rounded-full bg-emerald-400 animate-pulse"></div>
    <span class="text-xs text-slate-400">即時更新中</span>
  </div>
</div>

<!-- ───────── MAIN ───────── -->
<main class="px-4 sm:px-8 pb-12 space-y-6 mt-4">

  <!-- ── SUMMARY CARDS ── -->
  <section class="grid grid-cols-1 sm:grid-cols-2 xl:grid-cols-4 gap-4 fade-in">

    <!-- Total CON -->
    <div class="card col-span-1 sm:col-span-2 xl:col-span-1 flex flex-col gap-3"
         style="background:linear-gradient(135deg,rgba(79,96,245,.25),rgba(112,134,251,.1));border-color:rgba(79,96,245,.4)">
      <div class="flex items-center justify-between">
        <span class="text-slate-300 text-sm font-500">每日總 CON 數</span>
        <span class="badge badge-up">▲ 8.3%</span>
      </div>
      <div class="text-5xl font-800 text-white tracking-tight" id="total-con">247</div>
      <div class="text-slate-400 text-xs">vs 昨日 228 筆</div>
      <div id="sparkline-total" class="sparkline mt-1"></div>
    </div>

    <!-- Seoul -->
    <div class="card flex flex-col gap-3">
      <div class="flex items-center justify-between">
        <span class="text-slate-300 text-sm font-500">🇰🇷 首爾 Seoul</span>
        <span class="badge badge-up">▲ 5.1%</span>
      </div>
      <div class="text-3xl font-700 text-white" id="seoul-con">89</div>
      <div class="text-slate-400 text-xs">佔比 <span class="text-brand-400 font-600">36%</span></div>
      <div id="sparkline-seoul" class="sparkline mt-1"></div>
    </div>

    <!-- Hsinchu -->
    <div class="card flex flex-col gap-3">
      <div class="flex items-center justify-between">
        <span class="text-slate-300 text-sm font-500">🏙️ 新竹 Hsinchu</span>
        <span class="badge badge-up">▲ 12.4%</span>
      </div>
      <div class="text-3xl font-700 text-white" id="hsinchu-con">94</div>
      <div class="text-slate-400 text-xs">佔比 <span class="text-emerald-400 font-600">38%</span></div>
      <div id="sparkline-hsinchu" class="sparkline mt-1"></div>
    </div>

    <!-- Taichung -->
    <div class="card flex flex-col gap-3">
      <div class="flex items-center justify-between">
        <span class="text-slate-300 text-sm font-500">🏔️ 台中 Taichung</span>
        <span class="badge badge-down">▼ 2.7%</span>
      </div>
      <div class="text-3xl font-700 text-white" id="taichung-con">64</div>
      <div class="text-slate-400 text-xs">佔比 <span class="text-violet-400 font-600">26%</span></div>
      <div id="sparkline-taichung" class="sparkline mt-1"></div>
    </div>
  </section>

  <!-- ── ROW 2: PIE + REVENUE ── -->
  <section class="grid grid-cols-1 lg:grid-cols-2 gap-4 fade-in" style="animation-delay:.08s">

    <!-- New vs Returning -->
    <div class="card flex flex-col gap-4">
      <div class="flex flex-wrap items-center justify-between gap-2">
        <div>
          <h2 class="font-600 text-white text-base">新舊客比例</h2>
          <p class="text-slate-400 text-xs mt-0.5">New vs. Returning Customers</p>
        </div>
        <select id="nrFilter" class="select-styled" onchange="updateNRChart()">
          <option value="all">全部分店</option>
          <option value="seoul">首爾 Seoul</option>
          <option value="hsinchu">新竹 Hsinchu</option>
          <option value="taichung">台中 Taichung</option>
        </select>
      </div>
      <div class="flex items-center justify-center" style="height:260px">
        <canvas id="nrChart"></canvas>
      </div>
      <div class="flex justify-center gap-6 text-xs text-slate-400">
        <div class="flex items-center gap-1.5"><span class="w-2.5 h-2.5 rounded-full bg-brand-400 inline-block"></span>新客 New</div>
        <div class="flex items-center gap-1.5"><span class="w-2.5 h-2.5 rounded-full bg-emerald-400 inline-block"></span>回頭客 Returning</div>
      </div>
    </div>

    <!-- Revenue -->
    <div class="card flex flex-col gap-4">
      <div class="flex flex-wrap items-center justify-between gap-2">
        <div>
          <h2 class="font-600 text-white text-base">每週營業額趨勢</h2>
          <p class="text-slate-400 text-xs mt-0.5">Weekly Revenue Trend (NT$)</p>
        </div>
        <div class="flex items-center gap-2">
          <div class="flex gap-1 bg-white/5 rounded-lg p-0.5">
            <button class="tab-btn active" onclick="switchRevView('line',this)">折線</button>
            <button class="tab-btn" onclick="switchRevView('bar',this)">長條</button>
          </div>
          <select id="revFilter" class="select-styled" onchange="updateRevenueChart()">
            <option value="all">全部分店</option>
            <option value="seoul">首爾</option>
            <option value="hsinchu">新竹</option>
            <option value="taichung">台中</option>
          </select>
        </div>
      </div>
      <div style="height:260px;position:relative">
        <canvas id="revenueChart"></canvas>
      </div>
    </div>
  </section>

  <!-- ── ROW 3: CHANNEL + MONTHLY SUMMARY ── -->
  <section class="grid grid-cols-1 lg:grid-cols-3 gap-4 fade-in" style="animation-delay:.16s">

    <!-- Channel Chart -->
    <div class="card lg:col-span-2 flex flex-col gap-4">
      <div class="flex flex-wrap items-center justify-between gap-2">
        <div>
          <h2 class="font-600 text-white text-base">客人來源分析</h2>
          <p class="text-slate-400 text-xs mt-0.5">Customer Acquisition Channels</p>
        </div>
        <div class="flex items-center gap-2">
          <div class="flex gap-1 bg-white/5 rounded-lg p-0.5">
            <button class="tab-btn active" onclick="switchChView('bar',this)">長條</button>
            <button class="tab-btn" onclick="switchChView('pie',this)">圓餅</button>
          </div>
          <button onclick="openModal()" class="btn-primary text-xs flex items-center gap-1">
            <svg class="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M12 4v16m8-8H4"/></svg>新增來源
          </button>
        </div>
      </div>
      <div style="height:280px;position:relative">
        <canvas id="channelChart"></canvas>
      </div>
    </div>

    <!-- KPI Tiles -->
    <div class="flex flex-col gap-4">
      <div class="card flex flex-col gap-2" style="background:linear-gradient(135deg,rgba(52,211,153,.15),rgba(16,185,129,.05));border-color:rgba(52,211,153,.3)">
        <span class="text-slate-300 text-xs font-500">本月累計營業額</span>
        <div class="text-2xl font-700 text-emerald-400">NT$ 1,284,500</div>
        <div class="text-slate-400 text-xs flex items-center gap-1"><span class="badge badge-up">▲ 14.2%</span> vs 上月</div>
      </div>
      <div class="card flex flex-col gap-2" style="background:linear-gradient(135deg,rgba(167,139,250,.15),rgba(139,92,246,.05));border-color:rgba(167,139,250,.3)">
        <span class="text-slate-300 text-xs font-500">平均客單價</span>
        <div class="text-2xl font-700 text-violet-400">NT$ 2,380</div>
        <div class="text-slate-400 text-xs flex items-center gap-1"><span class="badge badge-up">▲ 3.8%</span> vs 上月</div>
      </div>
      <div class="card flex flex-col gap-2" style="background:linear-gradient(135deg,rgba(251,191,36,.15),rgba(245,158,11,.05));border-color:rgba(251,191,36,.3)">
        <span class="text-slate-300 text-xs font-500">本月新增會員</span>
        <div class="text-2xl font-700 text-amber-400">1,847</div>
        <div class="text-slate-400 text-xs flex items-center gap-1"><span class="badge badge-up">▲ 22.1%</span> vs 上月</div>
      </div>
      <div class="card flex flex-col gap-2">
        <span class="text-slate-300 text-xs font-500">回頭率 Return Rate</span>
        <div class="text-2xl font-700 text-brand-400">61.4%</div>
        <div class="w-full bg-white/10 rounded-full h-1.5 mt-1">
          <div class="bg-brand-400 h-1.5 rounded-full" style="width:61.4%"></div>
        </div>
      </div>
    </div>
  </section>

  <!-- ── ROW 4: DAILY BREAKDOWN TABLE ── -->
  <section class="card fade-in" style="animation-delay:.24s">
    <div class="flex flex-wrap items-center justify-between gap-2 mb-4">
      <div>
        <h2 class="font-600 text-white text-base">近7日各分店 CON 明細</h2>
        <p class="text-slate-400 text-xs mt-0.5">Daily Branch Conversion Breakdown</p>
      </div>
    </div>
    <div class="overflow-x-auto">
      <table class="w-full text-sm">
        <thead>
          <tr class="border-b border-white/10">
            <th class="text-left py-2 px-3 text-slate-400 font-500 text-xs">日期</th>
            <th class="text-right py-2 px-3 text-slate-400 font-500 text-xs">🇰🇷 首爾</th>
            <th class="text-right py-2 px-3 text-slate-400 font-500 text-xs">🏙️ 新竹</th>
            <th class="text-right py-2 px-3 text-slate-400 font-500 text-xs">🏔️ 台中</th>
            <th class="text-right py-2 px-3 text-slate-400 font-500 text-xs">合計</th>
            <th class="text-right py-2 px-3 text-slate-400 font-500 text-xs">日環比</th>
          </tr>
        </thead>
        <tbody id="conTable"></tbody>
      </table>
    </div>
  </section>

</main>

<!-- ───────── ADMIN MODAL ───────── -->
<div id="adminModal" class="modal-overlay" onclick="closeModalOutside(event)">
  <div class="modal-box">
    <div class="flex items-center justify-between mb-6">
      <div>
        <h3 class="text-white font-700 text-lg">Admin Settings</h3>
        <p class="text-slate-400 text-xs mt-0.5">管理客人來源渠道</p>
      </div>
      <button onclick="closeModal()" class="text-slate-500 hover:text-white transition-colors">
        <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"/></svg>
      </button>
    </div>

    <!-- Existing channels -->
    <div class="mb-5">
      <p class="text-slate-300 text-xs font-600 mb-2 uppercase tracking-wider">目前來源渠道</p>
      <div id="channelList"></div>
    </div>

    <!-- Add new channel -->
    <div class="border-t border-white/10 pt-5">
      <p class="text-slate-300 text-xs font-600 mb-3 uppercase tracking-wider">新增來源渠道</p>
      <div class="flex gap-2 mb-3">
        <input id="newChannelName" class="input-styled" placeholder="渠道名稱 (e.g. Instagram)" />
        <button onclick="addChannel()" class="btn-primary whitespace-nowrap">新增來源</button>
      </div>
      <div class="flex gap-2">
        <input id="newChannelData" class="input-styled" placeholder="數量 (e.g. 45)" type="number" min="0" />
      </div>
      <p class="text-slate-500 text-xs mt-2">新增後圖表即時更新 · 資料儲存於本地 Session</p>
    </div>
  </div>
</div>

<script>
// ─── DATA ──────────────────────────────────────────────────────────────────
const PALETTE = [
  '#7086fb','#34d399','#a78bfa','#fb923c','#f472b6','#38bdf8','#facc15',
  '#4ade80','#c084fc','#fb7185','#22d3ee','#fbbf24'
];

const weekdays = ['週一','週二','週三','週四','週五','週六','週日'];

const revenueData = {
  all:     [128500,145200,132800,168400,154700,189300,201100],
  seoul:   [ 46000, 52000, 47500, 60000, 55000, 68000, 72000],
  hsinchu: [ 48500, 54200, 49300, 63400, 58700, 72300, 77100],
  taichung:[ 34000, 39000, 36000, 45000, 41000, 49000, 52000],
};

const nrData = {
  all:     { new: 142, ret: 105 },
  seoul:   { new:  51, ret:  38 },
  hsinchu: { new:  56, ret:  38 },
  taichung:{ new:  35, ret:  29 },
};

const defaultChannels = [
  { name: 'Google',  value: 98,  color: PALETTE[0] },
  { name: 'FB',      value: 74,  color: PALETTE[1] },
  { name: '口碑',    value: 53,  color: PALETTE[2] },
];

const tableData = [
  { date:'06/20', seoul:82,  hsinchu:88, taichung:61 },
  { date:'06/21', seoul:79,  hsinchu:91, taichung:58 },
  { date:'06/22', seoul:85,  hsinchu:87, taichung:66 },
  { date:'06/23', seoul:90,  hsinchu:95, taichung:70 },
  { date:'06/24', seoul:84,  hsinchu:89, taichung:62 },
  { date:'06/25', seoul:88,  hsinchu:92, taichung:65 },
  { date:'06/26', seoul:89,  hsinchu:94, taichung:64 },
];

// ─── STATE ─────────────────────────────────────────────────────────────────
let channels = JSON.parse(sessionStorage.getItem('channels') || 'null') || defaultChannels.map(c=>({...c}));
let channelChartType = 'bar';
let revenueChartType = 'line';

function saveChannels() {
  sessionStorage.setItem('channels', JSON.stringify(channels));
}

// ─── INIT ──────────────────────────────────────────────────────────────────
document.addEventListener('DOMContentLoaded', () => {
  setDateLabel();
  startClock();
  buildTable();
  buildSparklines();
  initNRChart();
  initRevenueChart();
  initChannelChart();
  renderChannelList();
});

function setDateLabel() {
  const d = new Date();
  document.getElementById('date-label').textContent =
    d.toLocaleDateString('zh-TW', {year:'numeric',month:'long',day:'numeric',weekday:'long'});
}

function startClock() {
  const el = document.getElementById('live-clock');
  const tick = () => {
    el.textContent = new Date().toLocaleTimeString('zh-TW', {hour12:false});
  };
  tick(); setInterval(tick, 1000);
}

// ─── TABLE ─────────────────────────────────────────────────────────────────
function buildTable() {
  const tbody = document.getElementById('conTable');
  let prev = null;
  tbody.innerHTML = tableData.map(r => {
    const total = r.seoul + r.hsinchu + r.taichung;
    let badge = '';
    if (prev !== null) {
      const diff = total - prev;
      const pct  = ((diff / prev) * 100).toFixed(1);
      badge = diff >= 0
        ? `<span class="badge badge-up text-xs">▲ ${pct}%</span>`
        : `<span class="badge badge-down text-xs">▼ ${Math.abs(pct)}%</span>`;
    }
    prev = total;
    const isToday = r.date === '06/26';
    return `<tr class="border-b border-white/5 hover:bg-white/5 transition-colors ${isToday?'bg-brand-500/10':''}">
      <td class="py-2.5 px-3 font-500 text-slate-200 text-xs">${r.date}${isToday?' <span class="text-brand-400 text-xs">(今日)</span>':''}</td>
      <td class="py-2.5 px-3 text-right text-slate-200 text-xs">${r.seoul}</td>
      <td class="py-2.5 px-3 text-right text-slate-200 text-xs">${r.hsinchu}</td>
      <td class="py-2.5 px-3 text-right text-slate-200 text-xs">${r.taichung}</td>
      <td class="py-2.5 px-3 text-right font-600 text-white text-xs">${total}</td>
      <td class="py-2.5 px-3 text-right text-xs">${badge}</td>
    </tr>`;
  }).join('');
}

// ─── SPARKLINES ────────────────────────────────────────────────────────────
function sparkline(id, data, color) {
  const el = document.getElementById(id);
  const canvas = document.createElement('canvas');
  canvas.height = 48;
  el.appendChild(canvas);
  new Chart(canvas, {
    type: 'line',
    data: {
      labels: ['','','','','','',''],
      datasets: [{ data, borderColor: color, borderWidth: 2,
        pointRadius: 0, fill: true,
        backgroundColor: color.replace(')',',0.15)').replace('rgb','rgba') }]
    },
    options: {
      responsive: true, maintainAspectRatio: false,
      plugins: { legend: { display: false }, tooltip: { enabled: false } },
      scales: { x: { display: false }, y: { display: false } },
      animation: { duration: 800 }
    }
  });
}

function buildSparklines() {
  const totalVals = tableData.map(r => r.seoul+r.hsinchu+r.taichung);
  sparkline('sparkline-total',   totalVals,          '#7086fb');
  sparkline('sparkline-seoul',   tableData.map(r=>r.seoul),    '#7086fb');
  sparkline('sparkline-hsinchu', tableData.map(r=>r.hsinchu),  '#34d399');
  sparkline('sparkline-taichung',tableData.map(r=>r.taichung), '#a78bfa');
}

// ─── NR CHART ──────────────────────────────────────────────────────────────
let nrChart;
function initNRChart() {
  const ctx = document.getElementById('nrChart').getContext('2d');
  const d = nrData.all;
  nrChart = new Chart(ctx, {
    type: 'doughnut',
    data: {
      labels: ['新客 New', '回頭客 Returning'],
      datasets: [{
        data: [d.new, d.ret],
        backgroundColor: ['#7086fb','#34d399'],
        borderColor: 'transparent',
        hoverOffset: 8
      }]
    },
    options: {
      responsive: true, maintainAspectRatio: false,
      cutout: '65%',
      plugins: {
        legend: { display: false },
        tooltip: {
          backgroundColor: '#1e2035', borderColor: 'rgba(255,255,255,.1)', borderWidth: 1,
          titleColor: '#94a3b8', bodyColor: '#e2e8f0',
          callbacks: {
            label: ctx => {
              const total = ctx.dataset.data.reduce((a,b)=>a+b,0);
              return ` ${ctx.label}: ${ctx.raw} (${((ctx.raw/total)*100).toFixed(1)}%)`;
            }
          }
        }
      }
    }
  });
}

function updateNRChart() {
  const key = document.getElementById('nrFilter').value;
  const d = nrData[key];
  nrChart.data.datasets[0].data = [d.new, d.ret];
  nrChart.update('active');
}

// ─── REVENUE CHART ─────────────────────────────────────────────────────────
let revenueChart;
function initRevenueChart() {
  const ctx = document.getElementById('revenueChart').getContext('2d');
  const data = revenueData.all;
  const grad = ctx.createLinearGradient(0,0,0,260);
  grad.addColorStop(0,'rgba(112,134,251,0.3)');
  grad.addColorStop(1,'rgba(112,134,251,0)');
  revenueChart = new Chart(ctx, {
    type: 'line',
    data: {
      labels: weekdays,
      datasets: [{
        label: '營業額 (NT$)', data,
        borderColor: '#7086fb', backgroundColor: grad,
        borderWidth: 2.5, pointRadius: 4, pointBackgroundColor: '#7086fb',
        fill: true, tension: 0.4
      }]
    },
    options: {
      responsive: true, maintainAspectRatio: false,
      plugins: {
        legend: { display: false },
        tooltip: {
          backgroundColor: '#1e2035', borderColor: 'rgba(255,255,255,.1)', borderWidth: 1,
          titleColor: '#94a3b8', bodyColor: '#e2e8f0',
          callbacks: { label: c => ` NT$ ${c.raw.toLocaleString()}` }
        }
      },
      scales: {
        x: { grid: { color: 'rgba(255,255,255,.06)' }, ticks: { color: '#64748b', font:{size:11} } },
        y: {
          grid: { color: 'rgba(255,255,255,.06)' }, ticks: { color: '#64748b', font:{size:11},
          callback: v => 'NT$ '+v.toLocaleString() }
        }
      }
    }
  });
}

function updateRevenueChart() {
  const key = document.getElementById('revFilter').value;
  revenueChart.data.datasets[0].data = revenueData[key];
  revenueChart.update('active');
}

function switchRevView(type, btn) {
  document.querySelectorAll('#revenueChart ~ .tab-btn, .tab-btn').forEach(b=>{});
  btn.closest('.flex').querySelectorAll('.tab-btn').forEach(b=>b.classList.remove('active'));
  btn.classList.add('active');
  revenueChartType = type;
  revenueChart.config.type = type;
  if (type === 'bar') {
    revenueChart.data.datasets[0].backgroundColor = '#7086fb';
    revenueChart.data.datasets[0].fill = false;
  } else {
    const ctx = document.getElementById('revenueChart').getContext('2d');
    const grad = ctx.createLinearGradient(0,0,0,260);
    grad.addColorStop(0,'rgba(112,134,251,0.3)');
    grad.addColorStop(1,'rgba(112,134,251,0)');
    revenueChart.data.datasets[0].backgroundColor = grad;
    revenueChart.data.datasets[0].fill = true;
  }
  revenueChart.destroy();
  const ctx2 = document.getElementById('revenueChart').getContext('2d');
  revenueChart = new Chart(ctx2, {
    type,
    data: {
      labels: weekdays,
      datasets: [{
        label: '營業額 (NT$)',
        data: revenueData[document.getElementById('revFilter').value],
        borderColor: '#7086fb',
        backgroundColor: type==='bar' ? '#7086fb' : (() => {
          const g = ctx2.createLinearGradient(0,0,0,260);
          g.addColorStop(0,'rgba(112,134,251,0.3)'); g.addColorStop(1,'rgba(112,134,251,0)'); return g;
        })(),
        borderWidth: 2.5, pointRadius: type==='bar'?0:4,
        pointBackgroundColor: '#7086fb', fill: type==='line', tension: 0.4,
        borderRadius: type==='bar'?6:0,
      }]
    },
    options: revenueChart.options
  });
}

// ─── CHANNEL CHART ─────────────────────────────────────────────────────────
let channelChart;
function initChannelChart() {
  const ctx = document.getElementById('channelChart').getContext('2d');
  channelChart = new Chart(ctx, buildChannelConfig());
}

function buildChannelConfig() {
  const labels = channels.map(c=>c.name);
  const data   = channels.map(c=>c.value);
  const colors = channels.map(c=>c.color);
  const type   = channelChartType;
  return {
    type,
    data: {
      labels,
      datasets: [{
        label: '來源人數',
        data, backgroundColor: colors,
        borderColor: 'transparent',
        borderWidth: 0,
        borderRadius: type==='bar'?6:0,
        hoverOffset: type==='pie'?10:0,
      }]
    },
    options: {
      responsive: true, maintainAspectRatio: false,
      plugins: {
        legend: {
          display: type!=='bar',
          labels: { color:'#94a3b8', font:{size:12}, padding:16,
            usePointStyle:true, pointStyleWidth:8 }
        },
        tooltip: {
          backgroundColor: '#1e2035', borderColor: 'rgba(255,255,255,.1)', borderWidth: 1,
          titleColor: '#94a3b8', bodyColor: '#e2e8f0',
          callbacks: {
            label: c => {
              if (type==='pie') {
                const total = c.dataset.data.reduce((a,b)=>a+b,0);
                return ` ${c.label}: ${c.raw} (${((c.raw/total)*100).toFixed(1)}%)`;
              }
              return ` ${c.raw} 人`;
            }
          }
        }
      },
      scales: type==='bar' ? {
        x: { grid:{color:'rgba(255,255,255,.06)'}, ticks:{color:'#64748b',font:{size:12}} },
        y: { grid:{color:'rgba(255,255,255,.06)'}, ticks:{color:'#64748b',font:{size:11}} }
      } : {}
    }
  };
}

function refreshChannelChart() {
  channelChart.destroy();
  channelChart = new Chart(document.getElementById('channelChart').getContext('2d'), buildChannelConfig());
}

function switchChView(type, btn) {
  btn.closest('.flex').querySelectorAll('.tab-btn').forEach(b=>b.classList.remove('active'));
  btn.classList.add('active');
  channelChartType = type;
  refreshChannelChart();
}

// ─── CHANNEL MANAGEMENT ────────────────────────────────────────────────────
function renderChannelList() {
  const el = document.getElementById('channelList');
  el.innerHTML = channels.map((c,i) => `
    <div class="channel-tag">
      <span class="channel-dot" style="background:${c.color}"></span>
      <span class="flex-1 font-500">${c.name}</span>
      <input type="number" class="input-styled w-20 text-center text-xs py-1 px-2"
        value="${c.value}" min="0"
        onchange="updateChannelValue(${i},this.value)" />
      <span class="text-slate-500 text-xs">人</span>
      ${i >= 3 ? `<button class="btn-danger" onclick="removeChannel(${i})">刪除</button>` : ''}
    </div>`).join('');
}

function updateChannelValue(i, val) {
  channels[i].value = parseInt(val) || 0;
  saveChannels(); refreshChannelChart();
}

function removeChannel(i) {
  channels.splice(i, 1);
  saveChannels(); renderChannelList(); refreshChannelChart();
}

function addChannel() {
  const nameEl = document.getElementById('newChannelName');
  const dataEl = document.getElementById('newChannelData');
  const name   = nameEl.value.trim();
  const value  = parseInt(dataEl.value) || 0;
  if (!name) { nameEl.focus(); return; }
  if (channels.find(c=>c.name.toLowerCase()===name.toLowerCase())) {
    nameEl.value=''; nameEl.placeholder='⚠ 已存在此渠道'; return;
  }
  channels.push({ name, value, color: PALETTE[channels.length % PALETTE.length] });
  saveChannels(); renderChannelList(); refreshChannelChart();
  nameEl.value=''; dataEl.value='';
  nameEl.placeholder='渠道名稱 (e.g. Instagram)';
}

// ─── MODAL ─────────────────────────────────────────────────────────────────
function openModal()  {
  renderChannelList();
  document.getElementById('adminModal').classList.add('open');
}
function closeModal() { document.getElementById('adminModal').classList.remove('open'); }
function closeModalOutside(e) { if (e.target.id==='adminModal') closeModal(); }

document.addEventListener('keydown', e => { if (e.key==='Escape') closeModal(); });

// ─── Enter key in modal ─────────────────────────────────────────────────────
document.getElementById('newChannelName').addEventListener('keydown', e => {
  if (e.key==='Enter') addChannel();
});
document.getElementById('newChannelData').addEventListener('keydown', e => {
  if (e.key==='Enter') addChannel();
});
</script>
</body>
</html>
