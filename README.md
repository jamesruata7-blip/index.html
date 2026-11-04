
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Forex Screenshot & Paper Robot 3.0</title>
<style>
*{box-sizing:border-box}
body{margin:0;padding:16px;background:#101522;color:#f4f6fa;font-family:Arial,sans-serif}
.container{max-width:700px;margin:auto}
h1{color:#e8c66a;font-size:24px}
.card{background:#1b2435;padding:16px;border-radius:13px;margin:14px 0}
label{display:block;margin:12px 0 6px}
input,textarea,select{width:100%;padding:12px;border-radius:8px;border:1px solid #46536b;background:#111a29;color:white;font-size:15px}
textarea{min-height:90px}
button{padding:12px;border:0;border-radius:8px;font-weight:bold;margin:5px 2px;cursor:pointer}
.primary{background:#e8c66a;color:#151515}
.up{background:#168c62;color:white}
.down{background:#bd4552;color:white}
.neutral{background:#46536b;color:white}
.value{font-size:24px;font-weight:bold}
.green{color:#48d6a0}
.red{color:#ff7782}
.yellow{color:#e8c66a}
.small{font-size:13px;color:#aeb9cb;line-height:1.5}
.row{display:flex;gap:10px;flex-wrap:wrap}
.row>*{flex:1}
img{max-width:100%;border-radius:9px;margin-top:10px}
.hidden{display:none}
table{width:100%;border-collapse:collapse;font-size:13px}
td,th{padding:8px;border-bottom:1px solid #39445a;text-align:left}
</style>
</head>
<body>
<div class="container">
<h1>Forex Screenshot & Paper Robot 3.0</h1>
<p class="small">Screenshot analysis | RSI 14 | EMA 20 | Demo-only trading</p>

<div class="card">
<h3>Trading mode</h3>
<div class="row">
<button class="primary" onclick="setMode('demo')">DEMO MODE</button>
<button class="neutral" onclick="setMode('analysis')">ANALYSIS MODE</button>
</div>
<p id="modeText" class="green">DEMO MODE — virtual trades only</p>
</div>

<div class="card">
<h3>1. Screenshot analysis</h3>
<label>Upload your Pocket Option chart screenshot</label>
<input type="file" id="imageFile" accept="image/*">
<img id="preview" class="hidden" alt="Chart screenshot preview">
<button class="primary" onclick="readImage()">READ SCREENSHOT TEXT</button>
<p id="ocrStatus" class="small">Upload a chart image to begin.</p>
<label>Recognised text — check it yourself</label>
<textarea id="ocrText" placeholder="Recognised text will appear here. OCR may make mistakes."></textarea>
<p class="small">OCR is experimental. It cannot reliably identify candle closes, RSI or EMA from every chart. Confirm values against Pocket Option.</p>
</div>

<div class="card">
<h3>2. Market prices</h3>
<label>Currency pair</label>
<select id="pair">
<option>EUR/USD</option>
<option>GBP/USD</option>
<option>USD/JPY</option>
<option>USD/CHF</option>
<option>AUD/USD</option>
</select>
<label>Timeframe</label>
<select id="timeframe">
<option>M1 — 1 minute</option>
<option>M5 — 5 minutes</option>
<option>M15 — 15 minutes</option>
<option>H1 — 1 hour</option>
</select>
<label>Closing prices — oldest first</label>
<textarea id="prices" placeholder="Paste at least 21 completed candle closing prices, separated by commas"></textarea>
<button class="primary" onclick="analyze()">ANALYZE PRICES</button>
<p class="small">Use completed candles from the same pair, timeframe and price source.</p>
</div>

<div class="card">
<h3>3. Robot analysis</h3>
<div class="row">
<div>Signal<div id="signal" class="value yellow">WAIT</div></div>
<div>RSI 14<div id="rsi" class="value">—</div></div>
<div>EMA 20<div id="ema" class="value">—</div></div>
</div>
<p>Latest close: <strong id="latest">—</strong></p>
<p id="reason" class="small">Upload a screenshot or enter prices to start.</p>
</div>

<div class="card">
<h3>4. Live market data</h3>
<p class="yellow" id="liveStatus">NOT CONNECTED</p>
<p class="small">Live EUR/USD data requires an external provider and a secure backend connection. This GitHub Pages version does not fetch live quotes yet. Never paste a private API key into this public file.</p>
<button class="neutral" onclick="alert('Live data will be connected in the next development stage.')">LIVE DATA SETUP</button>
</div>

<div class="card" id="demoPanel">
<h3>5. Demo paper trading</h3>
<p>Virtual balance: <strong id="balance" class="value green">10,000.00</strong> credits</p>
<label>Virtual stake</label>
<input id="stake" type="number" min="1" value="100">
<label>Assumed payout (%)</label>
<input id="payout" type="number" min="1" max="100" value="80">
<p class="small">This is a simulation assumption, not a real broker payout.</p>
<div class="row">
<button class="up" onclick="startTrade('UP')">START UP</button>
<button class="down" onclick="startTrade('DOWN')">START DOWN</button>
</div>
<p id="tradeStatus" class="yellow">No open trade.</p>
<p id="timer" class="value">—</p>
<label>Expiry price after one minute</label>
<input id="expiryPrice" type="number" step="any" placeholder="Enter final market price">
<button class="primary" onclick="settleTrade()">SETTLE PAPER TRADE</button>
</div>

<div class="card">
<h3>6. Paper trading history</h3>
<p id="stats" class="small">Trades: 0 | Wins: 0 | Losses: 0</p>
<table>
<thead><tr><th>Pair</th><th>Direction</th><th>Result</th><th>Net P/L</th></tr></thead>
<tbody id="history"></tbody>
</table>
<button class="neutral" onclick="resetDemo()">RESET DEMO</button>
</div>

<div class="card">
<h3>Safety notice</h3>
<p class="small">Educational simulator only. No broker connection, live orders or real-money execution. Signals are not guarantees. Always verify screenshot readings and market prices.</p>
</div>
</div>

<script src="https://cdn.jsdelivr.net/npm/tesseract.js@5/dist/tesseract.min.js"></script>
<script>
let mode="demo",lastPrices=[],openTrade=null,countdown=null;
let balance=Number(localStorage.getItem("paperBalance")||10000);
let history=JSON.parse(localStorage.getItem("paperHistory")||"[]");

document.getElementById("imageFile").addEventListener("change",function(){
 const file=this.files[0];
 if(!file)return;
 if(!file.type.startsWith("image/")){alert("Please select an image.");return;}
 const reader=new FileReader();
 reader.onload=e=>{
  const img=document.getElementById("preview");
  img.src=e.target.result;
  img.classList.remove("hidden");
  document.getElementById("ocrStatus").textContent="Screenshot loaded. You can try reading its text.";
 };
 reader.readAsDataURL(file);
});

async function readImage(){
 const file=document.getElementById("imageFile").files[0];
 if(!file){alert("Upload a screenshot first.");return;}
 if(!window.Tesseract){
  alert("Text recognition library could not load. Check your internet connection.");
  return;
 }
 const status=document.getElementById("ocrStatus");
 status.textContent="Reading image. Please wait...";
 try{
  const result=await Tesseract.recognize(file,"eng");
  document.getElementById("ocrText").value=result.data.text;
  status.textContent="Reading finished. Check every number carefully.";
 }catch(e){
  status.textContent="Could not read the image. Try another screenshot.";
 }
}

function setMode(next){
 mode=next;
 document.getElementById("modeText").textContent=
 next==="demo"?"DEMO MODE — virtual trades only":"ANALYSIS MODE — signals only";
 document.getElementById("demoPanel").classList.toggle("hidden",next!=="demo");
}

function ema(values,period){
 if(values.length<period)return null;
 let result=values.slice(0,period).reduce((a,b)=>a+b,0)/period;
 const multiplier=2/(period+1);
 for(let i=period;i<values.length;i++)
 result=(values[i]-result)*multiplier+result;
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
 let avgGain=gains/period,avgLoss=losses/period;
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
 if(values.length<21||values.some(v=>!Number.isFinite(v)||v<=0)){
  alert("Enter at least 21 valid positive closing prices.");return;
 }
 lastPrices=values;
 const currentRSI=rsi(values);
 const previousRSI=rsi(values.slice(0,-1));
 const currentEMA=ema(values,20);
 const latest=values[values.length-1];
 let signal="WAIT";
 let reason="No confirmed setup. Waiting is also a decision.";
 if(latest>currentEMA&&currentRSI>=30&&currentRSI<=55&&currentRSI>previousRSI){
  signal="BUY";
  reason="Price above EMA 20; RSI is in the selected range and rising.";
 }else if(latest<currentEMA&&currentRSI>=45&&currentRSI<=70&&currentRSI<previousRSI){
  signal="SELL";
  reason="Price below EMA 20; RSI is in the selected range and falling.";
 }
 const el=document.getElementById("signal");
 el.textContent=signal;
 el.className="value "+(signal==="BUY"?"green":signal==="SELL"?"red":"yellow");
 document.getElementById("rsi").textContent=currentRSI.toFixed(2);
 document.getElementById("ema").textContent=currentEMA.toFixed(6);
 document.getElementById("latest").textContent=latest;
 document.getElementById("reason").textContent=reason;
}

function startTrade(direction){
 if(mode!=="demo"){alert("Switch to DEMO mode first.");return;}
 if(openTrade){alert("You already have an open paper trade.");return;}
 if(lastPrices.length<21){alert("Analyze at least 21 prices first.");return;}
 const stake=Number(document.getElementById("stake").value);
 const payout=Number(document.getElementById("payout").value);
 if(!Number.isFinite(stake)||stake<=0||stake>balance){alert("Check your virtual stake.");return;}
 if(!Number.isFinite(payout)||payout<=0||payout>100){alert("Payout must be between 1 and 100.");return;}
 openTrade={direction,stake,payout,entry:lastPrices[lastPrices.length-1],
 pair:document.getElementById("pair").value,end:Date.now()+60000};
 balance-=stake;
 save();
 document.getElementById("tradeStatus").textContent=
 direction+" opened at "+openTrade.entry+". Wait one minute.";
 tick();
 countdown=setInterval(tick,250);
}

function tick(){
 if(!openTrade)return;
 const seconds=Math.max(0,Math.ceil((openTrade.end-Date.now())/1000));
 document.getElementById("timer").textContent=seconds+" seconds remaining";
 if(seconds===0){
  clearInterval(countdown);countdown=null;
  document.getElementById("tradeStatus").textContent="Expiry reached. Enter expiry price.";
 }
}

function settleTrade(){
 if(!openTrade){alert("No open paper trade.");return;}
 if(Date.now()<openTrade.end){alert("Wait until the one-minute expiry.");return;}
 const finalPrice=Number(document.getElementById("expiryPrice").value);
 if(!Number.isFinite(finalPrice)||finalPrice<=0){alert("Enter a valid expiry price.");return;}
 let result,profit;
 if(finalPrice===openTrade.entry){result="DRAW";profit=openTrade.stake;}
 else if((openTrade.direction==="UP"&&finalPrice>openTrade.entry)||
 (openTrade.direction==="DOWN"&&finalPrice<openTrade.entry)){
  result="WIN";profit=openTrade.stake*(1+openTrade.payout/100);
 }else{result="LOSS";profit=0;}
 const net=profit-openTrade.stake;
 balance+=profit;
 history.unshift({pair:openTrade.pair,direction:openTrade.direction,result,net:Number(net.toFixed(2))});
 openTrade=null;
 document.getElementById("timer").textContent="Trade completed";
 document.getElementById("tradeStatus").textContent="Result: "+result+" | Net: "+net.toFixed(2)+" credits";
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
 `<tr><td>${t.pair}</td><td>${t.direction}</td><td>${t.result}</td><td>${Number(t.net).toFixed(2)}</td></tr>`).join("");
 const wins=history.filter(t=>t.result==="WIN").length;
 const losses=history.filter(t=>t.result==="LOSS").length;
 document.getElementById("stats").textContent=
 "Trades: "+history.length+" | Wins: "+wins+" | Losses: "+losses;
}
function resetDemo(){
 if(openTrade){alert("Finish the open trade first.");return;}
 if(!confirm("Reset virtual balance and history?"))return;
 balance=10000;history=[];save();
 document.getElementById("tradeStatus").textContent="Demo reset.";
 document.getElementById("timer").textContent="—";
}
render();
setMode("demo");
</script>
</body>
</html>
