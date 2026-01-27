<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Concept Energies – Demande de Matériel (WhatsApp + Historique)</title>
<style>
:root{
  --bg:#050708;
  --panel:#0b1114;
  --accent:#16f2c3;
  --accent2:#00cfa3;
  --danger:#ff4d4d;
  --text:#eafff9;
  --muted:#9bded0;
}

*{ box-sizing:border-box; }

body{
  margin:0;
  font-family:system-ui, Arial, sans-serif;
  background:linear-gradient(180deg,#020404,#050708);
  color:var(--text);
  display:flex;
  justify-content:center;
  align-items:flex-start;
  min-height:100vh;
  padding:14px;
}

.app{
  width:460px;
  max-width:100%;
  background:rgba(15,22,32,.80);
  border-radius:18px;
  border:1px solid rgba(22,242,195,.40);
  box-shadow:0 0 30px rgba(22,242,195,.25);
  padding:18px;
}

h1{
  margin:0 0 8px;
  text-align:center;
  color:var(--accent);
  letter-spacing:1px;
  font-size:18px;
}

p.sub{
  margin:0 0 12px;
  text-align:center;
  color:var(--muted);
  font-size:13px;
}

#summary{
  margin:10px 0 10px;
  padding:10px;
  border-radius:12px;
  background:#061015;
  border:1px solid rgba(22,242,195,.18);
  color:var(--muted);
  font-size:13px;
}

label{
  display:block;
  margin-top:12px;
  font-size:14px;
}

input, textarea, select{
  width:100%;
  margin-top:6px;
  padding:10px;
  border-radius:10px;
  border:none;
  background:#0a1317;
  color:white;
  font-size:14px;
}

textarea{resize:none;height:78px;}

button{
  margin-top:12px;
  width:100%;
  padding:12px;
  border-radius:12px;
  border:none;
  background:linear-gradient(90deg,var(--accent),var(--accent2));
  color:#002b22;
  font-size:16px;
  font-weight:800;
  cursor:pointer;
}

button:hover{opacity:.92;}

.btn-secondary{
  background:#0a1317;
  color:var(--text);
  border:1px solid rgba(22,242,195,.25);
}

.btn-danger{
  background:rgba(255,77,77,.14);
  border:1px solid rgba(255,77,77,.35);
  color:var(--text);
}

.badge{
  display:inline-flex;
  align-items:center;
  gap:6px;
  padding:3px 8px;
  border-radius:999px;
  border:1px solid rgba(22,242,195,.25);
  background:rgba(22,242,195,.06);
  color:var(--text);
  font-size:12px;
  margin-left:8px;
}

.badge.urgent{
  border-color: rgba(255,77,77,.45);
  background: rgba(255,77,77,.10);
}

.list{
  margin-top:16px;
  border-top:1px solid rgba(255,255,255,.10);
  padding-top:12px;
}

.item{
  background:#071015;
  border-radius:12px;
  padding:10px;
  margin-bottom:10px;
  border-left:4px solid var(--accent);
}

.item small{color:var(--muted);}

.linkbtn{
  display:inline-block;
  margin-top:8px;
  padding:8px 10px;
  border-radius:10px;
  border:1px solid rgba(22,242,195,.25);
  background:#0a1317;
  color:var(--text);
  text-decoration:none;
  font-weight:700;
}

.linkbtn:hover{opacity:.92;}

hr.sep{
  border:none;
  border-top:1px solid rgba(255,255,255,.10);
  margin:14px 0;
}

.note{
  color:var(--muted);
  font-size:12px;
  line-height:1.35;
}
</style>
</head>
<body>

<div class="app">
  <h1>⚡ CONCEPT ENERGIES</h1>
  <p class="sub">Demande de matériel chantier → WhatsApp + tableau central</p>

  <div id="summary"></div>

  <label>Ouvrier</label>
  <input id="worker" placeholder="Nom de l'ouvrier" />

  <label>Chantier</label>
  <input id="site" placeholder="Nom ou numéro du chantier" />

  <label>Type de matériel</label>
  <select id="type">
    <option>Câbles</option>
    <option>Tableau électrique</option>
    <option>Disjoncteurs</option>
    <option>Gaines</option>
    <option>Éclairage</option>
    <option>Outillage</option>
    <option>Autre</option>
  </select>

  <label>Urgence</label>
  <button type="button" id="urgentBtn" class="btn-secondary" onclick="toggleUrgent()">🚨 Urgence : NON</button>

  <label>Détail / quantité</label>
  <textarea id="details" placeholder="Ex: câble 3G2.5 – 100m"></textarea>

  <label>Photo (optionnel)</label>
  <input id="photo" type="file" accept="image/*" capture="environment" onchange="handlePhoto(event)" />
  <div id="photoInfo" class="note"></div>

  <button onclick="sendRequest()">📲 Envoyer la demande (WhatsApp)</button>

  <button class="btn-secondary" type="button" onclick="exportCSV()">🧾 Export Excel (CSV)</button>
  <button class="btn-danger" type="button" onclick="clearAll()">🗑️ Vider l’historique (ce téléphone)</button>

  <hr class="sep" />
  <div class="note">
    📌 <b>Photo + WhatsApp</b> : WhatsApp ne permet pas d'attacher la photo automatiquement via le lien. Ici, la photo est enregistrée dans l’historique : ouvre-la ensuite et partage-la dans le même chat WhatsApp.
  </div>

  <div class="list" id="list"></div>
</div>

<script>
// ====== CONFIG ======
const WHATSAPP_PHONE = "352621515160"; // bureau / stock (format sans +)

// (Option B) Tableau central : colle ici l'URL de ton Google Apps Script (Web App)
// Exemple: https://script.google.com/macros/s/XXXX/exec
const SHEET_WEBAPP_URL = "";
// ====================

let requests = JSON.parse(localStorage.getItem("materiel")) || [];
let isUrgent = false;
let photoDataUrl = "";
let photoName = "";

function save(){
  localStorage.setItem("materiel", JSON.stringify(requests));
}

function toggleUrgent(){
  isUrgent = !isUrgent;
  const btn = document.getElementById("urgentBtn");
  btn.textContent = isUrgent ? "🚨 Urgence : OUI (aujourd’hui)" : "🚨 Urgence : NON";
  btn.style.borderColor = isUrgent ? "rgba(255,77,77,.45)" : "rgba(22,242,195,.25)";
}

function handlePhoto(event){
  const file = event.target.files && event.target.files[0];
  if(!file){
    photoDataUrl = "";
    photoName = "";
    document.getElementById("photoInfo").textContent = "";
    return;
  }
  if(file.size > 2_000_000){
    alert("Photo trop lourde (max 2 Mo). Prends une photo plus légère.");
    event.target.value = "";
    return;
  }
  const reader = new FileReader();
  reader.onload = () => {
    photoDataUrl = String(reader.result || "");
    photoName = file.name || "photo";
    document.getElementById("photoInfo").textContent = `Photo ajoutée : ${photoName}`;
  };
  reader.readAsDataURL(file);
}

function buildWhatsAppText({worker, site, type, details, urgent, hasPhoto, date}){
  const lines = [];
  lines.push("⚡ DEMANDE MATERIEL – CONCEPT ENERGIES");
  if(urgent) lines.push("🚨 URGENT : AUJOURD’HUI");
  lines.push(`👷 Ouvrier : ${worker}`);
  lines.push(`🏗️ Chantier : ${site}`);
  lines.push(`📦 Type : ${type}`);
  lines.push(`📝 Détail : ${details}`);
  if(hasPhoto) lines.push("📸 Photo : enregistrée dans l’historique (à envoyer après)");
  lines.push(`⏰ Date : ${date}`);
  return lines.join("
");
}

async function sendToSheet(payload){
  if(!SHEET_WEBAPP_URL) return; // pas configuré
  try{
    await fetch(SHEET_WEBAPP_URL, {
      method: "POST",
      headers: {"Content-Type":"application/json"},
      body: JSON.stringify(payload)
    });
  }catch(e){
    // On ne bloque jamais WhatsApp si la connexion est faible sur chantier
    console.log("Sheet error", e);
  }
}

async function sendRequest(){
  const worker = document.getElementById("worker").value.trim();
  const site = document.getElementById("site").value.trim();
  const type = document.getElementById("type").value;
  const details = document.getElementById("details").value.trim();

  if(!worker || !site || !details){
    alert("Merci de remplir Ouvrier, Chantier et Détail / quantité");
    return;
  }

  const date = new Date().toLocaleString();
  const hasPhoto = !!photoDataUrl;

  const record = {
    worker, site, type, details,
    urgent: isUrgent ? "Oui" : "Non",
    hasPhoto,
    photoName,
    photoDataUrl,
    date
  };

  // 1) Enregistre local
  requests.unshift(record);
  save();
  render();

  // 2) Envoi tableau central (optionnel)
  await sendToSheet({
    worker, site, type, details,
    urgent: isUrgent ? "Oui" : "Non",
    hasPhoto: hasPhoto ? "Oui" : "Non",
    photoName: photoName || "",
    date
  });

  // 3) Ouvre WhatsApp
  const text = buildWhatsAppText({worker, site, type, details, urgent:isUrgent, hasPhoto, date});
  const url = `https://wa.me/${WHATSAPP_PHONE}?text=${encodeURIComponent(text)}`;
  window.open(url, "_blank");

  // Reset champs (on garde urgence)
  document.getElementById("details").value = "";
  document.getElementById("photo").value = "";
  document.getElementById("photoInfo").textContent = "";
  photoDataUrl = "";
  photoName = "";
}

function clearAll(){
  if(confirm("Vider tout l’historique sur ce téléphone ?")){
    requests = [];
    save();
    render();
  }
}

function remove(index){
  if(confirm("Marquer comme traité ?")){
    requests.splice(index, 1);
    save();
    render();
  }
}

function exportCSV(){
  const header = ["date","ouvrier","chantier","type","detail","urgent","photo"];
  const rows = requests.map(r => ([
    r.date || "",
    r.worker || "",
    r.site || "",
    r.type || "",
    (r.details || "").replace(/
?
/g," "),
    r.urgent || "Non",
    r.hasPhoto ? (r.photoName || "OUI") : "NON"
  ]));

  const esc = (s)=>('"'+String(s).replace(/"/g,'""')+'"');
  const csv = [header.map(esc).join(";"), ...rows.map(row => row.map(esc).join(";"))].join("
");

  const blob = new Blob(["﻿"+csv], {type:"text/csv;charset=utf-8;"});
  const a = document.createElement("a");
  a.href = URL.createObjectURL(blob);
  a.download = `demandes_materiel_${new Date().toISOString().slice(0,10)}.csv`;
  document.body.appendChild(a);
  a.click();
  a.remove();
}

function workerStats(){
  const stats = new Map();
  for(const r of requests){
    const k = (r.worker || "(inconnu)").trim();
    stats.set(k, (stats.get(k) || 0) + 1);
  }
  return Array.from(stats.entries()).sort((a,b)=>b[1]-a[1]);
}

function render(){
  const list = document.getElementById("list");
  const summary = document.getElementById("summary");

  const total = requests.length;
  const urg = requests.filter(r => r.urgent === "Oui").length;
  const top = workerStats().slice(0,4).map(([w,n])=>`${w}: ${n}`).join(" • ");
  summary.innerHTML = `📊 Demandes enregistrées : <b>${total}</b> • 🚨 urgentes : <b>${urg}</b><br>${top ? "👷 Par ouvrier : " + top : ""}`;

  list.innerHTML = "";

  requests.forEach((r,i)=>{
    const div = document.createElement("div");
    div.className = "item";

    const urgentBadge = r.urgent === "Oui" ? '<span class="badge urgent">🚨 urgent</span>' : '';
    const photoBadge = r.hasPhoto ? '<span class="badge">📸 photo</span>' : '';

    let photoBlock = "";
    if(r.hasPhoto && r.photoDataUrl){
      photoBlock = `
        <a class="linkbtn" href="${r.photoDataUrl}" target="_blank">📸 Ouvrir la photo</a>
        <div class="note" style="margin-top:6px">Ouvre la photo puis partage-la dans le même chat WhatsApp.</div>
      `;
    }

    div.innerHTML = `
      <b>${r.worker}</b> – ${r.site}${urgentBadge}${photoBadge}<br>
      <small>${r.type} • ${r.date || ""}</small><br>
      ${String(r.details || "").replace(/
/g,"<br>")}
      ${photoBlock}
      <button class="btn-danger" type="button" onclick="remove(${i})">✔ Traité</button>
    `;

    list.appendChild(div);
  });
}

render();
</script>

</body>
</html>
