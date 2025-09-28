<!doctype html>
<html lang="th">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>เครื่องวิเคราะห์ออดส์ (ต้นแบบ)</title>
<style>
  body{font-family:system-ui,-apple-system,Segoe UI,Roboto,Helvetica,Arial; padding:12px; background:#f6f8fb; color:#111}
  .box{background:#fff;border-radius:8px;padding:12px;margin-bottom:12px;box-shadow:0 1px 4px rgba(16,24,40,0.06)}
  header{display:flex;gap:12px;align-items:center;justify-content:space-between}
  h1{font-size:18px;margin:0}
  label{font-size:13px;color:#333}
  input[type=text],select{width:100%;padding:8px;border-radius:6px;border:1px solid #ddd;margin-top:6px}
  button{background:#2563eb;color:#fff;padding:8px 12px;border-radius:8px;border:none;cursor:pointer}
  table{width:100%;border-collapse:collapse;margin-top:12px}
  th,td{padding:8px;border-bottom:1px solid #eee;text-align:left;font-size:13px}
  th{background:#f3f6fb}
  .badge{display:inline-block;padding:4px 8px;border-radius:999px;font-size:12px}
  .green{background:#d1fae5;color:#064e3b}
  .yellow{background:#fff7cc;color:#7a5b00}
  .red{background:#ffe4e6;color:#7f1d1d}
  .muted{color:#666;font-size:13px}
  .small{font-size:12px;color:#666}
  .row{display:flex;gap:8px}
  @media(min-width:720px){ .row>div{flex:1} }
</style>
</head>
<body>
  <div class="box">
    <header>
      <h1>เครื่องวิเคราะห์ออดส์ — ต้นแบบ</h1>
      <div class="small muted">ดึงราคาจาก TheOddsAPI — ใส่ API Key ด้านล่าง</div>
    </header>

    <div style="margin-top:8px">
      <label>API Key (5039b764bc1ec7fc640762765afeb7f0)</label>
      <input id="apiKey" type="text" placeholder="วาง API Key ที่นี่ (จะไม่ถูกส่งให้ใคร)" />
    </div>

    <div style="margin-top:8px" class="row">
      <div>
        <label>เลือกลีก</label>
        <select id="league">
          <option value="soccer_epl">อังกฤษ — พรีเมียร์ลีก</option>
          <option value="soccer_spain_la_liga">สเปน — ลา ลีกา</option>
          <option value="soccer_italy_serie_a">อิตาลี — เซเรีย อา</option>
          <option value="soccer_germany_bundesliga">เยอรมัน — บุนเดสลีกา</option>
          <option value="soccer_france_ligue_one">ฝรั่งเศส — ลีก 1</option>
          <option value="soccer_england_championship">อังกฤษ — แชมเปียนชิพ</option>
          <option value="soccer_spain_segunda">สเปน — เซกุนด้า</option>
          <option value="soccer_italy_serie_b">อิตาลี — เซเรีย บี</option>
          <option value="soccer_germany_bundesliga2">เยอรมัน — 2. บุนเดสลีกา</option>
          <option value="soccer_france_ligue_2">ฝรั่งเศส — ลีก 2</option>
          <option value="soccer_japan_j_league">ญี่ปุ่น — เจลีก</option>
          <option value="soccer_korea_kleague1">เกาหลี — เค ลีก 1</option>
          <option value="soccer_china_csl">จีน — CSL</option>
          <option value="soccer_thailand">ไทย — ไทยลีก</option>
          <option value="soccer_asia_afc_champions_league">เอเชีย — ACL</option>
        </select>
      </div>
      <div>
        <label>ภูมิภาคผู้ให้ราคา (regions)</label>
        <select id="regions">
          <option value="uk">UK</option>
          <option value="eu">EU</option>
          <option value="us">US</option>
          <option value="au">AU</option>
        </select>
      </div>
    </div>

    <div style="margin-top:8px" class="row">
      <div style="flex:0 0 140px">
        <label>ดึงข้อมูล</label><br/>
        <button id="fetchBtn">ดึง & วิเคราะห์</button>
      </div>
      <div style="flex:1">
        <label>สถานะ</label>
        <div id="status" class="small muted">ยังไม่ได้ดึงข้อมูล</div>
      </div>
    </div>
    <div class="small muted" style="margin-top:8px">หมายเหตุ: ใส่คีย์แล้วดึงได้ทั้งก่อนแข่งและแบบสด (ถ้า API ให้ข้อมูลสด)</div>
  </div>

  <div id="results" class="box" style="display:none">
    <div style="display:flex;justify-content:space-between;align-items:center">
      <div><strong>ผลลัพธ์</strong> <span class="small muted" id="summaryCount"></span></div>
      <div class="small muted">ขอเวลา 1–5 วินาที / หน้าจอ</div>
    </div>
    <table id="tbl">
      <thead>
        <tr><th>เวลา</th><th>คู่</th><th>Fav</th><th>Model %</th><th>Market %</th><th>Edge</th><th>แฮนดิแคป</th><th>คำแนะนำ</th></tr>
      </thead>
      <tbody></tbody>
    </table>
  </div>

<script>
const BASE = "https://api.the-odds-api.com/v4/sports";
function implied(dec){ return dec ? 1.0/dec : 0; }
function normalize(arr){
  const s = arr.reduce((a,b)=>a+b,0);
  return s ? arr.map(x=>x/s) : arr;
}
function recommendHandicap(modelHome, modelAway){
  const diff = modelHome - modelAway;
  const ad = Math.abs(diff);
  if(ad<0.03) return "PK / เลี่ยง";
  if(diff>0){
    if(ad<0.10) return "เจ้าบ้าน -0.25";
    if(ad<0.20) return "เจ้าบ้าน -0.5";
    if(ad<0.30) return "เจ้าบ้าน -0.75";
    return "เจ้าบ้าน -1.0+";
  } else {
    if(ad<0.10) return "ทีมเยือน +0.25";
    if(ad<0.20) return "ทีมเยือน +0.5";
    if(ad<0.30) return "ทีมเยือน +0.75";
    return "ทีมเยือน +1.0+";
  }
}
function adviceFromEdge(edge){
  if(edge === null) return "ไม่มีราคา";
  if(edge >= 0.05) return {txt:"เล่นทีมต่อ", cls:"green"};
  if(edge > 0) return {txt:"มีมุมเล็กๆ", cls:"yellow"};
  if(edge <= 0) return {txt:"เล่นทีมรอง", cls:"red"};
  return {txt:"เลี่ยง", cls:"yellow"};
}

document.getElementById("fetchBtn").addEventListener("click", async ()=>{
  const key = document.getElementById("apiKey").value.trim();
  if(!key){ alert("กรุณาใส่ API Key ก่อน"); return; }
  const league = document.getElementById("league").value;
  const regions = document.getElementById("regions").value;
  document.getElementById("status").innerText = "กำลังดึงข้อมูล...";
  document.getElementById("results").style.display = "none";
  const url = `${BASE}/${league}/odds?regions=${regions}&markets=h2h,spreads,totals&oddsFormat=decimal&apiKey=${encodeURIComponent(key)}`;

  try {
    const r = await fetch(url);
    if(!r.ok){ const t = await r.text(); throw new Error("API error: "+r.status+" "+t); }
    const data = await r.json();
    document.getElementById("status").innerText = `พบ ${data.length} เหตุการณ์ — กำลังวิเคราะห์...`;
    const rows = [];
    for(const e of data){
      const homeName = e.home_team, awayName = e.away_team;
      const h2hList = [];
      for(const b of e.bookmakers || []){
        for(const m of b.markets || []){
          if(m.key === "h2h"){
            let pHome=null, pAway=null, pDraw=null;
            for(const o of m.outcomes || []){
              const n = (o.name||"").toLowerCase();
              if(n === (homeName||"").toLowerCase()) pHome = o.price;
              if(n === (awayName||"").toLowerCase()) pAway = o.price;
              if(n === "draw"||n==="tie"||n==="x") pDraw = o.price;
            }
            if(pHome && pAway){
              h2hList.push({h:pHome,d:pDraw||null,a:pAway});
            }
          }
        }
      }
      if(h2hList.length===0) continue;
      let sumH=0,sumD=0,sumA=0;
      for(const it of h2hList){
        sumH += implied(it.h);
        sumD += implied(it.d||Infinity)*0;
        sumA += implied(it.a);
      }
      let avgH = sumH / h2hList.length;
      let avgD = 0;
      let avgA = sumA / h2hList.length;
      const [mH,mD,mA] = normalize([avgH,avgD,avgA]);
      const favSide = (mH>mA) ? "เจ้าบ้าน" : "ทีมเยือน";
      let marketFavImplied = null;
      const firstB = e.bookmakers && e.bookmakers[0];
      if(firstB){
        for(const m of firstB.markets || []){
          if(m.key==="h2h"){
            for(const o of m.outcomes || []){
              if((mH>mA) && o.name===homeName) marketFavImplied = implied(o.price);
              if((mA>mH) && o.name===awayName) marketFavImplied = implied(o.price);
            }
          }
        }
      }
      const edge = marketFavImplied ? ((mH>mA)?mH-marketFavImplied:mA-marketFavImplied) : null;
      const hand = recommendHandicap(mH, mA);
      const adv = adviceFromEdge(edge);
      rows.push({
        time: new Date(e.commence_time).toLocaleString(),
        pair: `${homeName} vs ${awayName}`,
        fav: favSide,
        modelH: (mH*100).toFixed(1)+"%",
        modelA: (mA*100).toFixed(1)+"%",
        market: marketFavImplied ? ( (marketFavImplied*100).toFixed(1)+"%") : "-",
        edge: edge!==null ? ( (edge*100).toFixed(2)+"%") : "-",
        hand,
        adviceTxt: adv.txt,
        adviceCls: adv.cls
      });
    }

    const tbody = document.querySelector("#tbl tbody");
    tbody.innerHTML = "";
    for(const r of rows){
      const tr = document.createElement("tr");
      tr.innerHTML = `<td>${r.time}</td>
        <td>${r.pair}</td>
        <td>${r.fav}</td>
        <td>${r.modelH} / ${r.modelA}</td>
        <td>${r.market}</td>
        <td>${r.edge}</td>
        <td>${r.hand}</td>
        <td><span class="badge ${r.adviceCls}">${r.adviceTxt}</span></td>`;
      tbody.appendChild(tr);
    }
    document.getElementById("summaryCount").innerText = `${rows.length} คู่`;
    document.getElementById("results").style.display = rows.length? "block":"none";
    document.getElementById("status").innerText = `เรียบร้อย — พบ ${rows.length} คู่`;
  } catch (err){
    document.getElementById("status").innerText = "เกิดข้อผิดพลาด: "+err.message;
    console.error(err);
    alert("เกิดข้อผิดพลาด: "+err.message);
  }
});
</script>
</body>
</html>
