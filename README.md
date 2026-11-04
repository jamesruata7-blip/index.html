
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Forex Paper & Signal Robot 2.0</title>
<style>
*{box-sizing:border-box}
body{margin:0;padding:18px;background:#101522;color:#f4f6fa;font-family:Arial,sans-serif}
.container{max-width:650px;margin:auto}
h1{color:#e8c66a;font-size:25px}
.card{background:#1b2435;padding:17px;border-radius:13px;margin:15px 0}
label{display:block;margin:12px 0 6px}
input,textarea,select{width:100%;padding:12px;border-radius:8px;border:1px solid #46536b;background:#111a29;color:white;font-size:15px}
textarea{min-height:95px}
button{padding:12px;border:0;border-radius:8px;font-weight:bold;cursor:pointer;margin:5px 3px}
.primary{background:#e8c66a;color:#151515}
.up{background:#168c62;color:white}
.down{background:#bd4552;color:white}
.neutral{background:#46536b;color:white}
.value{font-size:25px;font-weight:bold}
.green{color:#48d6a0}
.red{color:#ff7782}
.yellow{color:#e8c66a}
.small{font-size:13px;color:#aeb9cb;line-height:1.5}
.row{display:flex;justify-content:space-between;gap:12px;flex-wrap:wrap}
.row>*{flex:1}
.hidden{display:none}
table{width:100%;border-collapse:collapse;font-size:13px}
td,th{padding:9px;border-bottom:1px solid #39445a;text-align:left}
</style>
</head>
<body>
<div class="container">
<h1>Forex Paper & Signal Robot 2.0</h1>
<p class="small">RSI 14 + EMA 20 | Pocket Option-style paper simulator</p>

<div class="card">
  <div class="row">
    <button class="primary" onclick="setMode('demo')">DEMO MODE</button>
    <button class="neutral" onclick="setMode('real')">REAL-MARKET ANALYSIS</button>
  </div>
  <p id="modeText" class="green">DEMO MODE — virtual trades only</p>
  <p class="small">Real-market analysis does not place orders or connect to your broker.</p>
</div>

<div class="card">
  <h3>Market prices</h3>
  <label>Currency pair</label>
  <select id="pair">
    <option>EUR/USD</option>
    <option>GBP/USD</option>
    <option>USD/JPY</option>
    <option>USD/CHF</option>
    <option>AUD/USD</option>
  </select>
  <label>Closing prices — oldest first, newest last</label>
  <textarea id="prices" placeholder="Enter at least 21 closing prices, separated by commas"></textarea>
  <button class="primary" onclick="analyze()">ANALYZE MARKET</button>
  <p class="small">Use completed candles from one consistent price feed and timeframe. These prices are not fetched automatically.</p>
</div>

<div class="card">
  <h3>Robot analysis</h3>
  <div class="row">
    <div>Signal<div id="signal" class="value yellow">WAIT</div></div>
    <div>RSI 14<div id="rsi" class="value">—</div></div>
    <div>EMA 20<div id="ema" class="value">—</div></div>
  </div>
  <p>Latest close: <strong id="latest">—</strong></p>
  <p id="reason" class="small">Enter prices and analyze the market.</p>
</div>

<div class="card" id="demoPanel">
  <h3>Paper trading simulator</h3>
  <p>Virtual balance: <strong id="balance" class="value green">10,000.00</strong> credits</p>
  <label>Virtual stake</label>
  <input id="stake" type="number" min="1" value="100">
  <label>Assumed payout (%)</label>
  <input id="payout" type="number" min="1" max="100" value="80">
  <p class="small">Payout is a simulation assumption, not a promised return.</p>
  <div class="row">
    <button class="up" onclick="startTrade('UP')">START UP</button>
    <button class="down" onclick="startTrade('DOWN')">START DOWN</button>
  </div>
  <p id="tradeStatus" class="yellow">No open trade.</p>
  <p id="timer" class="value">—</p>
  <label>Expiry price (enter after 1 minute)</label>
  <input id="expiryPrice" type="number" step="any" placeholder="Enter final market price">
  <button class="primary" onclick="settleTrade()">SETTLE PAPER TRADE</button>
</div>

<div class="card">
  <h3>Paper trading history</h3>
  <p id="stats" class="small">Trades: 0 | Wins: 0 | Losses: 0</p>
  <table>
    <thead><tr><th>Pair</th><th>Direction</th><th>Result</th><th>P/L</th></tr></thead>
    <tbody id="history"></tbody>
  </table>
  <button class="neutral" onclick="resetDemo()">RESET DEMO BALANCE & HISTORY</button>
</div>

<div class="card">
  <h3>Safety</h3>
  <p class="small">
  This is an educational paper-trading simulator. It does not connect to Pocket Option, fetch live quotes, or execute real orders. Real-market analysis is signal-only. Demo outcomes depend on the prices you enter. A signal does not guarantee a winning trade.
  </p>
</div>
</div>

<script>
let mode="demo";
let marketSignal="WAIT";
let lastPrices=[];
let openTrade=null;
let countdown=null;

let balance=Number(localStorage.getItem("paperBalance") || 10000);
let history=JSON.parse(localStorage.getItem("paperHistory") || "[]");

function setMode(next){
  mode=next;
  document.getElementById("modeText").textContent=
    next==="demo" ? "DEMO MODE — virtual trades only" :
    "REAL-MARKET ANALYSIS — signals only, no orders";
  document.getElementById("modeText").className=
    next==="demo" ? "green" : "yellow";
  document.getElementById("demoPanel").classList.toggle("hidden",next!=="demo");
}

function ema(values,period){
  if(values.length<period)return null;
  let result=values.slice(0,period).reduce((a,b)=>a+b,0)/period;
  const multiplier=2/(period+1);
  for(let i=period;i<values.length;i++){
    result=(values[i]-result)*multiplier+result;
  }
  return result;
}

function rsi(values,period=14){
  if(values.length<=period)return null;
  let gains=0,losses=0;
  for(let i=1;i<=period;i++){
    const change=values[i]-values[i-1];
    if(change>0)gains+=change;
    else losses-=change;
  }
  let avgGain=gains/period;
  let avgLoss=losses/period;
  for(let i=period+1;i<values.length;i++){
    const change=values[i]-values[i-1];
    avgGain=(avgGain*(period-1)+Math.max(change,0))/period;
    avgLoss=(avgLoss*(period-1)+Math.max(-change,0))/period;
  }
  if(avgLoss===0)return 100;
  return 100-(100/(1+avgGain/avgLoss));
}

function analyze(){
  const raw=document.getElementById("prices").value.trim();
  const values=raw.split(/[,\s]+/).filter(Boolean).map(Number);
  if(values.length<21 || values.some(v=>!Number.isFinite(v)||v<=0)){
    alert("Enter at least 21 valid positive closing prices.");
    return;
  }
  lastPrices=values;
  const currentRSI=rsi(values);
  const previousRSI=rsi(values.slice(0,-1));
  const currentEMA=ema(values,20);
  const latest=values[values.length-1];
  let signal="WAIT";
  let reason="No confirmed setup. Wait for better conditions.";

  if(latest>currentEMA && currentRSI>=30 && currentRSI<=55 && currentRSI>previousRSI){
    signal="BUY";
    reason="Price above EMA 20; RSI is in the BUY range and rising.";
  }else if(latest<currentEMA && currentRSI>=45 && currentRSI<=70 && currentRSI<previousRSI){
    signal="SELL";
    reason="Price below EMA 20; RSI is in the SELL range and falling.";
  }

  marketSignal=signal;
  const signalEl=document.getElementById("signal");
  signalEl.textContent=signal;
  signalEl.className="value "+(signal==="BUY"?"green":signal==="SELL"?"red":"yellow");
  document.getElementById("rsi").textContent=currentRSI.toFixed(2);
  document.getElementById("ema").textContent=currentEMA.toFixed(6);
  document.getElementById("latest").textContent=latest;
  document.getElementById("reason").textContent=reason;
}

function startTrade(direction){
  if(mode!=="demo"){alert("Paper trades are only available in DEMO mode.");return;}
  if(openTrade){alert("You already have an open paper trade.");return;}
  if(lastPrices.length<21){alert("Analyze at least 21 closing prices first.");return;}

  const stake=Number(document.getElementById("stake").value);
  const payout=Number(document.getElementById("payout").value);
  if(!Number.isFinite(stake)||stake<=0||stake>balance){
    alert("Enter a valid stake within your virtual balance.");return;
  }
  if(!Number.isFinite(payout)||payout<=0||payout>100){
    alert("Enter a payout between 1 and 100.");return;
  }

  openTrade={
    direction,
    stake,
    payout,
    entry:lastPrices[lastPrices.length-1],
    pair:document.getElementById("pair").value,
    end:Date.now()+60000
  };
  balance-=stake;
  save();
  document.getElementById("tradeStatus").textContent=
    direction+" trade opened at "+openTrade.entry+
    ". Enter the market price after 1 minute.";
  tick();
  countdown=setInterval(tick,250);
}

function tick(){
  if(!openTrade)return;
  const seconds=Math.max(0,Math.ceil((openTrade.end-Date.now())/1000));
  document.getElementById("timer").textContent=seconds+" seconds remaining";
  if(seconds===0){
    clearInterval(countdown);
    countdown=null;
    document.getElementById("tradeStatus").textContent=
      "Expiry reached. Enter the expiry price and settle the paper trade.";
  }
}

function settleTrade(){
  if(!openTrade){alert("There is no open paper trade.");return;}
  if(Date.now()<openTrade.end){
    alert("Please wait until the full 1-minute expiry has passed.");return;
  }
  const finalPrice=Number(document.getElementById("expiryPrice").value);
  if(!Number.isFinite(finalPrice)||finalPrice<=0){
    alert("Enter a valid expiry price.");return;
  }

  let result,profit;
  if(finalPrice===openTrade.entry){
    result="DRAW";
    profit=openTrade.stake;
  }else if((openTrade.direction==="UP" && finalPrice>openTrade.entry) ||
           (openTrade.direction==="DOWN" && finalPrice<openTrade.entry)){
    result="WIN";
    profit=openTrade.stake*(1+openTrade.payout/100);
  }else{
    result="LOSS";
    profit=0;
  }

  const net=profit-openTrade.stake;
  balance+=profit;
  history.unshift({
    pair:openTrade.pair,
    direction:openTrade.direction,
    result,
    net:Number(net.toFixed(2))
  });
  openTrade=null;
  document.getElementById("timer").textContent="Trade completed";
  document.getElementById("tradeStatus").textContent=
    "Paper trade result: "+result+". Net result: "+net.toFixed(2)+" credits.";
  document.getElementById("expiryPrice").value="";
  save();
}

function save(){
  localStorage.setItem("paperBalance",String(balance));
  localStorage.setItem("paperHistory",JSON.stringify(history));
  render();
}

function render(){
  document.getElementById("balance").textContent=balance.toFixed(2);
  document.getElementById("history").innerHTML=history.slice(0,30).map(t=>
    `<tr><td>${t.pair}</td><td>${t.direction}</td><td>${t.result}</td><td>${t.net.toFixed(2)}</td></tr>`
  ).join("");
  const wins=history.filter(t=>t.result==="WIN").length;
  const losses=history.filter(t=>t.result==="LOSS").length;
  document.getElementById("stats").textContent=
    "Trades: "+history.length+" | Wins: "+wins+" | Losses: "+losses;
}

function resetDemo(){
  if(openTrade){alert("Settle or finish the open trade first.");return;}
  if(!confirm("Reset virtual balance and erase paper history?"))return;
  balance=10000;
  history=[];
  save();
  document.getElementById("tradeStatus").textContent="Demo reset. No open trade.";
  document.getElementById("timer").textContent="—";
}

render();
setMode("demo");
</script>
</body>
</html>
