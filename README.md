<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <title>IBM - Simple Real-Time Stock Ticker</title>
  <style>
    body{font-family: Arial, sans-serif;background:#f4f6f8;color:#111;display:flex;align-items:center;justify-content:center;height:100vh;margin:0}
    .card{background:#fff;border:1px solid #dde3ea;padding:18px;border-radius:8px;box-shadow:0 6px 18px rgba(20,30,40,0.06);width:320px}
    h1{font-size:16px;margin:0 0 10px 0}
    .row{display:flex;justify-content:space-between;align-items:center;margin:8px 0}
    .price{font-size:28px;font-weight:700}
    .change{padding:4px 8px;border-radius:6px;font-weight:600}
    .muted{color:#5b6b78;font-size:13px}
    button{padding:8px 12px;border-radius:6px;border:1px solid #cbd6e0;background:#f7fafc;cursor:pointer}
    button.primary{background:#0b6efd;color:#fff;border:none}
  </style>
</head>
<body>
  <div class="card" role="region" aria-label="IBM simple stock ticker">
    <h1>IBM - Simple Real-Time Stock Ticker</h1>

    <div class="row">
      <div>
        <div class="muted">Symbol</div>
        <div id="symbol">IBM</div>
      </div>
      <div>
        <div class="muted">Last update</div>
        <div id="time">-</div>
      </div>
    </div>

    <div class="row" style="margin-top:12px">
      <div>
        <div class="muted">Price</div>
        <div id="price" class="price">-</div>
      </div>
      <div>
        <div class="muted">Change</div>
        <div id="change" class="change">-</div>
      </div>
    </div>

    <div class="row" style="margin-top:12px">
      <div class="muted">Volume</div>
      <div id="volume" class="muted">-</div>
    </div>

    <div style="display:flex;gap:8px;margin-top:14px">
      <button id="startBtn" class="primary">Start</button>
      <button id="stopBtn">Stop</button>
    </div>

    <p class="muted" style="margin-top:12px;font-size:12px">
      Note: Uses Yahoo Finance public endpoint. If you see CORS errors, run from a server or use a proxy.
    </p>
  </div>

<script>
  const API = 'https://query1.finance.yahoo.com/v7/finance/quote?symbols=IBM';
  const symbolEl = document.getElementById('symbol');
  const priceEl = document.getElementById('price');
  const changeEl = document.getElementById('change');
  const timeEl = document.getElementById('time');
  const volumeEl = document.getElementById('volume');
  const startBtn = document.getElementById('startBtn');
  const stopBtn = document.getElementById('stopBtn');

  let timer = null;

  function formatNum(n, d=2){
    return (n === null || n === undefined) ? '-' : Number(n).toLocaleString(undefined, {minimumFractionDigits:d, maximumFractionDigits:d});
  }

  async function fetchQuote(){
    try {
      const res = await fetch(API, {cache:'no-store'});
      if (!res.ok) throw new Error('network');
      const json = await res.json();
      const q = json?.quoteResponse?.result?.[0];
      if (!q) throw new Error('parse');

      const price = q.regularMarketPrice ?? q.ask ?? 0;
      const change = q.regularMarketChange ?? 0;
      const changePct = q.regularMarketChangePercent ?? 0;
      const vol = q.regularMarketVolume ?? 0;
      const time = q.regularMarketTime ?? Math.floor(Date.now()/1000);

      updateUI(price, change, changePct, vol, time);
    } catch (e) {
      // fallback simulated update
      simulate();
    }
  }

  function simulate(){
    // Simple random walk simulation so UI updates even when fetch fails
    const last = parseFloat(priceEl.textContent.replace(/,/g,'')) || 135;
    const drift = (Math.random() - 0.5) * 0.5;
    const newPrice = Math.max(0.01, +(last + drift).toFixed(2));
    const change = +(newPrice - last).toFixed(2);
    const changePct = last ? +(change / last * 100).toFixed(2) : 0;
    const vol = Math.round(1000 + Math.random()*50000);
    const time = Math.floor(Date.now()/1000);
    updateUI(newPrice, change, changePct, vol, time, true);
  }

  function updateUI(price, change, changePct, vol, time, simulated=false){
    priceEl.textContent = formatNum(price,2);
    const sign = change > 0 ? '+' : '';
    changeEl.textContent = `${sign}${formatNum(change,2)} (${formatNum(changePct,2)}%)`;
    changeEl.style.background = change > 0 ? 'rgba(16,185,129,0.12)' : (change < 0 ? 'rgba(239,68,68,0.08)' : 'transparent');
    changeEl.style.color = change > 0 ? '#108b5a' : (change < 0 ? '#b02a2a' : '#5b6b78');
    timeEl.textContent = new Date(time * 1000).toLocaleTimeString();
    volumeEl.textContent = vol ? Number(vol).toLocaleString() : '-';
    if (simulated) {
      // small visual indicator when simulated
      symbolEl.textContent = 'IBM (simulated)';
    } else {
      symbolEl.textContent = 'IBM';
    }
  }

  function start(){
    if (timer) return;
    fetchQuote();
    timer = setInterval(fetchQuote, 5000);
    startBtn.disabled = true;
    stopBtn.disabled = false;
  }

  function stop(){
    if (!timer) return;
    clearInterval(timer);
    timer = null;
    startBtn.disabled = false;
    stopBtn.disabled = true;
  }

  // initial state
  stopBtn.disabled = true;
  startBtn.addEventListener('click', start);
  stopBtn.addEventListener('click', stop);

  // auto-start once if you want:
  // start();
</script>
</body>
</html>
