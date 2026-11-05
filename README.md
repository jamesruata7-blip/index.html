
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Multi-Market Robot 4.0</title>
<style>
*{box-sizing:border-box}
body{margin:0;padding:15px;background:#101522;color:#f4f6fa;font-family:Arial,sans-serif}
main{max-width:750px;margin:auto}
h1{color:#e8c66a;font-size:24px}
.card{background:#1b2435;padding:15px;border-radius:12px;margin:14px 0}
label{display:block;margin:12px 0 6px}
input,textarea,select{width:100%;padding:12px;border-radius:8px;border:1px solid #46536b;background:#111a29;color:white;font-size:15px}
textarea{min-height:90px}
button{padding:12px;border:0;border-radius:8px;font-weight:bold;margin:5px 2px;cursor:pointer}
.primary{background:#e8c66a;color:#151515}
.up{background:#168c62;color:white}
.down{background:#bd4552;color:white}
.neutral{background:#46536b;color:white}
.value{font-size:23px;font-weight:bold}
.green{color:#48d6a0}
.red{color:#ff7782}
.yellow{color:#e8c66a}
.small{font-size:13px;color:#aeb9cb;line-height:1.5}
.row{display:flex;gap:10px;flex-wrap:wrap}
.row>div{flex:1;min-width:100px}
.hidden{display:none}
img{max-width:100%;border-radius:8px;margin-top:10px}
table{width:100%;border-collapse:collapse;font-size:12px}
td,th{padding:8px;border-bottom:1px solid #39445a;text-align:left}
</style>
</head>
<body>
<main>
<h1>Multi-Market Robot 4.0</h1>
<p class="small">Forex • Crypto • Stocks • Screenshot Analysis • RSI 14 • EMA 20 • Demo Trading</p>

<div class="card">
<h3>1. Trading mode</h3>
<button class="primary" onclick="setMode('demo')">DEMO MODE</button>
<button class="neutral" onclick="setMode('analysis')">ANALYSIS MODE</button>
<p id="modeText" class="green">DEMO MODE — virtual trades only</p>
</div>

<div class="card">
<h3>2. Select market</h3>
<label>Market category</label>
<select id="market" onchange="updateSymbols()">
<option value="forex">Forex</option>
<option value="crypto">Cryptocurrency</option>
<option value="stocks">Stocks</option>
<option value="currency">Currency exchange</option>
</select>
<label>Instrument</label>
<select id="symbol"></select>
<label>Custom symbol (optional)</label>
<input id="custom" placeholder="Example: EUR/SEK or BTC/USD">
<label>Timeframe</label>
<select id="timeframe">
<option>M1</option><option>M5</option><option>M15</option>
<option>M30</option><option>H1</option><option>H4</option><option>D1</option>
</select>
</div>

<div class="card">
<h3>3. Upload chart screenshot</h3>
<input type="file" id="imageFile" accept="image/*">
<img id="preview" class="hidden" alt="Chart screenshot">
<button class="primary" onclick="readImage()">READ SCREENSHOT</button>
<p id="ocrStatus" class="small">Upload a chart screenshot.</p>
<textarea id="ocrText" placeholder="Recognised text will appear here. Check all numbers yourself."></textarea>
<p class="small">Screenshot text recognition is experimental. It does not automatically calculate RSI or EMA from chart pixels.</p>
</div>

<div class="card">
<h3>4. Analyze market prices</h3>
<p class="small">Enter at least 21 completed closing prices, oldest first, separated by commas.</p>
<textarea id="prices" placeholder="Example: 1.08210, 1.08230, 1.08225, ..."></textarea>
<button class="primary" onclick="analyze()">ANALYZE PRICES</button>
<div class="row">
<div>Signal<div id="signal" class="value yellow">WAIT</div></div>
<div>RSI 14<div id="rsi" class="value">—</div></div>
<div>EMA 20<div id="ema" class="value">—</div></div>
</div>
<p>Latest close: <strong id="latest">—</strong></p>
<p id="reason" class="small">Enter prices to begin.</p>
<p id="analysisMeta" class="small"></p>
</div>

<div class="card">
<h3>5. Live data connection</h3>
<p class="yellow">NOT CONNECTED</p>
<p class="small">This version uses manually entered prices. Live market data requires an external provider and secure backend. Do not put private API keys in public GitHub code.</p>
</div>

<div class="card" id="demoPanel">
<h3>6. Demo paper trading</h3>
<p>Virtual balance: <strong id="balance" class="value green">10000.00</strong> credits</p>
<label>Virtual stake</label>
<input id="stake" type="number" value="100" min="1">
<label>Assumed payout (%)</label>
<input id="payout" type="number" value="80" min="1" max="100">
<div>
<button class="up" onclick="startTrade('UP')">START UP</button>
<button class="down" onclick="startTrade('DOWN')">START DOWN</button>
</div>
<p id="tradeStatus" class="yellow">No open trade.</p>
<p id="timer" class="value">—</p>
<label>Expiry price after one minute</label>
<input id="expiryPrice" type="number" step="any" placeholder="Enter final price">
<button class="primary" onclick="settleTrade()">SETTLE TRADE</button>
</div>

<div class="card">
<h3>7. Trading history</h3>
<p id="stats" class="small">Trades: 0 | Wins: 0 | Losses: 0</p>
<table>
<thead><tr><th>Instrument</th><th>Time</th><th>Direction</th><th>Result</th><th>Net</th></tr></thead>
<tbody id="history"></tbody>
</table>
<button class="neutral" onclick="resetDemo()">RESET DEMO</button>
</div>

<div class="card">
<h3>Important safety notice</h3>
<p class="small">This is an educational simulator only. It does not connect to Pocket Option or any broker. It cannot place real trades. Signals are examples, not guaranteed predictions. Always practise with virtual money.</p>
</div>
</main>

<script src="https://cdn.jsdelivr.net/npm/tesseract.js@5/dist/tesseract.min.js"></script>
<script>
const markets={
forex:["EUR/USD","GBP/USD","USD/JPY","USD/CHF","AUD/USD","NZD/USD","USD/CAD","EUR/GBP","EUR/JPY","EUR/CHF","GBP/JPY","GBP/CHF","AUD/JPY","CAD/JPY"],
crypto:["BTC/USD","ETH/USD","BNB/USD","SOL/USD","XRP/USD","ADA/USD","DOGE/USD","AVAX/USD","LINK/USD","DOT/USD","LTC/USD","TRX/USD"],
stocks:["AAPL","TSLA","NVDA","AMZN","MSFT","GOOGL","META","NFLX","AMD","INTC","ORCL","DIS","JPM","KO"],
currency:["EUR/SEK","USD/SEK","GBP/SEK","NOK/SEK","DKK/SEK","CHF/SEK","JPY/SEK","CAD/SEK","AUD/SEK","EUR/USD","GBP/USD"]
};

let mode="demo";
let lastPrices=[];
let openTrade=null;
let countdown=null;
let balance=Number(localStorage.getItem("paperBalance")||10000);
let history=[];
try{history=JSON.parse(localStorage.getItem("paperHistory")||"[]");if(!Array.isArray(history))history=[];}catch(e){history=[];}

function updateSymbols(){
 const list=markets[document.getElementById("market").value];
 const select=document.getElementById("symbol");
 select.innerHTML="";
 list.forEach(item=>{
  const option=document.createElement("option");
  option.value=item;
  option.textContent=item;
  select.appendChild(option);
 });
}
function getInstrument(){
 return document.getElementById("custom").value.trim()||document.getElementById("symbol").value;
}

document.getElementById("imageFile").addEventListener("change",function(){
 const file=this.files[0];
 if(!file)return;
 if(!file.type.startsWith("image/")){alert("Please choose an image.");return;}
 const reader=new FileReader();
 reader.onload=e=>{
  const img=document.getElementById("preview");
  img.src=e.target.result;
  img.classList.remove("hidden");
  document.getElementById("ocrStatus").textContent="Screenshot loaded.";
 };
 reader.readAsDataURL(file);
});

async function readImage(){
 const file=document.getElementById("imageFile").files[0];
 if(!file){alert("Upload a screenshot first.");return;}
 if(!window.Tesseract){alert("Text recognition could not load. Check your internet connection.");return;}
 const status=document.getElementById("ocrStatus");
 status.textContent="Reading screenshot. Please wait...";
 try{
  const result=await Tesseract.recognize(file,"eng");
  document.getElementById("ocrText").value=result.data.text;
  status.textContent="Finished. Please check the recognised text.";
 }catch(e){
  status.textContent="Could not read this image. Try another screenshot.";
 }
}

function setMode(next){
 mode=next;
 document.getElementById("modeText").textContent=next==="demo"?"DEMO MODE — virtual trades only":"ANALYSIS MODE — signals only";
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
 let avgGain=gains/period,avgLoss=losses/period;
 for(let i=period+1;i<values.length;i++){
  const change=values[i]-values[i-1];
  avgGain=(avgGain*(period-1)+Math.max(change,0))/period;
  avgLoss=(avgLoss*(period-1)+Math.max(-change,0))/period;
 }
 if(avgLoss===0)return 100;
 if(avgGain===0)return 0;
 return 100-(100/(1+avgGain/avgLoss));
}

function analyze(){
 const raw=document.getElementById("prices").value.trim();
 const values=raw.split(/[,\s;]+/).filter(Boolean).map(Number);
 if(values.length<21||values.some(v=>!Number.isFinite(v)||v<=0)){
  alert("Enter at least 21 valid positive closing prices.");
  return;
 }
 lastPrices=values;
 const currentRSI=rsi(values);
 const previousRSI=rsi(values.slice(0,-1));
 const currentEMA=ema(values,20);
 const previousEMA=ema(values.slice(0,-1),20);
 const latest=values[values.length-1];
 let signal="WAIT";
 let reason="Conditions are mixed. Waiting is also a decision.";
 if(latest>currentEMA&&currentEMA>previousEMA&&currentRSI>50&&currentRSI<70&&currentRSI>previousRSI){
  signal="BUY";
  reason="Possible bullish setup: price above rising EMA20 and RSI above 50 and rising.";
 }else if(latest<currentEMA&&currentEMA<previousEMA&&currentRSI<50&&currentRSI>30&&currentRSI<previousRSI){
  signal="SELL";
  reason="Possible bearish setup: price below falling EMA20 and RSI below 50 and falling.";
 }
 const el=document.getElementById("signal");
 el.textContent=signal;
 el.className="value "+(signal==="BUY"?"green":signal==="SELL"?"red":"yellow");
 document.getElementById("rsi").textContent=currentRSI.toFixed(2);
 document.getElementById("ema").textContent=currentEMA.toPrecision(8);
 document.getElementById("latest").textContent=latest;
 document.getElementById("reason").textContent=reason;
 document.getElementById("analysisMeta").textContent=getInstrument()+" | "+document.getElementById("timeframe").value+" | "+values.length+" manual closing prices";
}

function startTrade(direction){
 if(mode!=="demo"){alert("Switch to DEMO mode first.");return;}
 if(openTrade){alert("You already have an open paper trade.");return;}
 if(lastPrices.length<21){alert("Analyze at least 21 prices first.");return;}
 const stake=Number(document.getElementById("stake").value);
 const payout=Number(document.getElementById("payout").value);
 if(!Number.isFinite(stake)||stake<=0||stake>balance){alert("Check your virtual stake.");return;}
 if(!Number.isFinite(payout)||payout<=0||payout>100){alert("Payout must be between 1 and 100.");return;}
 openTrade={
  direction:direction,
  stake:stake,
  payout:payout,
  entry:lastPrices[lastPrices.length-1],
  instrument:getInstrument(),
  timeframe:document.getElementById("timeframe").value,
  end:Date.now()+60000
 };
 balance-=stake;
 save();
 document.getElementById("tradeStatus").textContent=direction+" paper trade opened. Wait one minute.";
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
  document.getElementById("tradeStatus").textContent="Expiry reached. Enter the final price.";
 }
}

function settleTrade(){
 if(!openTrade){alert("No open paper trade.");return;}
 if(Date.now()<openTrade.end){alert("Wait until the one-minute expiry.");return;}
 const finalPrice=Number(document.getElementById("expiryPrice").value);
 if(!Number.isFinite(finalPrice)||finalPrice<=0){alert("Enter a valid expiry price.");return;}
 let result,returned;
 if(finalPrice===openTrade.entry){
  result="DRAW";
  returned=openTrade.stake;
 }else if((openTrade.direction==="UP"&&finalPrice>openTrade.entry)||(openTrade.direction==="DOWN"&&finalPrice<openTrade.entry)){
  result="WIN";
  returned=openTrade.stake*(1+openTrade.payout/100);
 }else{
  result="LOSS";
  returned=0;
 }
 const net=returned-openTrade.stake;
 balance+=returned;
 history.unshift({
  instrument:openTrade.instrument,
  timeframe:openTrade.timeframe,
  direction:openTrade.direction,
  result:result,
  net:Number(net.toFixed(2))
 });
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

function escapeHTML(value){
 return String(value).replace(/[&<>"']/g,c=>({
  "&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#39;"
 }[c]));
}

function render(){
 document.getElementById("balance").textContent=balance.toFixed(2);
 document.getElementById("history").innerHTML=history.slice(0,30).map(t=>
  "<tr><td>"+escapeHTML(t.instrument||"")+"</td><td>"+escapeHTML(t.timeframe||"")+"</td><td>"+escapeHTML(t.direction||"")+"</td><td>"+escapeHTML(t.result||"")+"</td><td>"+Number(t.net||0).toFixed(2)+"</td></tr>"
 ).join("");
 const wins=history.filter(t=>t.result==="WIN").length;
 const losses=history.filter(t=>t.result==="LOSS").length;
 document.getElementById("stats").textContent="Trades: "+history.length+" | Wins: "+wins+" | Losses: "+losses;
}

function resetDemo(){
 if(openTrade){alert("Finish the open trade first.");return;}
 if(!confirm("Reset virtual balance and history?"))return;
 balance=10000;
 history=[];
 save();
 document.getElementById("tradeStatus").textContent="Demo reset.";
 document.getElementById("timer").textContent="—";
}

updateSymbols();
render();
setMode("demo");
</script>
</body>
</html>
