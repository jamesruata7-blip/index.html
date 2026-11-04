# index.html 

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Forex Trading Robot</title>
<style>
body {
  background:#101827;color:#fff;
  font-family:Arial,sans-serif;
  padding:20px;max-width:650px;margin:auto
}
.card {
  background:#1d293b;padding:18px;
  border-radius:14px;margin:14px 0
}
h1 {color:#59d6ad}
input,textarea,button {
  box-sizing:border-box;width:100%;
  padding:13px;margin:7px 0;
  border-radius:8px;border:0;font-size:15px
}
textarea {height:105px}
button {background:#18a77c;color:white;font-weight:bold}
.signal {font-size:27px;font-weight:bold;text-align:center;padding:15px}
.small {color:#aebbd0;font-size:13px}
</style>
</head>
<body>

<h1>🤖 Forex Trading Robot</h1>
<p>RSI 14 + EMA 20 | Demo analysis only</p>

<div class="card">
<h3>Market settings</h3>
<p>Currency pair</p>
<select id="pair">
<option>EUR/USD</option>
<option>GBP/USD</option>
<option>USD/JPY</option>
<option>USD/CHF</option>
<option>AUD/USD</option>
</select>
<p>Timeframe: 1 minute</p>
</div>

<div class="card">
<h3>Enter closing prices</h3>
<p class="small">
Paste at least 21 closing prices, oldest first,
separated by commas.
</p>
<textarea id="prices"
placeholder="1.1500, 1.1502, 1.1498, ..."></textarea>
<button onclick="analyse()">ANALYSE MARKET</button>
</div>

<div class="card">
<h3>Robot signal</h3>
<div class="signal" id="signal">WAITING</div>
<p id="details">Enter closing prices to begin.</p>
<p id="rsi">RSI: --</p>
<p id="ema">EMA 20: --</p>
<p id="price">Latest close: --</p>
</div>

<div class="card">
<h3>Demo trading journal</h3>
<p class="small">Record signals and their outcomes manually.</p>
<button onclick="recordTrade()">RECORD CURRENT SIGNAL</button>
<div id="journal"></div>
</div>

<script>
let currentSignal = "WAIT";

function ema(values, period) {
  if(values.length < period) return null;
  let result = values.slice(0,period)
    .reduce((a,b)=>a+b,0)/period;
  let multiplier = 2/(period+1);
  for(let i=period;i<values.length;i++){
    result=(values[i]-result)*multiplier+result;
  }
  return result;
}

function rsi(values, period=14) {
  if(values.length <= period) return null;
  let gains=0, losses=0;
  for(let i=values.length-period;i<values.length;i++){
    let change=values[i]-values[i-1];
    if(change>0) gains+=change;
    else losses-=change;
  }
  if(losses===0) return gains===0?50:100;
  let rs=(gains/period)/(losses/period);
  return 100-(100/(1+rs));
}

function analyse(){
  const values=document.getElementById("prices")
    .value.split(/[\s,;]+/)
    .map(Number).filter(x=>Number.isFinite(x)&&x>0);

  if(values.length<21){
    alert("Please enter at least 21 valid closing prices.");
    return;
  }

  const last=values[values.length-1];
  const previous=values[values.length-2];
  const e=ema(values,20);
  const r=rsi(values,14);
  const oldR=rsi(values.slice(0,-1),14);

  currentSignal="WAIT";

  // Trend confirmation + RSI direction
  if(last>e && r>30 && r<55 && r>oldR){
    currentSignal="BUY";
  } else if(last<e && r<70 && r>45 && r<oldR){
    currentSignal="SELL";
  }

  const signal=document.getElementById("signal");
  signal.textContent=currentSignal;
  signal.style.color=currentSignal==="BUY"?"#59d6ad":
    currentSignal==="SELL"?"#ff7777":"#ffd166";

  document.getElementById("details").textContent=
    currentSignal==="BUY"?"Bullish conditions detected. Check the next candle.":
    currentSignal==="SELL"?"Bearish conditions detected. Check the next candle.":
    "No confirmed setup. Wait for better conditions.";

  document.getElementById("rsi").textContent=
    "RSI 14: "+r.toFixed(2);
  document.getElementById("ema").textContent=
    "EMA 20: "+e.toFixed(6);
  document.getElementById("price").textContent=
    "Latest close: "+last.toFixed(6);
}

function recordTrade(){
  if(currentSignal==="WAIT"){
    alert("There is no BUY or SELL signal to record.");
    return;
  }
  let trades=JSON.parse(localStorage.getItem("trades")||"[]");
  trades.unshift({
    pair:document.getElementById("pair").value,
    side:currentSignal,
    time:new Date().toLocaleString(),
    outcome:"Pending"
  });
  localStorage.setItem("trades",JSON.stringify(trades));
  showJournal();
}

function setOutcome(index,outcome){
  let trades=JSON.parse(localStorage.getItem("trades")||"[]");
  trades[index].outcome=outcome;
  localStorage.setItem("trades",JSON.stringify(trades));
  showJournal();
}

function showJournal(){
  let trades=JSON.parse(localStorage.getItem("trades")||"[]");
  document.getElementById("journal").innerHTML=
    trades.map((t,i)=>`
      <div class="card">
      <b>${t.pair} — ${t.side}</b>
      <p class="small">${t.time}</p>
      <p>Result: ${t.outcome}</p>
      <button onclick="setOutcome(${i},'Win')">WIN</button>
      <button onclick="setOutcome(${i},'Loss')">LOSS</button>
      </div>`).join("");
}
showJournal();
</script>
</body>
</html>

