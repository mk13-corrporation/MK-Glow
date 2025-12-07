# MK-Glow
<!doctype html>
<html lang="fr">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>GlowUp Planner — Release</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.3/dist/chart.min.js"></script>
<style>
:root{--bg:#000;--panel:#071018;--muted:#9aa6b2;--accent:#fff}
*{box-sizing:border-box}
html,body{height:100%;margin:0;font-family:Inter,system-ui,Segoe UI,Roboto,Arial;}
body{background:var(--bg);color:var(--accent);padding:0}
.app{display:flex;min-height:100vh}
.sidebar{width:260px;background:#05060a;border-right:1px solid rgba(255,255,255,0.04);padding:18px;flex-shrink:0}
.brand{font-weight:800;font-size:18px;margin-bottom:8px}
.small{font-size:13px;color:var(--muted)}
.nav{display:flex;flex-direction:column;gap:8px;margin-top:12px}
.btn{background:transparent;color:var(--accent);border:1px solid rgba(255,255,255,0.04);padding:10px;border-radius:8px;text-align:left;cursor:pointer; transition: background 0.2s, color 0.2s;}
.btn.active{background:#fff;color:#000}
.main{flex:1;padding:16px;overflow:auto}
.header{display:flex;align-items:center;gap:12px;margin-bottom:12px}
.h-title{font-size:20px;font-weight:700}
.controls{margin-left:auto;display:flex;gap:8px;align-items:center}
.input,select,textarea{background:#000;border:1px solid rgba(255,255,255,0.06);padding:8px;border-radius:8px;color:var(--accent)}
.card{background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));padding:12px;border-radius:10px;border:1px solid rgba(255,255,255,0.03)}
.grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(240px,1fr));gap:12px}

/* Style pour l'emploi du temps dynamique */
.timetable-container {
    display: grid;
    grid-template-columns: 60px repeat(7, 1fr); /* Heure + 7 jours */
    gap: 0;
    border: 1px solid rgba(255, 255, 255, 0.05);
    border-radius: 8px;
    background: #000;
}
.day-header {
    text-align: center;
    padding: 8px 4px;
    font-weight: 700;
    font-size: 14px;
    border-bottom: 1px solid rgba(255, 255, 255, 0.05);
}
.time-col {
    grid-column: 1;
    position: relative;
    text-align: right;
    padding-right: 5px;
    border-right: 1px solid rgba(255, 255, 255, 0.05);
}
.time-label {
    position: absolute;
    top: -8px;
    font-size: 10px;
    color: var(--muted);
    z-index: 10;
    background: var(--bg);
    padding: 0 4px;
}
.day-column {
    position: relative;
    /* La colonne des jours sera un conteneur pour les séances */
}

/* Grille pour les lignes horaires (1 ligne par minute, ou plus simple, par heure) */
.time-slot-line {
    border-top: 1px dashed rgba(255, 255, 255, 0.03);
    height: 60px; /* 60px par heure pour une meilleure visualisation */
}

/* Séances dynamiques */
.session-block {
    position: absolute;
    /* width: 100%; (Supprimé, géré par JS) */
    /* left: 0; (Supprimé, géré par JS) */
    padding: 6px;
    border-radius: 6px;
    z-index: 20;
    cursor: pointer;
    overflow: hidden;
    color: #000;
    box-shadow: 0 2px 4px rgba(0,0,0,0.2);
    border: 1px solid transparent;
    transition: width 0.3s, left 0.3s; /* Transition pour un effet visuel lors du chevauchement */
}
.session-block.default-bg {
    background: rgba(255, 255, 255, 0.9);
}
.session-block .title {
    font-weight: 700;
    font-size: 13px;
    line-height: 1.2;
}
.session-block .time {
    font-size: 11px;
    margin-top: 2px;
}

.modal{position:fixed;inset:0;display:none;align-items:center;justify-content:center;background:rgba(0,0,0,0.6);z-index:80}
.modal-card{background:#071018;padding:14px;border-radius:8px;width:700px;max-width:95%;border:1px solid rgba(255,255,255,0.03)}
.form-row{display:flex;gap:8px;margin-top:8px;align-items:center}
.form-row.vertical { flex-direction: column; align-items: stretch; }
.table{width:100%;border-collapse:collapse;margin-top:8px}
.table th,.table td{padding:8px;border-bottom:1px dashed rgba(255,255,255,0.03);text-align:left}
.small-btn{padding:6px 8px;border-radius:6px;border:1px solid rgba(255,255,255,0.04);background:transparent;color:var(--accent);cursor:pointer; transition: background 0.2s;}
.small-btn:hover { background: rgba(255,255,255,0.08); }
.tag{padding:4px 8px;border-radius:999px;background:rgba(255,255,255,0.03);font-size:12px}
.kpi { font-size: 28px; font-weight: 800; }
/* Conteneur pour le SVG */
#svgRadarContainer { width: 100%; height: 400px; max-width: 100%; margin-top: 12px; display: flex; justify-content: center; align-items: center;}

@media(max-width:900px){ .sidebar{display:none} .timetable-container{grid-template-columns:60px repeat(1,1fr)} .header{flex-direction:column;align-items:flex-start} .modal-card{width:94%} }
</style>
</head>
<body>
<div class="app">
  <aside class="sidebar">
    <div class="brand">GlowUp Planner</div>
    <div class="small">Prototype — dark mode</div>
    <nav class="nav" id="nav">
      <button class="btn active" data-page="dashboard">Dashboard</button>
      <button class="btn" data-page="timetable">Emploi du temps</button>
      <button class="btn" data-page="notes">Notes</button>
      <button class="btn" data-page="competences">Compétences</button>
      <button class="btn" data-page="results">Résultats</button>
      <button class="btn" data-page="agenda">Agenda</button>
      <button class="btn" data-page="cahier">Cahier de texte</button>
      <button class="btn" data-page="sanctions">Auto-sanctions</button>
      <button class="btn" data-page="settings">Paramètres</button>
    </nav>
    <div style="margin-top:12px" class="small">Sauvegarde: localStorage • Export/Import JSON</div>
  </aside>

  <main class="main">
    <header class="header">
      <div class="h-title" id="pageTitle">Dashboard</div>
      <div class="controls">
        <div class="small">Semaine :</div>
        <select id="weekPicker" class="input"></select>
        <button class="small-btn" id="prevWeek">&larr;</button>
        <button class="small-btn" id="nextWeek">&rarr;</button>
        <button class="small-btn" id="exportBtn">Exporter</button>
        <button class="small-btn" id="importBtn">Importer</button>
        <input type="file" id="importFile" style="display:none">
      </div>
    </header>

    <section id="content">
      <div id="dashboardView">
        <div class="grid">
          <div class="card">
            <div style="display:flex;justify-content:space-between;align-items:center">
              <div><strong>Emploi du temps</strong><div class="small">Accès rapide → semaine courante</div></div>
              <div><button class="small-btn" onclick="navigate('timetable')">Ouvrir</button></div>
            </div>
            <div id="dash-timetable-preview" style="margin-top:8px"></div>
          </div>
          <div class="card">
            <div style="display:flex;justify-content:space-between;align-items:center">
              <div><strong>Notes</strong><div class="small">Ajouter vos matières (aucune pré-définie)</div></div>
              <div><button class="small-btn" onclick="navigate('notes')">Ouvrir</button></div>
            </div>
            <div id="dash-notes-preview" style="margin-top:8px"></div>
          </div>
          <div class="card">
            <div style="display:flex;justify-content:space-between;align-items:center">
              <div><strong>Compétences</strong><div class="small">Créez vos compétences (0 de base)</div></div>
              <div><button class="small-btn" onclick="navigate('competences')">Ouvrir</button></div>
            </div>
            <div id="dash-comp-preview" style="margin-top:8px"></div>
          </div>
          <div class="card">
            <div style="display:flex;justify-content:space-between;align-items:center">
              <div><strong>Agenda</strong><div class="small">Évènements</div></div>
              <div><button class="small-btn" onclick="navigate('agenda')">Ouvrir</button></div>
            </div>
            <div id="dash-agenda-preview" style="margin-top:8px"></div>
          </div>
        </div>
      </div>

      <div id="timetableView" style="display:none">
        <div class="card" style="margin-bottom: 12px;">
          <div style="display:flex;justify-content:space-between;align-items:center;flex-wrap:wrap;">
            <div><strong>Configuration de la grille</strong><div class="small">Configurer les heures d'affichage (pas de slots fixes)</div></div>
            <div style="display:flex;gap:8px;align-items:center;margin-top:4px;">
              <label class="small">Début</label>
              <select id="hourStart" class="input" style="width:70px"></select>
              <label class="small">Fin</label>
              <select id="hourEnd" class="input" style="width:70px"></select>
              <button class="small-btn" id="applyTimetable">Appliquer</button>
            </div>
          </div>
        </div>
        
        <div class="card">
          <div style="display:flex;justify-content:space-between;align-items:center;flex-wrap:wrap; margin-bottom: 10px;">
            <div><strong>Outils de copie</strong><div class="small">Copier un jour ou la semaine entière</div></div>
            <div style="display:flex;gap:8px;align-items:center;">
              <button class="small-btn" id="copyDayBtn">Copier Jour</button>
              <button class="small-btn" id="pasteDayBtn">Coller Jour</button>
              <span style="color:var(--muted)">|</span>
              <button class="small-btn" id="copyWeekBtn">Copier Semaine</button>
              <button class="small-btn" id="pasteWeekBtn">Coller Semaine</button>
            </div>
          </div>
          <div id="clipboardStatus" class="small" style="margin-bottom: 8px; color: yellow;"></div>

          <div id="timetableGridWrap"></div>
          <div style="margin-top:10px" class="small">Astuce: Cliquez sur la zone de l'heure désirée pour ajouter une séance.</div>
        </div>
      </div>

      <div id="notesView" style="display:none">
        <div class="card">
          <div style="display:flex;justify-content:space-between;align-items:center">
            <div><strong>Notes</strong><div class="small">Aucune matière par défaut — crée les tiennes</div></div>
            <div><button class="small-btn" id="addSubjectBtn">Ajouter matière</button></div>
          </div>
          <table class="table" id="subjectsTable"><thead><tr><th>Matière</th><th>Couleur</th><th>Coef</th><th>Notes</th><th>Moyenne</th><th></th></tr></thead><tbody></tbody></table>
        </div>
      </div>

      <div id="competencesView" style="display:none">
        <div class="card">
          <div style="display:flex;justify-content:space-between;align-items:center">
            <div><strong>Compétences</strong><div class="small">Créées à partir de zéro</div></div>
            <div><button class="small-btn" id="addCompetenceBtn">Ajouter compétence</button></div>
          </div>
          <div id="svgRadarContainer"></div>
          <div id="competenceList" style="margin-top:8px"></div>
        </div>
      </div>

      <div id="resultsView" style="display:none">
        <div class="card">
          <div><strong>Résultats</strong><div class="small">Bilans & KPI</div></div>
          <div id="resultsContent" style="margin-top:8px"></div>
        </div>
      </div>

      <div id="agendaView" style="display:none">
        <div class="card">
          <div style="display:flex;justify-content:space-between;align-items:center">
            <div><strong>Agenda</strong></div>
            <div><button class="small-btn" id="addEventBtn">Ajouter</button></div>
          </div>
          <div id="eventsList" style="margin-top:8px"></div>
        </div>
      </div>

      <div id="cahierView" style="display:none">
        <div class="card">
          <div style="display:flex;justify-content:space-between;align-items:center">
            <div><strong>Cahier de texte</strong></div>
            <div><button class="small-btn" id="addTaskBtn">Ajouter devoir</button></div>
          </div>
          <div id="tasksList" style="margin-top:8px"></div>
        </div>
      </div>

      <div id="sanctionsView" style="display:none">
        <div class="card">
          <div style="display:flex;justify-content:space-between;align-items:center">
            <div><strong>Auto-sanctions</strong></div>
            <div><button class="small-btn" id="addSanctionBtn">Nouveau</button></div>
          </div>
          <div id="sanctionsList" style="margin-top:8px"></div>
        </div>
      </div>

      <div id="settingsView" style="display:none">
        <div class="card">
          <div><strong>Paramètres & sauvegarde</strong></div>
          <div style="margin-top:8px" class="form-row">
            <button class="small-btn" id="exportJSON">Exporter JSON</button>
            <button class="small-btn" id="clearAll">Réinitialiser tout</button>
          </div>

          <hr style="margin:12px 0;border-color:rgba(255,255,255,0.03)">

          <div><strong>Templates (enregistreur de séances)</strong><div class="small">Réutilise tes séances pour gagner du temps</div></div>
          <div id="templatesList" style="margin-top:8px"></div>

          <hr style="margin:12px 0;border-color:rgba(255,255,255,0.03)">

          <div><strong>Périodes (surlignage)</strong><div class="small">Crée une période pour surligner les semaines</div></div>
          <div style="margin-top:8px" class="form-row">
            <input type="date" id="periodStart" class="input" />
            <input type="date" id="periodEnd" class="input" />
            <input type="color" id="periodColor" value="#ffd166" />
            <button class="small-btn" id="addPeriodBtn">Ajouter</button>
          </div>
          <div id="periodsList" style="margin-top:8px"></div>
        </div>
      </div>

    </section>
  </main>
</div>

<div id="modal" class="modal"><div class="modal-card" id="modalCard"></div></div>

<script>
/* State */
const STORE = 'glowup_release_v3_no_overlap_render'; // Nouvelle clé de sauvegarde
let state = {
  timetable:{config:{hourStart:8,hourEnd:18},weeks:{}}, // Weeks now store sessions {id, title, date, startH, startM, durationM, content, color}
  templates:[], 
  periods:[], 
  subjects:[], 
  competences:[],
  events:[],
  tasks:[],
  sanctions:[],
  meta:{currentWeekMonday:null}
};

let clipboard = null; // Presse-papiers pour copier/coller
let radarChart = null; 
const HOUR_HEIGHT = 60; // 60 pixels par heure dans la grille de rendu

/* --- Global Utilities & State Management --- */
function save(){ localStorage.setItem(STORE, JSON.stringify(state)); }
function load(){ 
    const s = localStorage.getItem(STORE); 
    if(s) {
        state = JSON.parse(s);
        // Nettoyage ou initialisation des configurations manquantes
        if (!state.timetable.config.hourStart) {
             state.timetable.config = {hourStart: 8, hourEnd: 18};
             save();
        }
    } else {
        init();
    }
}
function init(){
  const mon = mondayOf(new Date()); state.meta.currentWeekMonday = isoDate(mon);
  state.timetable = {config:{hourStart:8,hourEnd:18},weeks:{}}; // NOUVEAU FORMAT
  state.templates=[]; state.periods=[]; state.subjects=[]; state.competences=[]; state.events=[]; state.tasks=[]; state.sanctions=[];
  save();
}

/* helpers */
function id(){ return Math.random().toString(36).slice(2,9); }
function isoDate(d){ return new Date(d.getFullYear(),d.getMonth(),d.getDate()).toISOString().slice(0,10); }
function mondayOf(d){ 
  const x=new Date(d); 
  const day=x.getDay(); 
  const diff=(day === 0) ? 6 : day - 1; 
  x.setDate(x.getDate()-diff); 
  x.setHours(0,0,0,0); 
  return x; 
}
function addDays(d,n){ const x=new Date(d); x.setDate(x.getDate()+n); return x; }
function escapeHtml(s){ return String(s||'').replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;'); }

/**
 * Analyse une heure:minute (ex: 09:30) en heures et minutes.
 * @param {string} timeString - Format HH:MM
 * @returns {{hour: number, minute: number} | null}
 */
function parseTime(timeString) {
    const parts = (timeString || '').match(/^(\d{1,2}):(\d{2})$/);
    if (!parts) return null;
    const hour = parseInt(parts[1], 10);
    const minute = parseInt(parts[2], 10);
    if (hour >= 0 && hour <= 23 && minute >= 0 && minute <= 59) {
        return { hour, minute };
    }
    return null;
}

/* Modal */
function showModal(content){
    document.getElementById('modalCard').innerHTML = content;
    document.getElementById('modal').style.display = 'flex';
}
function closeModal(){
    document.getElementById('modal').style.display = 'none';
    document.getElementById('modalCard').innerHTML = '';
}


/* --- Navigation --- */
const pages=['dashboard','timetable','notes','competences','results','agenda','cahier','sanctions','settings'];
document.querySelectorAll('#nav .btn').forEach(b=>b.addEventListener('click',()=>{ 
    document.querySelectorAll('#nav .btn').forEach(x=>x.classList.remove('active')); 
    b.classList.add('active'); 
    navigate(b.dataset.page); 
}));

function navigate(page){
  pages.forEach(p=>document.getElementById(p+'View').style.display='none');
  const viewElement = document.getElementById(page+'View');
  if(viewElement) viewElement.style.display='block';
  document.getElementById('pageTitle').innerText = page==='timetable'?'Emploi du temps':page.charAt(0).toUpperCase()+page.slice(1);
  
  if(page === 'dashboard') renderDashboard();
  if(page === 'timetable') renderTimetable();
  if(page === 'notes') renderNotes();
  if(page === 'competences') { 
    renderCompetences(); 
    drawSvgRadarChart(); 
  }
  if(page === 'results') renderResults();
  if(page === 'agenda') renderAgenda();
  if(page === 'cahier') renderTasks();
  if(page === 'sanctions') renderSanctions();
  if(page === 'settings'){ renderTemplates(); renderPeriods(); }
}

/* Week picker & Navigation */
function populateWeekPicker(){
  const sel = document.getElementById('weekPicker'); sel.innerHTML='';
  const today = new Date();
  const base = mondayOf(today);
  const currentWeekKey = isoDate(mondayOf(today));
  
  for(let i=-52;i<=52;i++){ 
    const m = addDays(base,i*7); 
    const opt=document.createElement('option'); 
    opt.value=isoDate(m); 
    opt.text='Sem. '+isoDate(m); 
    sel.appendChild(opt); 
  }
  
  sel.value = state.meta.currentWeekMonday || currentWeekKey;
  sel.addEventListener('change',e=>{ state.meta.currentWeekMonday=e.target.value; save(); renderTimetable(); renderDashboard(); });
}

function changeWeek(direction){
    const currentMonday = new Date(state.meta.currentWeekMonday+'T00:00:00');
    const newMonday = addDays(currentMonday, direction * 7);
    state.meta.currentWeekMonday = isoDate(newMonday);
    
    const sel = document.getElementById('weekPicker');
    if (sel.querySelector(`option[value="${state.meta.currentWeekMonday}"]`)) {
        sel.value = state.meta.currentWeekMonday;
    } else {
        populateWeekPicker();
        sel.value = state.meta.currentWeekMonday;
    }

    save(); 
    renderTimetable(); 
    renderDashboard();
}

document.getElementById('prevWeek').onclick = ()=>changeWeek(-1);
document.getElementById('nextWeek').onclick = ()=>changeWeek(1);
document.getElementById('exportBtn').onclick = ()=>downloadJSON();
document.getElementById('importBtn').onclick = ()=>document.getElementById('importFile').click();
document.getElementById('importFile').onchange = function(e){ const f=e.target.files[0]; if(!f) return; const r=new FileReader(); r.onload=ev=>{ try{ 
    const newState = JSON.parse(ev.target.result); 
    if(newState && newState.timetable && newState.meta) { 
        state = newState; 
        save(); 
        renderAll(); 
        alert('Import OK'); 
    } else {
        alert('Fichier d\'import invalide.');
    }
}catch(err){ alert('Erreur d\'import: '+err.message); } }; r.readAsText(f); };

function downloadJSON(){
  const data = JSON.stringify(state,null,2);
  const blob = new Blob([data],{type:'application/json'}); const url = URL.createObjectURL(blob);
  const a = document.createElement('a'); a.href=url; a.download='glowup_planner_export_dynamic_v3.json'; a.click();
  URL.revokeObjectURL(url);
}

document.getElementById('exportJSON').onclick = ()=>downloadJSON();
document.getElementById('clearAll').onclick = ()=>{ if(confirm('ATTENTION: Voulez-vous vraiment réinitialiser toutes les données ?')) { init(); renderAll(); alert('Données réinitialisées.'); } };


/* --- Timetable controls --- */
function buildHourSelectors(){
  const hs=document.getElementById('hourStart'), he=document.getElementById('hourEnd');
  hs.innerHTML=''; he.innerHTML=''; 
  // Suppression de slotsPerDay
  for(let h=0;h<=24;h++){ const o=document.createElement('option'); o.value=h; o.text=h+':00'; hs.appendChild(o.cloneNode(true)); he.appendChild(o.cloneNode(true)); }
  hs.value=state.timetable.config.hourStart; he.value=state.timetable.config.hourEnd; 
  document.getElementById('applyTimetable').onclick = ()=>{ 
      state.timetable.config.hourStart=parseInt(hs.value); 
      state.timetable.config.hourEnd=parseInt(he.value); 
      save(); 
      renderTimetable(); 
  };
}


/* --- Render Functions --- */
function renderAll(){ 
    populateWeekPicker(); 
    buildHourSelectors(); 
    renderDashboard(); 
    renderTimetable(); 
    renderNotes(); 
    renderCompetences(); 
    renderResults(); 
    renderAgenda(); 
    renderTasks(); 
    renderSanctions(); 
}

function renderDashboard(){
  document.getElementById('dash-timetable-preview').innerHTML = `<div class="small">Semaine du ${state.meta.currentWeekMonday}</div>`;
  const np=document.getElementById('dash-notes-preview'); np.innerHTML=''; state.subjects.slice(0,3).forEach(s=> np.innerHTML+=`<div class="tag">${escapeHtml(s.name)} • moy=${subjectAverage(s).toFixed(2)}</div>`);
  const cp=document.getElementById('dash-comp-preview'); cp.innerHTML=''; state.competences.slice(0,3).forEach(c=> cp.innerHTML+=`<div class="tag">${escapeHtml(c.name)} ${c.score}%</div>`);
  const ap=document.getElementById('dash-agenda-preview'); ap.innerHTML=''; 
  state.events.filter(e => !e.done).slice(0,3).forEach(e=> ap.innerHTML+=`<div class="small">${escapeHtml(e.title)} • ${e.start}</div>`);
}

/* Timetable rendering with dynamic sessions */
function renderTimetable(){
  const wrap=document.getElementById('timetableGridWrap'); wrap.innerHTML='';
  const cfg = state.timetable.config;
  const weekKey = state.meta.currentWeekMonday || isoDate(mondayOf(new Date()));
  const monday = new Date(weekKey+'T00:00:00');
  
  // Afficher le statut du presse-papier
  const clipStatus = document.getElementById('clipboardStatus');
  if (clipboard) {
    clipStatus.innerText = `Presse-papiers : Copie de type "${clipboard.type}". Prêt à coller.`;
    clipStatus.style.color = '#10b981'; // Green
  } else {
    clipStatus.innerText = `Presse-papiers vide.`;
    clipStatus.style.color = '#f59e0b'; // Yellow
  }

  const totalHours = Math.max(1, cfg.hourEnd - cfg.hourStart);
  const totalHeight = totalHours * HOUR_HEIGHT; // Hauteur totale en pixels
  
  let html = `<div class="timetable-container" style="grid-template-rows: auto ${totalHeight}px;">`;
  
  // 1. Headers (Jours)
  const dayNames = ['Lun', 'Mar', 'Mer', 'Jeu', 'Ven', 'Sam', 'Dim'];
  const datesOfWeek = [];
  
  html += `<div class="day-header" style="border:none;"></div>`; // Coin vide
  
  // Headers des jours
  for(let d=0;d<7;d++){ 
      const dd=addDays(monday,d); 
      const ddIso = isoDate(dd);
      datesOfWeek.push(ddIso); // Stocke les dates ISO Lundi-Dimanche
      let bg='transparent';
      // Check periods
      state.periods.forEach(p=>{ 
          const s=new Date(p.start+'T00:00:00'); 
          const e=new Date(p.end+'T00:00:00'); 
          if(dd >= s && dd <= addDays(e,1)) bg=p.color; 
      });
      html += `<div class="day-header" style="background:${bg === 'transparent' ? '#000' : bg};">${dayNames[d]} ${dd.toLocaleDateString('fr-FR',{day:'2-digit',month:'2-digit'})}</div>`;
  }
  
  // 2. Colonne des heures
  html += `<div class="time-col" style="height:${totalHeight}px; grid-row: 2;">`;
  for(let h=cfg.hourStart; h<=cfg.hourEnd; h++){
      if(h !== cfg.hourStart) {
          // Ligne de l'heure
          const top = (h - cfg.hourStart) * HOUR_HEIGHT;
          html += `<div class="time-label" style="top: ${top}px;">${h.toString().padStart(2, '0')}:00</div>`;
      }
  }
  html += `</div>`; // Fin time-col
  
  // 3. Colonnes des jours et Sessions
  datesOfWeek.forEach((dateKey, dayIndex) => {
      const col = dayIndex + 2; // Colonne 2 à 8
      
      // Conteneur de la journée (clic pour ajouter une nouvelle session)
      html += `<div class="day-column" 
                      style="grid-row: 2; grid-column: ${col}; height:${totalHeight}px;" 
                      data-date="${dateKey}"
                      onclick="openSessionModal(null, '${weekKey}', '${dateKey}', event)">`;

      // Ajouter les lignes horaires de fond
      for(let h=cfg.hourStart + 1; h<=cfg.hourEnd; h++){
          const top = (h - cfg.hourStart) * HOUR_HEIGHT;
          html += `<div class="time-slot-line" style="top:${top}px;"></div>`;
      }

      // Rendu des sessions pour ce jour
      const sessions = (state.timetable.weeks?.[weekKey]?.[dateKey]) || [];
      
      // Préparation des données de session avec les coordonnées en minutes
      const sessionData = sessions.map(session => {
          const startMinutesTotal = session.startHour * 60 + session.startMinute;
          const endMinutesTotal = startMinutesTotal + session.durationMinutes;
          
          return {
              ...session,
              start: startMinutesTotal,
              end: endMinutesTotal,
              // Position Y en pixels depuis le haut de la grille (début de cfg.hourStart)
              topPx: ((startMinutesTotal / 60) - cfg.hourStart) * HOUR_HEIGHT,
              // Hauteur en pixels
              heightPx: (session.durationMinutes / 60) * HOUR_HEIGHT,
              col: -1, // Colonne d'affichage (-1 = pas encore assignée)
              width: 100, // Largeur d'affichage (%)
              left: 0 // Position horizontale (%)
          };
      }).filter(s => s.topPx >= 0 && s.topPx + s.heightPx > 0); // Filtrer les sessions hors-grille ou de durée nulle

      
      // --- Logique de détection et d'ajustement des chevauchements (pour éviter la 'division') ---
      
      // Triez par heure de début
      sessionData.sort((a, b) => a.start - b.start);

      // Structure pour garder une trace des colonnes occupées
      // Chaque entrée dans ce tableau représente une "piste" ou colonne d'affichage.
      const tracks = []; 

      sessionData.forEach((current) => {
          let placed = false;
          
          // 1. Essayer de placer la session dans une piste existante
          for (let i = 0; i < tracks.length; i++) {
              let track = tracks[i];
              // Vérifier si le dernier élément de cette piste se chevauche avec la session actuelle
              const lastInTrack = track[track.length - 1];

              // Si la session actuelle commence APRÈS la fin de la dernière session de la piste,
              // elle peut utiliser la même piste (même colonne visuelle).
              if (current.start >= lastInTrack.end) {
                  track.push(current);
                  current.col = i;
                  placed = true;
                  break;
              }
          }

          // 2. Si aucune piste n'est libre, créer une nouvelle piste
          if (!placed) {
              current.col = tracks.length;
              tracks.push([current]);
          }
      });
      
      // Calcul final de la largeur et de la position LEFT pour chaque session
      sessionData.forEach(session => {
          const maxColumns = tracks.length;
          session.width = 100 / maxColumns;
          session.left = session.col * session.width;
      });


      // Rendu HTML final des sessions ajustées
      sessionData.forEach(session => {
          const endDate = session.startHour.toString().padStart(2, '0') + ':' + session.startMinute.toString().padStart(2, '0');
          const endHour = Math.floor(session.end / 60);
          const endMinute = session.end % 60;
          const endTime = endHour.toString().padStart(2, '0') + ':' + endMinute.toString().padStart(2, '0');

          const blockStyle = `top: ${session.topPx}px; 
                              height: ${session.heightPx}px; 
                              background: ${session.color};
                              width: ${session.width}%;
                              left: ${session.left}%;`; // Utilisation de left et width ajustés
          
          html += `<div class="session-block ${session.color === 'transparent' || session.color === '#ffffff' ? 'default-bg' : ''}" 
                        style="${blockStyle}" 
                        data-session-id="${session.id}"
                        onclick="event.stopPropagation(); openSessionModal('${session.id}', '${weekKey}', '${dateKey}')">
                      <div class="title" style="color:${session.color === 'transparent' || session.color === '#ffffff' ? '#000' : 'white'}">${escapeHtml(session.title)}</div>
                      <div class="time" style="color:${session.color === 'transparent' || session.color === '#ffffff' ? 'rgba(0,0,0,0.6)' : 'rgba(255,255,255,0.8)'}">${endDate} - ${endTime}</div>
                  </div>`;
      });
      
      html += `</div>`; // Fin day-column
  });

  html += `</div>`; // Fin timetable-container
  wrap.innerHTML = html;
  
  attachCopyPasteListeners();
}


/**
 * Calcule l'heure de début estimée pour un clic dans la grille.
 * @param {Event} event - L'événement de clic
 * @param {number} hourStart - Heure de début d'affichage
 * @returns {{hour: number, minute: number}}
 */
function getEstimatedStartTime(event, hourStart) {
    const rect = event.currentTarget.getBoundingClientRect();
    const clickY = event.clientY - rect.top; // Position Y dans l'élément jour
    
    // Calcul de l'heure cliquée en minutes totales depuis le début de l'affichage
    const minutesSinceStart = (clickY / HOUR_HEIGHT) * 60;
    
    // Décalage pour obtenir l'heure réelle
    const totalMinutes = Math.round((hourStart * 60) + minutesSinceStart);
    
    // Arrondir au 5 minutes le plus proche
    const nearest5 = Math.round(totalMinutes / 5) * 5; 
    
    const hour = Math.floor(nearest5 / 60);
    const minute = nearest5 % 60;
    
    return { hour, minute };
}

/* Slot modal: create/edit and save as template option */
function openSessionModal(sessionId, weekKey, date, event = null){
    const isNew = sessionId === null;
    let session = {id:id(), title:'', content:'', startHour: 9, startMinute: 0, durationMinutes: 60, color:'#60a5fa'};

    if (!isNew) {
        const daySessions = state.timetable.weeks?.[weekKey]?.[date];
        session = daySessions?.find(s => s.id === sessionId) || session;
    } else if (event) {
        // Estimer l'heure de début pour une nouvelle session
        const estTime = getEstimatedStartTime(event, state.timetable.config.hourStart);
        session.startHour = estTime.hour;
        session.startMinute = estTime.minute;
    }

    const startH = session.startHour.toString().padStart(2, '0');
    const startM = session.startMinute.toString().padStart(2, '0');

    let modalContent = `<div style="display:flex;justify-content:space-between;align-items:center"><strong>${isNew ? 'Créer' : 'Editer'} séance (${date})</strong><div><button class="small-btn" onclick="closeModal()">X</button></div></div>
      <div class="form-row vertical">
          <input class="input" id="mTitle" placeholder="Titre" value="${escapeHtml(session.title)}" />
      </div>
      
      <div style="margin-top:8px;"><strong>Heures & Durée</strong></div>
      <div class="form-row">
          <label class="small">Début (HH:MM):</label>
          <input class="input" id="mStart" placeholder="ex: 18:30" value="${startH}:${startM}" style="width:120px;"/>
          <label class="small">Durée (min):</label>
          <input type="number" class="input" id="mDuration" value="${session.durationMinutes}" style="width:80px;"/>
      </div>
      
      <div class="form-row vertical" style="margin-top:8px;">
          <input type="color" id="mColor" value="${session.color}" style="width:100%;" />
      </div>
      <div class="form-row vertical">
          <textarea id="mContent" class="input" placeholder="Contenu">${escapeHtml(session.content)}</textarea>
      </div>
      
      <div style="margin-top: 12px; padding: 8px; border: 1px dashed rgba(255,255,255,0.1); border-radius: 6px;">
          <div class="small">Actions</div>
          <div class="form-row" style="margin-top: 6px;">
              <button class="small-btn" onclick="saveSession('${weekKey}','${date}', '${sessionId}', ${isNew})">Enregistrer</button>
              ${!isNew ? `<button class="small-btn" onclick="deleteSession('${weekKey}','${date}','${sessionId}')">Supprimer</button>` : ''}
              <button class="small-btn" onclick="saveTemplateFromSession('${weekKey}','${date}', '${sessionId}')">Enregistrer template</button>
          </div>
          
          <div class="form-row" style="margin-top: 6px;">
              <button class="small-btn" onclick="splitSession('${weekKey}','${date}', '${sessionId}')" ${isNew ? 'disabled' : ''}>Diviser la durée en deux (crée une nouvelle séance consécutive)</button>
              <button class="small-btn" onclick="mergeSession('${weekKey}','${date}', '${sessionId}')" ${isNew ? 'disabled' : ''}>Réunir avec la suivante (par chevauchement)</button>
          </div>
      </div>
      
      <div class="form-row" style="margin-top: 12px;">
          <select id="tplSelect" class="input" style="width:auto;margin-left:auto;"><option value="">Utiliser Template</option></select>
      </div>`;

    showModal(modalContent);
    
    const tplSelect = document.getElementById('tplSelect');
    state.templates.forEach(t => {
        const opt = document.createElement('option');
        opt.value = t.id;
        opt.textContent = t.name;
        tplSelect.appendChild(opt);
    });
    
    tplSelect.onchange = (e) => {
        const tpl = state.templates.find(t => t.id === e.target.value);
        if (tpl) {
            document.getElementById('mTitle').value = tpl.name;
            document.getElementById('mDuration').value = tpl.durationMinutes;
            document.getElementById('mContent').value = tpl.content;
            document.getElementById('mColor').value = tpl.color;
        }
    };
}

function saveSession(weekKey, date, sessionId, isNew) {
    const title = document.getElementById('mTitle').value;
    const durationMinutes = parseInt(document.getElementById('mDuration').value);
    const content = document.getElementById('mContent').value;
    const color = document.getElementById('mColor').value || 'transparent';
    const startTime = parseTime(document.getElementById('mStart').value);

    if (!title || !startTime || isNaN(durationMinutes) || durationMinutes <= 0) {
        return alert("Veuillez remplir le titre, l'heure de début (HH:MM) et une durée valide.");
    }
    
    const newSession = {
        id: isNew ? id() : sessionId,
        title,
        content,
        startHour: startTime.hour,
        startMinute: startTime.minute,
        durationMinutes,
        color,
        date: date
    };

    if (!state.timetable.weeks[weekKey]) state.timetable.weeks[weekKey] = {};
    if (!state.timetable.weeks[weekKey][date]) state.timetable.weeks[weekKey][date] = [];

    if (isNew) {
        state.timetable.weeks[weekKey][date].push(newSession);
    } else {
        const index = state.timetable.weeks[weekKey][date].findIndex(s => s.id === sessionId);
        if (index !== -1) {
            state.timetable.weeks[weekKey][date][index] = newSession;
        } else {
             // Fallback: si l'ID n'est pas trouvé, le traiter comme nouveau
             state.timetable.weeks[weekKey][date].push(newSession);
        }
    }
    
    // Trier les sessions par heure de début pour un meilleur rendu
    state.timetable.weeks[weekKey][date].sort((a, b) => {
        const timeA = a.startHour * 60 + a.startMinute;
        const timeB = b.startHour * 60 + b.startMinute;
        return timeA - timeB;
    });

    save(); 
    renderTimetable(); 
    renderDashboard(); 
    closeModal();
}

function deleteSession(weekKey, date, sessionId){
    if(confirm('Supprimer cette séance ?') && state.timetable.weeks[weekKey] && state.timetable.weeks[weekKey][date]){ 
        state.timetable.weeks[weekKey][date] = state.timetable.weeks[weekKey][date].filter(s => s.id !== sessionId);
        save(); 
        renderTimetable(); 
        closeModal(); 
    }
}

/**
 * Littéralement diviser une séance existante en deux séances de durée égale.
 * (Créer une nouvelle séance à la fin de la première moitié, et réduire la première)
 */
function splitSession(weekKey, date, sessionId) {
    const daySessions = state.timetable.weeks[weekKey][date];
    const originalIndex = daySessions.findIndex(s => s.id === sessionId);
    if (originalIndex === -1) return alert("Séance non trouvée.");

    const originalSession = daySessions[originalIndex];
    const duration = originalSession.durationMinutes;

    if (duration < 2) return alert("Durée trop courte pour être divisée.");

    const halfDuration = Math.floor(duration / 2);
    const secondHalfDuration = duration - halfDuration;

    // 1. Mise à jour de la séance originale (première moitié)
    originalSession.durationMinutes = halfDuration;
    originalSession.title += " (Partie 1)";
    
    // 2. Calcul de l'heure de début de la seconde moitié
    const totalMinutesStart = originalSession.startHour * 60 + originalSession.startMinute;
    const totalMinutesSecondStart = totalMinutesStart + halfDuration;
    
    const secondStartH = Math.floor(totalMinutesSecondStart / 60);
    const secondStartM = totalMinutesSecondStart % 60;

    // 3. Création de la nouvelle séance (seconde moitié)
    const newSession = {
        ...JSON.parse(JSON.stringify(originalSession)), // Copie profonde
        id: id(),
        title: originalSession.title.replace("(Partie 1)", "(Partie 2)"),
        startHour: secondStartH,
        startMinute: secondStartM,
        durationMinutes: secondHalfDuration
    };

    daySessions.push(newSession);
    
    // Trier et sauvegarder
    daySessions.sort((a, b) => (a.startHour * 60 + a.startMinute) - (b.startHour * 60 + b.startMinute));

    save();
    alert(`Séance divisée en deux sessions de ${halfDuration} min et ${secondHalfDuration} min.`);
    renderTimetable();
    closeModal();
}

/**
 * Réunir la séance actuelle avec la séance qui la suit immédiatement (par chevauchement ou heure de début).
 */
function mergeSession(weekKey, date, sessionId) {
    const daySessions = state.timetable.weeks[weekKey][date];
    const currentSession = daySessions.find(s => s.id === sessionId);
    if (!currentSession) return alert("Séance non trouvée.");

    const currentEndMinutes = currentSession.startHour * 60 + currentSession.startMinute + currentSession.durationMinutes;

    // Trouver la session immédiatement suivante (celle qui commence le plus tôt après la fin de la session actuelle)
    let nextSession = null;
    let minTimeDiff = Infinity;
    
    daySessions.forEach(s => {
        if (s.id === sessionId) return;

        const nextStartMinutes = s.startHour * 60 + s.startMinute;
        const timeDiff = nextStartMinutes - currentEndMinutes;

        // Si la session suivante commence après (ou exactement à la fin) de la session actuelle
        // On cherche également la session la plus proche en temps (même si chevauchement)
        if (nextStartMinutes > currentSession.startHour * 60 + currentSession.startMinute && 
            nextStartMinutes < currentEndMinutes + 60 && // Cherche dans la prochaine heure
            (nextSession === null || nextStartMinutes < nextSession.startHour * 60 + nextSession.startMinute)) 
        {
            // Ceci est une simplification. Dans une grille complexe, la détection est plus difficile,
            // mais ce critère est suffisant pour le cas "diviser puis réunir".
            nextSession = s;
        }
    });

    if (!nextSession) {
        return alert("Aucune séance suivante trouvée immédiatement ou par chevauchement proche.");
    }

    if (!confirm(`Voulez-vous fusionner "${currentSession.title}" avec "${nextSession.title}" ? La première prendra toute la durée.`)) {
        return;
    }

    // 1. Mise à jour de la séance actuelle (première)
    const nextEndMinutes = nextSession.startHour * 60 + nextSession.startMinute + nextSession.durationMinutes;
    const currentStartMinutes = currentSession.startHour * 60 + currentSession.startMinute;
    
    // La nouvelle durée = durée totale entre le début de la première et la fin de la seconde
    currentSession.durationMinutes = nextEndMinutes - currentStartMinutes;
    currentSession.title = currentSession.title.replace(" (Partie 1)", "").replace(" (Fusion)", "") + " (Fusion)";
    currentSession.content += "\n\n--- FUSION AVEC ---\n" + nextSession.content;
    
    // 2. Suppression de la séance suivante
    state.timetable.weeks[weekKey][date] = daySessions.filter(s => s.id !== nextSession.id);

    save();
    alert(`Séances fusionnées. Nouvelle durée totale : ${currentSession.durationMinutes} min.`);
    renderTimetable();
    closeModal();
}

/* --- Copier/Coller Logique --- */
function attachCopyPasteListeners(){
    const currentWeekKey = state.meta.currentWeekMonday;
    const monday = new Date(currentWeekKey+'T00:00:00');
    const datesOfWeek = [];
    for(let d=0;d<7;d++){ datesOfWeek.push(isoDate(addDays(monday,d))); }


    // 1. Copier Jour
    document.getElementById('copyDayBtn').onclick = () => {
        const dayHeaders = document.querySelectorAll('.day-header');
        
        let promptMessage = 'Entrez le numéro du jour à copier (1=Lun, 7=Dim) :';
        dayHeaders.forEach((h, index) => {
            if(index > 0) promptMessage += `\n${index}: ${h.innerText.split(' ')[0]} (${h.innerText.split(' ')[1]})`;
        });
        
        let dayIndex = parseInt(prompt(promptMessage));
        
        // Attention : dayHeaders[0] est le coin vide. Lun est [1], Mar est [2], etc.
        if (isNaN(dayIndex) || dayIndex < 1 || dayIndex > 7) {
            return alert("Sélection du jour invalide.");
        }
        
        const dateToCopy = datesOfWeek[dayIndex - 1]; // dayIndex 1 -> index 0 dans datesOfWeek
        const dayData = state.timetable.weeks?.[currentWeekKey]?.[dateToCopy] || [];

        // Créer une copie profonde des sessions
        const copiedData = JSON.parse(JSON.stringify(dayData));

        clipboard = {
            type: 'day',
            data: copiedData
        };
        alert(`Jour (${dateToCopy}) copié dans le presse-papiers.`);
        renderTimetable(); 
    };

    // 2. Coller Jour
    document.getElementById('pasteDayBtn').onclick = () => {
        if (!clipboard || clipboard.type !== 'day') {
            return alert("Le presse-papiers ne contient pas de données de jour à coller.");
        }

        const dayHeaders = document.querySelectorAll('.day-header');
        let promptMessage = 'Entrez le numéro du jour **de cette semaine** où coller les données (1=Lun, 7=Dim) :';
        
        dayHeaders.forEach((h, index) => {
             if(index > 0) promptMessage += `\n${index}: ${h.innerText.split(' ')[0]} (${h.innerText.split(' ')[1]})`;
        });
        
        let dayIndex = parseInt(prompt(promptMessage));
        
        if (isNaN(dayIndex) || dayIndex < 1 || dayIndex > 7) {
            return alert("Sélection du jour invalide.");
        }
        
        const targetDate = datesOfWeek[dayIndex - 1];
        
        if (!state.timetable.weeks[currentWeekKey]) {
            state.timetable.weeks[currentWeekKey] = {};
        }

        // Coller les données, générer de nouveaux IDs pour les sessions
        const newSessions = clipboard.data.map(session => ({
            ...session,
            id: id(),
            date: targetDate // Assurez-vous que la date de la session est la date cible
        }));

        state.timetable.weeks[currentWeekKey][targetDate] = newSessions;
        
        save();
        alert(`Données du jour collées sur le ${targetDate}.`);
        renderTimetable();
    };

    // 3. Copier Semaine
    document.getElementById('copyWeekBtn').onclick = () => {
        const weekData = state.timetable.weeks?.[currentWeekKey] || {};
        
        clipboard = {
            type: 'week',
            data: JSON.parse(JSON.stringify(weekData))
        };
        alert(`Semaine du ${currentWeekKey} copiée.`);
        renderTimetable(); 
    };

    // 4. Coller Semaine
    document.getElementById('pasteWeekBtn').onclick = () => {
        if (!clipboard || clipboard.type !== 'week') {
            return alert("Le presse-papiers ne contient pas de données de semaine à coller.");
        }
        
        let targetWeekKey = prompt(`Coller la semaine copiée sur quelle date de Lundi ? (Format YYYY-MM-JJ). La semaine actuelle est ${currentWeekKey}`);

        if (!targetWeekKey) return;
        
        if (!/^\d{4}-\d{2}-\d{2}$/.test(targetWeekKey)) {
            return alert("Format de date invalide. Utilisez YYYY-MM-JJ.");
        }
        
        const dateObj = new Date(targetWeekKey + 'T00:00:00');
        if (isoDate(dateObj) !== targetWeekKey || isoDate(mondayOf(dateObj)) !== targetWeekKey) {
            return alert("La date cible doit être un Lundi.");
        }

        if (confirm(`Voulez-vous vraiment écraser les données de la semaine du ${targetWeekKey} ?`)) {
             // Coller les données (copie profonde) et régénérer les IDs et mettre à jour les dates des sessions.
            const newWeekData = {};
            const copiedDays = clipboard.data;

            const baseMondayCopied = mondayOf(new Date(targetWeekKey + 'T00:00:00'));
            
            // On itère sur les 7 jours à partir du lundi cible
            for (let i = 0; i < 7; i++) {
                const targetDate = isoDate(addDays(baseMondayCopied, i));
                
                // On trouve la date correspondante dans les données copiées (ex: on copie Lun -> on colle sur Lun)
                // C'est plus facile si l'objet de la semaine contient les 7 jours, mais on se base ici sur la structure de l'objet copié.
                
                const originalDateKey = datesOfWeek[i]; 

                if(copiedDays[originalDateKey]) {
                     newWeekData[targetDate] = copiedDays[originalDateKey].map(session => ({
                        ...session,
                        id: id(),
                        date: targetDate // MAJ de la date de la session
                    }));
                }
            }

            state.timetable.weeks[targetWeekKey] = newWeekData;
            save();
            alert(`Semaine collée sur la semaine du ${targetWeekKey}.`);
            
            if (targetWeekKey !== currentWeekKey) {
                state.meta.currentWeekMonday = targetWeekKey;
                populateWeekPicker(); 
            }
            renderTimetable();
        }
    };
}


/* --- Templates (session recorder) */
function saveTemplateFromSession(weekKey, date, sessionId){
  const daySessions = state.timetable.weeks?.[weekKey]?.[date] || [];
  const session = daySessions.find(s => s.id === sessionId);

  if(!session || !session.title) return alert('Aucune séance valide à enregistrer');
  
  const tpl = {
    id:id(),
    name:session.title,
    content:session.content,
    durationMinutes: session.durationMinutes,
    color:session.color
  };
  state.templates.push(tpl); save(); renderTemplates(); alert('Template enregistré');
  closeModal(); 
}

function renderTemplates(){
  const wrap=document.getElementById('templatesList'); wrap.innerHTML='';
  if(state.templates.length===0) wrap.innerHTML='<div class="small">Aucun template</div>';
  state.templates.slice().reverse().forEach(t=>{
    const d=document.createElement('div'); d.style.display='flex'; d.style.justifyContent='space-between'; d.style.alignItems='center'; d.style.marginTop='6px';
    d.innerHTML = `<div><strong>${escapeHtml(t.name)}</strong><div class="small">${t.durationMinutes} minutes</div></div><div><button class="small-btn" onclick='deleteTemplate("${t.id}")'>Suppr</button></div>`;
    wrap.appendChild(d);
  });
}

function deleteTemplate(idt){ state.templates = state.templates.filter(t=>t.id!==idt); save(); renderTemplates(); }

/* --- Periods (surlignage) --- */
document.getElementById('addPeriodBtn').onclick = ()=>{
  const s=document.getElementById('periodStart').value; 
  const e=document.getElementById('periodEnd').value; 
  const c=document.getElementById('periodColor').value;
  if(!s||!e) return alert('Choisissez les dates'); 
  state.periods.push({id:id(),start:s,end:e,color:c}); 
  save(); 
  renderPeriods(); 
  renderTimetable();
};
function renderPeriods(){
  const wrap=document.getElementById('periodsList'); wrap.innerHTML='';
  if(state.periods.length===0) wrap.innerHTML='<div class="small">Aucune période</div>';
  state.periods.slice().reverse().forEach(p=>{ 
    const d=document.createElement('div'); 
    d.style.marginTop='6px'; 
    d.innerHTML=`<div style="display:flex;justify-content:space-between;align-items:center">
        <div><strong>${p.start} → ${p.end}</strong></div>
        <div style="display:flex;align-items:center;">
            <div style="width:18px;height:18px;background:${p.color};border-radius:4px;margin-right:8px"></div>
            <button class="small-btn" onclick='deletePeriod("${p.id}")'>Suppr</button>
        </div>
    </div>`; 
    wrap.appendChild(d); 
  });
}
function deletePeriod(pid){ state.periods = state.periods.filter(p=>p.id!==pid); save(); renderPeriods(); renderTimetable(); }

/* --- Notes --- */
function subjectAverage(s){ 
  if(!s.grades||s.grades.length===0) return 0; 
  let sum=0; 
  let totalScale=0;
  s.grades.forEach(g=>{
    sum += g.val * (20 / g.scale); // Normalise la note à /20
    totalScale += 20; // Ajoute 20 à l'échelle totale (car chaque note est normalisée à /20)
  }); 
  return Math.round((sum/s.grades.length)*100)/100; 
}
function renderNotes(){
  const tbody=document.querySelector('#subjectsTable tbody'); tbody.innerHTML='';
  if(state.subjects.length===0) tbody.innerHTML='<tr><td colspan="6" class="small">Aucune matière — ajoute la première</td></tr>';
  state.subjects.forEach(s=>{
    const avg = subjectAverage(s).toFixed(2);
    const tr=document.createElement('tr');
    tr.innerHTML = `<td><strong style="color:${s.color||'#fff'}">${escapeHtml(s.name)}</strong></td>
            <td><div style="width:18px;height:18px;background:${s.color||'#fff'};border-radius:4px"></div></td>
            <td>${s.coef||1}</td>
            <td>${s.grades.map((g,i)=>`<span class="tag" onclick="deleteGrade('${s.id}', ${i})">${g.val}/${g.scale}</span>`).join(' ')}</td>
            <td>${avg}</td>
            <td><button class="small-btn" onclick="editSubject('${s.id}')">Edit</button> <button class="small-btn" onclick="deleteSubject('${s.id}')">X</button></td>`;
    tbody.appendChild(tr);
  });
}
document.getElementById('addSubjectBtn').onclick = ()=>{
  showModal(`<div style="display:flex;justify-content:space-between;align-items:center"><strong>Ajouter Matière</strong><div><button class="small-btn" onclick="closeModal()">X</button></div></div>
    <div class="form-row vertical"><input class="input" id="newSubName" placeholder="Nom de la matière"/></div>
    <div class="form-row vertical"><input type="color" id="newSubColor" value="#ffffff" style="width:100%;"/></div>
    <div class="form-row vertical"><input type="number" class="input" id="newSubCoef" placeholder="Coefficient" value="1"/></div>
    <div class="form-row"><button class="small-btn" onclick="saveNewSubject()">Ajouter</button></div>`);
};
function saveNewSubject(){
    const name = document.getElementById('newSubName').value;
    const color = document.getElementById('newSubColor').value;
    const coef = parseFloat(document.getElementById('newSubCoef').value)||1;
    if(!name) return alert('Le nom est requis.');
    state.subjects.push({id:id(),name,coef,color,grades:[]}); 
    save(); 
    renderNotes();
    closeModal();
}
function editSubject(sid){
  const s = state.subjects.find(x=>x.id===sid); if(!s) return;
  showModal(`<div style="display:flex;justify-content:space-between;align-items:center"><strong>Edit ${escapeHtml(s.name)}</strong><div><button class="small-btn" onclick="closeModal()">X</button></div></div>
    <div class="form-row vertical"><input class="input" id="subName" value="${escapeHtml(s.name)}" /></div>
    <div class="form-row vertical"><input type="color" class="input" id="subColor" value="${s.color||'#ffffff'}" style="width:100%;" /></div>
    <div class="form-row vertical"><input type="number" class="input" id="subCoef" value="${s.coef||1}" /></div>
    <div style="margin-top:8px"><strong>Ajouter note</strong></div>
    <div class="form-row"><input class="input" id="gradeVal" placeholder="valeur (ex: 15)"/><input class="input" id="gradeScale" placeholder="sur (ex: 20)"/></div>
    <div class="form-row"><button class="small-btn" onclick="addGrade('${s.id}')">Ajouter note</button><button class="small-btn" onclick="saveSubjectEdits('${s.id}')">Sauver Edits</button></div>`);
}
function addGrade(sid){
  const v=parseFloat(document.getElementById('gradeVal').value); const sc=parseFloat(document.getElementById('gradeScale').value);
  if(isNaN(v)||isNaN(sc)||sc<=0||v<0) return alert('valeurs invalides'); 
  const s=state.subjects.find(x=>x.id===sid); 
  s.grades.push({val:v,scale:sc}); 
  save(); 
  renderNotes(); 
  closeModal(); 
  editSubject(sid); 
}
function deleteGrade(sid, index){
    const s = state.subjects.find(x=>x.id===sid);
    if(confirm(`Supprimer la note ${s.grades[index].val}/${s.grades[index].scale} ?`)){
        s.grades.splice(index, 1);
        save();
        renderNotes();
    }
}
function saveSubjectEdits(sid){ 
    const s=state.subjects.find(x=>x.id===sid); 
    s.name=document.getElementById('subName').value; 
    s.color=document.getElementById('subColor').value; 
    s.coef=parseFloat(document.getElementById('subCoef').value)||s.coef; 
    save(); 
    renderNotes(); 
    closeModal(); 
}
function deleteSubject(sid){
    if(confirm('Supprimer cette matière et toutes ses notes ?')){
        state.subjects = state.subjects.filter(x=>x.id!==sid);
        save();
        renderNotes();
    }
}

/* --- Competences --- */
function renderCompetences(){ 
    const list=document.getElementById('competenceList'); 
    list.innerHTML=''; 
    if(state.competences.length===0) list.innerHTML='<div class="small">Aucune compétence — ajoute la première</div>'; 
    state.competences.forEach(c=>{ 
        list.innerHTML+=`<div class="card" style="margin-top:8px;"><div style="display:flex;justify-content:space-between;align-items:center;">
            <div><strong>${escapeHtml(c.name)}</strong> (${c.score}%)</div>
            <div><button class="small-btn" onclick="editCompetence('${c.id}')">Edit</button> <button class="small-btn" onclick="deleteCompetence('${c.id}')">X</button></div>
        </div></div>`; 
    }); 
    drawSvgRadarChart(); 
}
document.getElementById('addCompetenceBtn').onclick = ()=>{ 
    const name=prompt('Nom compétence'); 
    if(!name) return; 
    state.competences.push({id:id(),name,score:50}); 
    save(); 
    renderCompetences(); 
};
function editCompetence(cid){ 
    const c=state.competences.find(x=>x.id===cid); 
    const v=parseInt(prompt('Score 0-100',c.score)); 
    if(!isNaN(v)){ 
        c.score=Math.max(0,Math.min(100,v)); 
        save(); 
        renderCompetences(); 
    } 
}
function deleteCompetence(cid){
    if(confirm('Supprimer cette compétence ?')){
        state.competences = state.competences.filter(c=>c.id!==cid);
        save();
        renderCompetences();
    }
}

// Diagramme Radar en SVG Natif
function drawSvgRadarChart(){
    const container = document.getElementById('svgRadarContainer');
    container.innerHTML = '';
    const competences = state.competences;
    
    if (competences.length === 0) {
        container.innerHTML = '<div class="small" style="text-align:center;">Ajoutez des compétences pour voir le diagramme.</div>';
        return;
    }

    const size = 400; 
    const center = size / 2;
    const radius = size / 2 * 0.8; 
    const numPoints = competences.length;

    let svg = `<svg width="${size}" height="${size}" viewBox="0 0 ${size} ${size}" xmlns="http://www.w3.org/2000/svg" style="overflow: visible;">`;
    
    const steps = 4; 
    const angleIncrement = (2 * Math.PI) / numPoints;
    
    function getCoords(value, index, maxRadius) {
        const angle = index * angleIncrement - Math.PI / 2; 
        const r = maxRadius * (value / 100);
        const x = center + r * Math.cos(angle);
        const y = center + r * Math.sin(angle);
        return { x, y };
    }

    for (let i = 1; i <= steps; i++) {
        let polygonPoints = '';
        const currentRadius = (i / steps) * radius;
        for (let j = 0; j < numPoints; j++) {
            const { x, y } = getCoords(100, j, currentRadius);
            polygonPoints += `${x},${y} `;
        }
        svg += `<polygon points="${polygonPoints.trim()}" stroke="rgba(255, 255, 255, 0.1)" fill="none" stroke-width="1" stroke-dasharray="2,2"/>`;
    }

    for (let i = 0; i < numPoints; i++) {
        const { x, y } = getCoords(100, i, radius);
        svg += `<line x1="${center}" y1="${center}" x2="${x}" y2="${y}" stroke="rgba(255, 255, 255, 0.2)" stroke-width="1" />`;
    }

    let dataPoints = '';
    competences.forEach((c, index) => {
        const { x, y } = getCoords(c.score, index, radius);
        dataPoints += `${x},${y} `;
    });

    svg += `<polygon points="${dataPoints.trim()}" fill="rgba(96, 165, 250, 0.2)" stroke="rgba(96, 165, 250, 1)" stroke-width="2" />`;

    competences.forEach((c, index) => {
        const { x, y } = getCoords(c.score, index, radius);
        svg += `<circle cx="${x}" cy="${y}" r="4" fill="rgba(96, 165, 250, 1)" stroke="#fff" stroke-width="1.5" />`;

        const labelOffset = 20; 
        const { x: lx, y: ly } = getCoords(100, index, radius + labelOffset);

        let anchor = 'middle';
        if (lx < center - 10) anchor = 'end'; 
        if (lx > center + 10) anchor = 'start'; 
        
        svg += `<text x="${lx}" y="${ly}" dominant-baseline="central" text-anchor="${anchor}" fill="white" font-size="12px">${escapeHtml(c.name)} (${c.score}%)</text>`;
    });

    svg += `<text x="${center}" y="${center + 5}" dominant-baseline="middle" text-anchor="middle" fill="rgba(255, 255, 255, 0.5)" font-size="10px">0%</text>`;
    const halfRadius = radius / 2;
    svg += `<text x="${center}" y="${center - halfRadius + 5}" dominant-baseline="middle" text-anchor="middle" fill="rgba(255, 255, 255, 0.5)" font-size="10px">50%</text>`;
    svg += `<text x="${center}" y="${center - radius + 5}" dominant-baseline="middle" text-anchor="middle" fill="rgba(255, 255, 255, 0.5)" font-size="10px">100%</text>`;

    svg += `</svg>`;
    container.innerHTML = svg;
}


/* --- Results --- */
function renderResults(){ 
    const cont=document.getElementById('resultsContent'); 
    const avgAll = state.subjects.length? (state.subjects.reduce((a,s)=>a+subjectAverage(s),0)/state.subjects.length).toFixed(2):'N/A'; 
    const doneTasks = state.tasks.filter(t=>t.done).length; 
    const totalTasks = state.tasks.length;
    const sanctionCount = state.sanctions.length;
    
    cont.innerHTML=`
    <div class="grid" style="gap:20px;">
        <div style="flex:1" class="card">
            <div class="kpi">${avgAll} / 20</div>
            <div class="small">Moyenne générale</div>
        </div>
        <div style="flex:1" class="card">
            <div class="kpi">${doneTasks} / ${totalTasks}</div>
            <div class="small">Tâches complétées</div>
        </div>
        <div style="flex:1" class="card">
            <div class="kpi">${sanctionCount}</div>
            <div class="small">Auto-sanctions</div>
        </div>
    </div>`; 
}

/* --- Agenda (Events) --- */
document.getElementById('addEventBtn').onclick = ()=>{ showModal(`<div style="display:flex;justify-content:space-between;align-items:center"><strong>Nouvel évènement</strong><div><button class="small-btn" onclick="closeModal()">X</button></div></div>
  <div class="form-row vertical"><input class="input" id="evTitle" placeholder="Titre"/></div>
  <div class="form-row"><input class="input" id="evDate" type="date"/></div>
  <div class="form-row"><input class="input" id="evStart" placeholder="Début (hh:mm)"/><input class="input" id="evEnd" placeholder="Fin (hh:mm)"/></div>
  <div class="form-row vertical"><input class="input" id="evPlace" placeholder="Lieu"/></div>
  <div class="form-row vertical"><textarea class="input" id="evNotes" placeholder="Notes"></textarea></div>
  <div class="form-row"><button class="small-btn" onclick="saveEvent()">Ajouter</button></div>`); };

function saveEvent(){ 
    const e={
        id:id(),
        title:document.getElementById('evTitle').value||'Ev',
        start:document.getElementById('evDate').value,
        timeStart:document.getElementById('evStart').value,
        timeEnd:document.getElementById('evEnd').value,
        place:document.getElementById('evPlace').value,
        notes:document.getElementById('evNotes').value,
        done:false 
    }; 
    if(!e.start) return alert('La date est requise');
    state.events.push(e); 
    state.events.sort((a,b) => a.start.localeCompare(b.start)); 
    save(); 
    renderAgenda(); 
    closeModal(); 
}

function renderAgenda(){ 
    const wrap=document.getElementById('eventsList'); wrap.innerHTML=''; 
    if(state.events.length===0) wrap.innerHTML='<div class="small">Aucun évènement</div>'; 
    
    const groupedEvents = state.events.reduce((acc, e) => {
        const date = e.start;
        if (!acc[date]) acc[date] = [];
        acc[date].push(e);
        return acc;
    }, {});

    let html = '';
    const sortedDates = Object.keys(groupedEvents).sort();
    
    sortedDates.forEach(date => {
        html += `<div style="margin-top:12px; font-weight:700; color:white;">${new Date(date).toLocaleDateString('fr-FR', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' })}</div>`;
        groupedEvents[date].forEach(e => {
            const opacity = e.done ? 'opacity:0.6;' : '';
            const textDecoration = e.done ? 'text-decoration:line-through;' : '';
            
            html += `<div class="card" style="margin-top:6px; ${opacity}">
                <div style="display:flex;justify-content:space-between;align-items:center;">
                    <div style="flex:1; display:flex; align-items:center;">
                        <input type="checkbox" style="margin-right:10px; width:18px; height:18px;" 
                               ${e.done ? 'checked' : ''} 
                               onclick="toggleEventDone('${e.id}')">
                        
                        <div>
                            <strong style="${textDecoration}">${escapeHtml(e.title)}</strong>
                            <div class="small">${e.timeStart} ${e.timeEnd? '→ '+e.timeEnd : ''} ${e.place? '• '+escapeHtml(e.place) : ''}</div>
                        </div>
                    </div>
                    <div><button class="small-btn" onclick="deleteEvent('${e.id}')">Suppr</button></div>
                </div>
            </div>`;
        });
    });

    wrap.innerHTML = html; 
}

function toggleEventDone(idv){ 
    const event = state.events.find(e=>e.id===idv);
    if(event) event.done = !event.done;
    save(); 
    renderAgenda(); 
}

function deleteEvent(idv){ state.events=state.events.filter(x=>x.id!==idv); save(); renderAgenda(); }

/* --- Tasks / Cahier --- */
document.getElementById('addTaskBtn').onclick = ()=>{ showModal(`<div style="display:flex;justify-content:space-between;align-items:center"><strong>Nouveau Devoir</strong><div><button class="small-btn" onclick="closeModal()">X</button></div></div>
  <div class="form-row vertical"><input class="input" id="taskTitle" placeholder="Devoir/Tâche"/></div>
  <div class="form-row vertical"><input class="input" id="taskSubject" placeholder="Matière (facultatif)"/></div>
  <div class="form-row"><label class="small">Date Limite</label><input class="input" id="taskDate" type="date"/></div>
  <div class="form-row vertical"><textarea class="input" id="taskNotes" placeholder="Détails"></textarea></div>
  <div class="form-row"><button class="small-btn" onclick="saveTask()">Ajouter</button></div>`); };

function saveTask(){ 
    const t={
        id:id(),
        title:document.getElementById('taskTitle').value||'Tâche',
        subject:document.getElementById('taskSubject').value,
        due:document.getElementById('taskDate').value,
        notes:document.getElementById('taskNotes').value,
        done:false
    }; 
    if(!t.due) return alert('La date limite est requise');
    state.tasks.push(t); 
    state.tasks.sort((a,b) => a.due.localeCompare(b.due)); 
    save(); 
    renderTasks(); 
    closeModal(); 
}

function renderTasks(){ 
    const wrap=document.getElementById('tasksList'); wrap.innerHTML=''; 
    if(state.tasks.length===0) wrap.innerHTML='<div class="small">Aucun devoir à faire</div>'; 
    
    const todo = state.tasks.filter(t => !t.done).sort((a,b) => a.due.localeCompare(b.due));
    const done = state.tasks.filter(t => t.done).sort((a,b) => a.due.localeCompare(b.due));

    let html = '';
    
    if (todo.length > 0) {
        html += '<div style="font-weight:700; margin-bottom: 6px;">À faire</div>';
        todo.forEach(t => {
            html += `<div class="card" style="margin-bottom:6px;">
                <div style="display:flex;justify-content:space-between;align-items:center;">
                    <div style="flex:1">
                        <strong>${escapeHtml(t.title)}</strong> ${t.subject ? `<span class="tag" style="background:rgba(255,255,255,0.06); margin-left:8px;">${escapeHtml(t.subject)}</span>` : ''}
                        <div class="small">Échéance: ${t.due}</div>
                    </div>
                    <div style="display:flex;gap:6px;">
                        <button class="small-btn" onclick="toggleTaskDone('${t.id}')">Fait</button>
                        <button class="small-btn" onclick="deleteTask('${t.id}')">Suppr</button>
                    </div>
                </div>
            </div>`;
        });
    }

    if (done.length > 0) {
        html += '<div style="font-weight:700; margin-top:12px; margin-bottom: 6px; color:rgba(255,255,255,0.5);">Terminé</div>';
        done.forEach(t => {
            html += `<div class="card" style="margin-bottom:6px; background:rgba(255,255,255,0.01);">
                <div style="display:flex;justify-content:space-between;align-items:center; opacity:0.6;">
                    <div style="flex:1">
                        <strong style="text-decoration:line-through;">${escapeHtml(t.title)}</strong>
                        <div class="small">Terminé</div>
                    </div>
                    <div style="display:flex;gap:6px;">
                        <button class="small-btn" onclick="toggleTaskDone('${t.id}')">Annuler</button>
                        <button class="small-btn" onclick="deleteTask('${t.id}')">Suppr</button>
                    </div>
                </div>
            </div>`;
        });
    }
    
    wrap.innerHTML = html; 
}

function toggleTaskDone(idv){ 
    const task = state.tasks.find(t=>t.id===idv);
    if(task) task.done = !task.done;
    save(); 
    renderTasks(); 
    renderResults(); 
}
function deleteTask(idv){ state.tasks=state.tasks.filter(x=>x.id!==idv); save(); renderTasks(); renderResults(); }

/* --- Sanctions --- */
document.getElementById('addSanctionBtn').onclick = ()=>{ showModal(`<div style="display:flex;justify-content:space-between;align-items:center"><strong>Nouvelle Auto-sanction</strong><div><button class="small-btn" onclick="closeModal()">X</button></div></div>
  <div class="form-row vertical"><input class="input" id="sanctionReason" placeholder="Raison de la sanction"/></div>
  <div class="form-row vertical"><input class="input" id="sanctionPunishment" placeholder="Pénalité (ex: 1h sans téléphone)"/></div>
  <div class="form-row"><label class="small">Date</label><input class="input" id="sanctionDate" type="date" value="${isoDate(new Date())}"/></div>
  <div class="form-row"><button class="small-btn" onclick="saveSanction()">Ajouter</button></div>`); };

function saveSanction(){ 
    const s={
        id:id(),
        reason:document.getElementById('sanctionReason').value||'Manquement',
        punishment:document.getElementById('sanctionPunishment').value,
        date:document.getElementById('sanctionDate').value
    }; 
    if(!s.punishment) return alert('La pénalité est requise');
    state.sanctions.push(s); 
    state.sanctions.sort((a,b) => b.date.localeCompare(a.date)); 
    save(); 
    renderSanctions(); 
    renderResults(); 
    closeModal(); 
}

function renderSanctions(){ 
    const wrap=document.getElementById('sanctionsList'); wrap.innerHTML=''; 
    if(state.sanctions.length===0) wrap.innerHTML='<div class="small">Aucune auto-sanction enregistrée</div>'; 
    
    state.sanctions.forEach(s => {
        wrap.innerHTML += `<div class="card" style="margin-bottom:6px;">
            <div style="display:flex;justify-content:space-between;align-items:center;">
                <div style="flex:1">
                    <strong>${escapeHtml(s.reason)}</strong>
                    <div class="small">Pénalité: ${escapeHtml(s.punishment)} • Date: ${s.date}</div>
                </div>
                <div><button class="small-btn" onclick="deleteSanction('${s.id}')">Suppr</button></div>
            </div>
        </div>`;
    });
}
function deleteSanction(idv){ state.sanctions=state.sanctions.filter(x=>x.id!==idv); save(); renderSanctions(); renderResults(); }


/* --- Initialisation --- */
load();
renderAll();
navigate('dashboard');
</script>
</body>
</html>
