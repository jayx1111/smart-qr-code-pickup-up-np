<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no" />
<meta name="theme-color" content="#0ea5a4" />
<meta name="apple-mobile-web-app-capable" content="yes" />
<meta name="apple-mobile-web-app-status-bar-style" content="default" />
<title>Smart QR Food Pickup — Mobile</title>
<style>
:root{--bg:#f7fafc;--card:#fff;--accent:#0ea5a4;--muted:#6b7280}
*{box-sizing:border-box}
body{font-family:Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial; background:var(--bg); color:#0f172a; margin:0; padding:22px}
.container{max-width:980px;margin:0 auto}
.header{display:flex;gap:12px;align-items:center}
.logo{width:56px;height:56px;border-radius:10px;background:linear-gradient(135deg,var(--accent),#0369a1);display:flex;align-items:center;justify-content:center;color:white;font-weight:700;font-size:18px}
.h1{margin:0}
.card{background:var(--card);border-radius:12px;padding:16px;margin-top:16px;box-shadow:0 6px 18px rgba(15,23,42,0.06)}
.row{display:flex;gap:12px}
.col{flex:1}
.controls{display:flex;gap:8px;flex-wrap:wrap}
.btn{background:var(--accent);color:white;padding:8px 12px;border-radius:8px;text-decoration:none;cursor:pointer;border:none}
.small{font-size:0.9rem;color:var(--muted)}
.menu-list{margin-top:10px}
.menu-item{display:flex;justify-content:space-between;padding:10px 0;border-bottom:1px dashed #eee}
.item-left{display:flex;gap:12px;align-items:center}
.qty{width:36px;padding:6px;border-radius:6px;border:1px solid #e6eef0;text-align:center}
.badge{display:inline-block;padding:4px 8px;border-radius:999px;background:#fde68a}
.kv{display:flex;gap:8px;align-items:center}
.footer{font-size:0.85rem;color:var(--muted);margin-top:12px}
.input,select{padding:8px;border-radius:8px;border:1px solid #e6eef0}
.checkout{display:flex;gap:12px;align-items:center;justify-content:space-between;margin-top:12px}
.lockers{display:grid;grid-template-columns:repeat(6,1fr);gap:8px;margin-top:12px}
.locker{padding:8px;border-radius:8px;background:#f8fafc;border:1px solid #e6eef0;text-align:center}
.qr-box{display:flex;gap:12px;align-items:center}
.code{font-weight:700;letter-spacing:1px}
.note{font-size:0.85rem;color:#444}
.toast{position:fixed;right:18px;bottom:18px;background:#111;color:white;padding:10px 14px;border-radius:10px;display:none}
@media(max-width:720px){.row{flex-direction:column}.container{padding:6px}.card{padding:12px}.btn{width:100%}}
</style>
</head>
<body>
<div class="container">
  <div class="header">
    <div class="logo">QR</div>
    <div>
      <h1 class="h1">Smart QR Food Pickup — Prototype</h1>
      <div class="small">Scan QR at stalls → view live menu & queue → pre-order & pay → collect at smart pickup.</div>
    </div>
  </div>

  <div class="card">
    <div class="row">
      <div class="col">
        <strong>Live menu & crowd</strong>
        <div class="small">Menu items below are sample data. "Crowd level" updates simulate students placing orders.</div>

        <div style="margin-top:12px">
          <div class="kv"><div class="small">Selected Stall</div><select id="stallSelect" class="input" style="margin-left:8px"><option value="Canteen A">Canteen A - Rice & Noodles</option><option value="Canteen B">Canteen B - Western & Drinks</option></select></div>
        </div>

        <div id="crowdBox" class="card" style="margin-top:10px;padding:12px;background:#f8fbfb">
          <div style="display:flex;justify-content:space-between;align-items:center">
            <div><strong>Estimated queue</strong><div id="queueLen" class="small">0 people</div></div>
            <div><strong>Avg prep time</strong><div id="prepTime" class="small">5 min</div></div>
          </div>
        </div>

        <div id="menu" class="menu-list"></div>
      </div>

      <div class="col" style="max-width:420px">
        <strong>Your cart & pickup</strong>
        <div class="small">Add items, choose pickup time, pay mock, and get a pickup code for the smart locker.</div>

        <div id="cart" class="card" style="margin-top:10px;padding:12px">
          <div id="cartItems" class="small">Cart is empty.</div>
          <div style="margin-top:10px">
            <label class="small">Pickup in (minutes)</label>
            <select id="pickupTime" class="input" style="width:100%;margin-top:6px">
              <option value="5">ASAP (~5)</option>
              <option value="10">10</option>
              <option value="15">15</option>
              <option value="20">20</option>
            </select>
          </div>

          <div class="checkout">
            <div>
              <div class="small">Total:</div>
              <div style="font-size:1.15rem;font-weight:700" id="total">S$0.00</div>
            </div>
            <div>
              <button class="btn" id="payBtn">Pay & Generate Pickup</button>
            </div>
          </div>

          <div id="pickupInfo" style="margin-top:12px;display:none">
            <div class="small">Pickup code (show to locker or staff):</div>
            <div class="code" id="pickupCode">--</div>
            <div style="margin-top:8px"><button class="btn" id="openLockerBtn">Open Locker (simulate)</button></div>
          </div>
        </div>

        <div class="card" style="margin-top:12px">
          <strong>Smart lockers (demo)</strong>
          <div class="small">Enter code to open the correct locker. This demo simulates 6 lockers.</div>
          <div class="lockers" id="lockers"></div>
          <div style="margin-top:8px"><input id="lockerInput" class="input" placeholder="Enter pickup code" /><button class="btn" id="lockerOpenManual" style="margin-left:8px">Open</button></div>
        </div>

        <div class="card" style="margin-top:12px">
          <strong>Phone access (QR / Link)</strong>
          <div class="small">Host this file online and students can scan a QR code or tap a link on their phone. Works on iOS & Android browsers.</div>
          <div style="margin-top:10px">
            <div class="small">Example link (replace with your real hosted URL):</div>
            <input class="input" style="width:100%" value="https://your-school.github.io/smart-pickup" readonly />
          </div>
          <div class="qr-box" style="margin-top:10px">
            <button class="btn" id="downloadBtn">Download HTML</button>
            <button class="btn" id="hostGuideBtn">Hosting guide</button>
          </div>
          <div class="note" style="margin-top:8px">Tip: After hosting, generate a QR code for the link and paste it at stalls, canteen entrances, or student portals.</div>
        </div>
          <div style="margin-top:8px" class="qr-box">
            <button class="btn" id="downloadBtn">Download HTML</button>
            <button class="btn" id="hostGuideBtn">Hosting guide</button>
          </div>
        </div>

      </div>
    </div>

    <div class="footer">Notes: This is a client-only prototype — in production you'd connect to a backend for real-time menus, payments (Stripe/PayNow), and locker integration (IoT API).</div>
  </div>
</div>

<div id="toast" class="toast"></div>

<script>
// Sample data (would be provided by backend in production)
const stalls = {
  "Canteen A": [
    {id:1,name:"Chicken Rice",price:3.5,eta:5},
    {id:2,name:"Veg Noodles",price:3.0,eta:6},
    {id:3,name:"Fruit Bowl",price:2.5,eta:3},
    {id:4,name:"Iced Lemon Tea",price:1.2,eta:2}
  ],
  "Canteen B": [
    {id:11,name:"Beef Burger",price:4.2,eta:8},
    {id:12,name:"Fish & Chips",price:4.8,eta:9},
    {id:13,name:"Salad Bowl",price:3.0,eta:4},
    {id:14,name:"Coffee",price:1.6,eta:2}
  ]
}

let state = {
  stall: document.getElementById('stallSelect').value,
  menu: [],
  cart: {},
  queueLen: Math.floor(Math.random()*6),
}

// Helpers
function showToast(msg, t=2200){
  const el = document.getElementById('toast');
  el.textContent = msg; el.style.display='block';
  setTimeout(()=>el.style.display='none', t);
}

function formatS(n){ return 'S$' + n.toFixed(2) }

function renderMenu(){
  const menuDiv = document.getElementById('menu');
  menuDiv.innerHTML = '';
  const list = stalls[state.stall] || [];
  list.forEach(it=>{
    const div = document.createElement('div'); div.className='menu-item';
    div.innerHTML = `
      <div class="item-left"><div><strong>${it.name}</strong><div class="small">est ${it.eta} min</div></div></div>
      <div style="display:flex;gap:8px;align-items:center">
        <div class="small">${formatS(it.price)}</div>
        <input class="qty" data-id="${it.id}" value="0" type="number" min="0" />
        <button class="btn small" data-add="${it.id}">Add</button>
      </div>`
    menuDiv.appendChild(div);
  })
}

function updateCrowdUI(){
  document.getElementById('queueLen').textContent = state.queueLen + ' people';
  // dynamic prep time estimate
  const avg = Math.max(3, 5 + Math.round(state.queueLen/2));
  document.getElementById('prepTime').textContent = avg + ' min';
}

function renderCart(){
  const el = document.getElementById('cartItems');
  const keys = Object.keys(state.cart);
  if(keys.length===0){ el.innerHTML='Cart is empty.'; document.getElementById('total').textContent=formatS(0); return }
  let html = '<div>';
  let total=0;
  keys.forEach(k=>{
    const item = state.cart[k];
    html += `<div style="display:flex;justify-content:space-between;padding:6px 0"><div>${item.name} × ${item.qty}</div><div>${formatS(item.qty*item.price)}</div></div>`
    total += item.qty*item.price;
  })
  html += '</div>';
  el.innerHTML = html;
  document.getElementById('total').textContent = formatS(total);
}

// wire menu interactions
function wireMenu(){
  document.getElementById('menu').addEventListener('click', e=>{
    const add = e.target.closest('[data-add]');
    if(add){
      const id = parseInt(add.getAttribute('data-add'));
      const list = stalls[state.stall];
      const item = list.find(x=>x.id===id);
      if(!item) return;
      const q = state.cart[id]?.qty || 0;
      state.cart[id] = {id:item.id,name:item.name,price:item.price,qty:q+1};
      renderCart(); showToast('Added ' + item.name);
      state.queueLen += 1; updateCrowdUI();
    }
  })
}

// Simulate other students ordering randomly to change queue length (fake real-time)
setInterval(()=>{
  const change = Math.random() < 0.6 ? 1 : -1;
  state.queueLen = Math.max(0, state.queueLen + (change));
  updateCrowdUI();
}, 8000)

// Pay & generate pickup code
function generatePickupCode(){
  const code = 'PX-' + Math.random().toString(36).slice(2,7).toUpperCase();
  return code;
}

function onPay(){
  const keys = Object.keys(state.cart);
  if(keys.length===0){ showToast('Cart empty — add items first'); return }
  // mock payment success
  const pickup = generatePickupCode();
  localStorage.setItem('lastPickup', JSON.stringify({code:pickup,items:state.cart,stall:state.stall,pickupIn:document.getElementById('pickupTime').value,ts:Date.now()}));
  document.getElementById('pickupCode').textContent = pickup;
  document.getElementById('pickupInfo').style.display = 'block';
  showToast('Payment success — pickup code: ' + pickup, 3200);
  // assign a locker (demo: 6 lockers)
  const lockerIdx = (Math.abs(hashCode(pickup)) % 6) + 1;
  localStorage.setItem('assignedLocker', lockerIdx);
  renderLockers();
  // clear cart
  state.cart = {};
  renderCart();
}

function hashCode(s){
  let h=0; for(let i=0;i<s.length;i++){ h = ((h<<5)-h) + s.charCodeAt(i); h |= 0; } return h;
}

function renderLockers(){
  const wrap = document.getElementById('lockers'); wrap.innerHTML='';
  const assigned = parseInt(localStorage.getItem('assignedLocker')) || null;
  for(let i=1;i<=6;i++){
    const div = document.createElement('div'); div.className='locker';
    div.innerHTML = `<div style="font-weight:700">L${i}</div><div class="small">${i===assigned? 'Has order' : 'Empty'}</div>`;
    wrap.appendChild(div);
  }
}

// manual open
function openLockerWithCode(code){
  const stored = JSON.parse(localStorage.getItem('lastPickup') || 'null');
  if(!stored || stored.code !== code){ showToast('Invalid pickup code'); return }
  const locker = parseInt(localStorage.getItem('assignedLocker')) || 0;
  showToast('Locker L' + locker + ' opened — enjoy your meal!');
  // mark emptied
  localStorage.removeItem('lastPickup'); localStorage.removeItem('assignedLocker'); renderLockers(); document.getElementById('pickupInfo').style.display='none';
}

// Download HTML helper
function downloadHTML(){
  const html = `<!doctype html>\n` + document.documentElement.outerHTML;
  const blob = new Blob([html], {type:'text/html'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a'); a.href=url; a.download='smart_qr_food_proto.html'; a.click();
  URL.revokeObjectURL(url);
}

// hosting guide
function showHostingGuide(){
  const msg = `Hosting options:\n\n1) GitHub Pages (free): create a repo, commit this file as index.html, enable Pages.\n2) Netlify (free): drag-and-drop the HTML file into Sites -> New site from drag & drop.\n3) Vercel: import repo and deploy.\n\nOnce hosted, create a QR for the hosted https:// URL using any QR generator and print/stick at stalls.`;
  alert(msg);
}

// initial render
function init(){
  renderMenu(); wireMenu(); renderCart(); updateCrowdUI(); renderLockers();
  document.getElementById('stallSelect').addEventListener('change', e=>{ state.stall = e.target.value; renderMenu(); })
  document.getElementById('payBtn').addEventListener('click', onPay);
  document.getElementById('openLockerBtn').addEventListener('click', ()=>{
    const last = JSON.parse(localStorage.getItem('lastPickup') || 'null');
    if(!last){ showToast('No pickup found in this browser.'); return }
    openLockerWithCode(last.code);
  })
  document.getElementById('lockerOpenManual').addEventListener('click', ()=>{
    const code = document.getElementById('lockerInput').value.trim(); if(!code){ showToast('Enter a code'); return } openLockerWithCode(code);
  })
  document.getElementById('downloadBtn').addEventListener('click', downloadHTML);
  document.getElementById('hostGuideBtn').addEventListener('click', showHostingGuide);
}

init();
</script>
</body>
</html>
