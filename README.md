<!doctype html>
<html lang="pt">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>WorldGuessr — Jogo de localização</title>
<style>
  :root{
    --bg:#0f1724; --card:#0b1220; --accent:#3b82f6; --muted:#94a3b8; --glass: rgba(255,255,255,0.04);
  }
  html,body{height:100%; margin:0; font-family: Inter, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial; background: linear-gradient(180deg,#071029 0%, #0f1724 100%); color:#e6eef8;}
  .app{
    display:grid; grid-template-columns: 1fr; grid-template-rows:auto 1fr; gap:12px; padding:18px; max-width:1400px; margin:0 auto;
  }
  header{display:flex; align-items:center; gap:16px; justify-content:space-between;}
  .brand{display:flex; gap:12px; align-items:center;}
  .logo{width:56px;height:56px;border-radius:12px;background:linear-gradient(135deg,var(--accent),#06b6d4);display:flex;align-items:center;justify-content:center;font-weight:700;box-shadow:0 6px 20px rgba(11,20,40,0.6);}
  h1{font-size:20px;margin:0}
  .controls{display:flex;gap:8px;align-items:center}
  .btn{background:var(--glass); border:1px solid rgba(255,255,255,0.03); padding:8px 12px; border-radius:10px; cursor:pointer; color:inherit; font-weight:600;}
  .btn.primary{background:linear-gradient(90deg,var(--accent),#06b6d4); color:#021124; box-shadow:0 8px 30px rgba(59,130,246,0.14);}
  .card{background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01)); border-radius:14px; padding:14px; box-shadow: 0 8px 30px rgba(2,6,23,0.6); border:1px solid rgba(255,255,255,0.02);}
  main{display:grid; grid-template-columns: 520px 1fr; gap:16px; align-items:start;}
  #menu{min-width:320px;}
  .menu-row{display:flex; gap:8px; margin:8px 0; flex-wrap:wrap;}
  label{display:block; font-size:13px; color:var(--muted); margin-bottom:6px;}
  select,input[type=number]{background:transparent;border:1px solid rgba(255,255,255,0.03); color:inherit; padding:8px;border-radius:8px; min-width:140px;}
  .small{font-size:13px; color:var(--muted);}
  #game-area{position:relative; min-height:520px; overflow:hidden;}
  #pano{width:100%; height:520px; border-radius:12px; background:linear-gradient(180deg,#0b1220,#041025); display:flex; align-items:center; justify-content:center; color:var(--muted); position:relative;}
  #top-controls{position:absolute; left:16px; top:16px; display:flex; gap:8px; z-index:20;}
  #finishBtn{background:#ef4444; color:white; border:none; padding:8px 12px; border-radius:10px; cursor:pointer; font-weight:700;}
  #timer{background:rgba(0,0,0,0.35); padding:6px 10px; border-radius:10px; font-weight:700; color:#fff; display:inline-block;}
  #minimap{position:absolute; right:16px; bottom:16px; width:240px; height:140px; border-radius:10px; overflow:hidden; border:1px solid rgba(255,255,255,0.03); background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01)); z-index:15;}
  #minimap svg{width:100%; height:100%; display:block; cursor:crosshair;}
  #infoPanel{margin-top:12px;}
  .stat{display:flex;justify-content:space-between;padding:10px;border-radius:10px;background:rgba(255,255,255,0.02);margin-bottom:8px;}
  .results{display:none; padding:12px; margin-top:12px;}
  .rounds-list{max-height:200px; overflow:auto; margin-top:8px;}
  footer{display:flex;justify-content:space-between;align-items:center;color:var(--muted); font-size:13px; padding:8px 4px;}
  .lang-toggle{display:flex; gap:6px; align-items:center;}
  .chip{padding:6px 8px; border-radius:8px; background:rgba(255,255,255,0.02); border:1px solid rgba(255,255,255,0.02); cursor:pointer;}
  .hide{display:none;}
  .marker-real{fill:#22c55e; stroke:#fff; stroke-width:0.8}
  .marker-guess{fill:#fb923c; stroke:#fff; stroke-width:0.7}
  .summary-map{width:100%; height:360px; border-radius:8px; background:linear-gradient(180deg,#071029,#081126); border:1px solid rgba(255,255,255,0.02); margin-top:12px; display:flex; align-items:center; justify-content:center; color:var(--muted);}
  .center{display:flex;align-items:center;justify-content:center}
  /* responsive */
  @media (max-width:1100px){ main{grid-template-columns: 1fr; } #pano{height:420px} }
</style>
</head>
<body>
<div class="app">
  <header>
    <div class="brand">
      <div class="logo">WG</div>
      <div>
        <h1 id="title">WorldGuessr</h1>
        <div class="small" id="subtitle">Descobre onde no mundo estás — Street View / Panorâmico</div>
      </div>
    </div>
    <div class="controls">
      <div class="lang-toggle card" style="display:flex; gap:8px; align-items:center;">
        <div class="chip" id="lang-pt">PT</div>
        <div class="chip" id="lang-en">EN</div>
      </div>
      <button class="btn" id="viewScoresBtn">Histórico</button>
    </div>
  </header>

  <main>
    <aside id="menu" class="card">
      <div>
        <label id="lbl-level">Nível / Mode</label>
        <div class="menu-row">
          <button class="btn" data-level="easy" id="btnEasy">Fácil / Easy</button>
          <button class="btn" data-level="medium" id="btnMedium">Médio / Medium</button>
          <button class="btn" data-level="hard" id="btnHard">Difícil / Hard</button>
          <button class="btn" data-level="infinite" id="btnInfinite">Infinito / Infinite</button>
          <button class="btn" data-level="custom" id="btnCustom">Personalizado / Custom</button>
        </div>
        <div id="customArea" class="hide" style="margin-top:10px;">
          <label id="lbl-custom-n">Número de rondas / Rounds</label>
          <input type="number" id="customRounds" min="1" max="100" value="10" />
          <label id="lbl-custom-t">Tempo por ronda (s) / Time (s)</label>
          <input type="number" id="customSeconds" min="10" max="180" value="60" />
        </div>

        <div style="margin-top:12px;">
          <label id="lbl-loc-type">Tipo de locais / Location type</label>
          <select id="difficultyFilter">
            <option value="mixed">Misturado / Mixed</option>
            <option value="cities">Cidades / Cities</option>
            <option value="villages">Vilas & Aldeias / Villages</option>
          </select>
        </div>

        <div style="margin-top:12px;">
          <label id="lbl-lang">Idioma / Language</label>
          <select id="uiLang">
            <option value="pt">Português</option>
            <option value="en">English</option>
          </select>
        </div>

        <div style="margin-top:12px;">
          <label id="lbl-online">Detecção de Internet / Online detection</label>
          <div class="small" id="onlineState">A verificar...</div>
        </div>

        <div style="margin-top:14px; display:flex; gap:8px;">
          <button class="btn primary" id="startBtn">Começar / Start</button>
          <button class="btn" id="resetScores">Limpar histórico / Clear history</button>
        </div>

        <div id="infoPanel">
          <div class="stat"><div id="statMode">Modo</div><div id="statRounds">—</div></div>
          <div class="stat"><div id="statTime">Tempo</div><div id="statRemain">—</div></div>
          <div class="stat"><div id="statScore">Pontuação</div><div id="statPoints">0</div></div>
        </div>

        <div class="results card" id="endSummary">
          <h3 id="summaryTitle">Resumo do jogo</h3>
          <div id="summaryStats" class="small"></div>
          <div class="rounds-list" id="roundsList"></div>
          <div style="margin-top:8px;"><button class="btn primary" id="backToMenu">Voltar ao menu / Back</button></div>
        </div>
      </div>
    </aside>

    <section id="game-area" class="card">
      <div id="pano">
        <div id="top-controls">
          <div id="timer" class="center">00:00</div>
          <button id="finishBtn">Acabar</button>
        </div>

        <!-- Street View container (if online) or static panorama (if offline) -->
        <div id="streetviewContainer" style="width:100%;height:100%; display:block;">
          <div id="panoInner" style="width:100%;height:100%;"></div>
        </div>

        <!-- minimap: simple interactive equirectangular canvas/svg -->
        <div id="minimap" title="Clica para colocar o teu palpite">
          <svg viewBox="0 0 360 180" id="worldSvg" xmlns="http://www.w3.org/2000/svg" style="background:linear-gradient(180deg,#05283d,#05283d);">
            <!-- simple ocean -->
            <rect x="0" y="0" width="360" height="180" fill="#062b42"/>
            <!-- graticule -->
            <g stroke="rgba(255,255,255,0.04)" stroke-width="0.25">
              <path d="M0,30 L360,30" />
              <path d="M0,60 L360,60" />
              <path d="M0,90 L360,90" />
              <path d="M0,120 L360,120" />
              <path d="M0,150 L360,150" />
            </g>
            <!-- marker groups -->
            <g id="markers"></g>
            <text id="minimapHint" x="6" y="16" font-size="10" fill="#a9c5d9">Mapa-múndi: clica para adivinhar</text>
          </svg>
        </div>
      </div>

      <div style="margin-top:12px;">
        <div class="card" style="padding:12px;">
          <div style="display:flex;justify-content:space-between;align-items:center;">
            <div><strong id="roundInfo">Ronda —</strong> <span class="small" id="locHint">—</span></div>
            <div class="small">Distância: <span id="lastDistance">—</span> • Pontos: <span id="lastPoints">—</span></div>
          </div>

          <div style="margin-top:12px; display:flex; gap:8px; align-items:center;">
            <button class="btn" id="revealBtn">Revelar local (após terminar)</button>
            <button class="btn" id="nextBtn">Próxima / Next</button>
            <button class="btn" id="stopGameBtn">Parar Jogo / Stop</button>
          </div>

          <div style="margin-top:12px;">
            <h4 id="historyTitle">Últimos jogos / Scores</h4>
            <div class="rounds-list" id="scoresHistory"></div>
          </div>
        </div>

        <div class="card" style="margin-top:12px;">
          <h4 id="howPlayTitle">Como jogar / How to play</h4>
          <ul class="small">
            <li>Com internet: explora no Street View, escolhe onde achas que o local está clicando no mapa-múndi (canto inferior direito) e pressiona "Acabar" antes do tempo.</li>
            <li>Sem internet: verás uma vista panorâmica estática (não dá para andar). Usa o mapa-múndi para chutar a localização e pressiona "Acabar".</li>
            <li>Quanto mais perto, mais pontos. Se não responderes dentro do tempo tens 0 pontos nessa ronda.</li>
          </ul>
        </div>
      </div>
    </section>
  </main>

  <footer>
    <div class="small">WorldGuessr — Feito para jogar localmente. API Key do Maps usada conforme pedido.</div>
    <div class="small">Pontuações guardadas no teu navegador.</div>
  </footer>
</div>

<script>
/*
  WorldGuessr - Single file HTML/JS implementation.
  - Detects online (navigator.onLine)
  - Uses Google Maps StreetView if online (script tag included below)
  - Offline: uses static panorama placeholder
  - Interactive minimap: simple equirectangular clickable SVG mapping x->lng, y->lat
  - Multiple modes, timer, scoring, localStorage history
*/
(function(){
  // -------------- CONFIG & DATA --------------
  const GOOGLE_API_KEY = "1nOufJJa0nejHzAtziBjWdpglY8EXChj"; // provided by user
  const UI_TEXTS = {
    pt: {
      start:"Começar", finish:"Acabar", stop:"Parar Jogo", rounds:"Rondas", time:"Tempo", points:"Pontos", mode:"Modo",
      summary:"Resumo do jogo", noonline:"Sem Internet — modo panorâmico", online:"Com Internet — Street View",
      best:"Melhor", avg:"Média", date:"Data"
    },
    en: {
      start:"Start", finish:"Finish", stop:"Stop Game", rounds:"Rounds", time:"Time", points:"Points", mode:"Mode",
      summary:"Game summary", noonline:"Offline — panoramic mode", online:"Online — Street View",
      best:"Best", avg:"Average", date:"Date"
    }
  };

  // A pool of diverse, mostly non-touristy coordinates.
  // Each entry: {lat, lng, name (optional), difficulty: 'easy'|'medium'|'hard', type:'city'|'village'}
  // NOTE: these are representative coordinates (small towns, villages, lesser-known localities)
  const LOCATIONS = [
    {lat:51.032,-2.286,name:"Bradford-on-Avon (UK)", difficulty:"easy", type:"town"},
    {lat:47.092, lng:15.432, name:"Hartberg (Austria)", difficulty:"easy", type:"town"},
    {lat:41.219, lng:20.388, name:"Gjirokastër outskirts (AL)", difficulty:"medium", type:"town"},
    {lat:46.080, lng:13.237, name:"Tolmin (Slovenia)", difficulty:"easy", type:"town"},
    {lat:43.345, lng:17.809, name:"Trebinje area (BA)", difficulty:"medium", type:"town"},
    {lat:44.200, lng:26.179, name:"Buftea (Romania)", difficulty:"easy", type:"town"},
    {lat:35.263, lng:24.804, name:"Pitsidia (Crete)", difficulty:"medium", type:"village"},
    {lat:40.727, lng:-8.658, name:"Figueira da Foz outskirts (PT)", difficulty:"easy", type:"city"},
    {lat:39.463, lng:-8.129, name:"Tomar region (PT)", difficulty:"easy", type:"town"},
    {lat:38.708, lng:-9.356, name:"Sesimbra outskirts (PT)", difficulty:"medium", type:"town"},
    {lat:41.178, lng:-8.611, name:"Guimarães outskirts (PT)", difficulty:"medium", type:"city"},
    {lat:42.283, lng:-8.728, name:"Ourense region (ES)", difficulty:"medium", type:"town"},
    {lat:44.429, lng:26.106, name:"Ploiești outskirts (RO)", difficulty:"medium", type:"town"},
    {lat:48.470, lng:7.751, name:"Wissembourg area (FR)", difficulty:"medium", type:"village"},
    {lat:45.922, lng:6.869, name:"Aiguebelle (FR)", difficulty:"hard", type:"village"},
    {lat:46.620, lng:1.901, name:"Saint-Amand-Montrond area (FR)", difficulty:"medium", type:"town"},
    {lat:50.123, lng:14.450, name:"Jičín region (CZ)", difficulty:"hard", type:"town"},
    {lat:52.060, lng:23.800, name:"Białowieża outskirts (PL)", difficulty:"hard", type:"village"},
    {lat:55.063, lng:82.937, name:"Novosibirsk countryside (RU)", difficulty:"hard", type:"village"},
    {lat:34.408, lng:35.841, name:"Baalbek region (LB)", difficulty:"hard", type:"town"},
    {lat:9.437, lng:-13.741, name:"Guinea-Bissau inland (GW)", difficulty:"hard", type:"village"},
    {lat:-33.783, lng:151.183, name:"Northern Beaches area (AU)", difficulty:"easy", type:"town"},
    {lat:-36.884, lng:174.740, name:"Auckland rural (NZ)", difficulty:"easy", type:"town"},
    {lat:-2.589, lng:139.658, name:"Papua (ID) village region", difficulty:"hard", type:"village"},
    {lat:14.640, lng:-61.001, name:"Dominica interior (DM)", difficulty:"hard", type:"village"},
    {lat:10.301, lng:-85.838, name:"Nicoya region (CR)", difficulty:"medium", type:"village"},
    {lat:6.878, lng:125.307, name:"Mindanao rural (PH)", difficulty:"hard", type:"village"},
    {lat:3.151, lng:101.694, name:"Kuala Selangor area (MY)", difficulty:"medium", type:"town"},
    {lat:23.352, lng:120.523, name:"Pingtung rural (TW)", difficulty:"hard", type:"village"},
    {lat:31.003, lng:121.420, name:"Kunshan suburbs (CN)", difficulty:"medium", type:"town"},
    {lat:19.652, lng:-99.140, name:"Toluca outskirts (MX)", difficulty:"medium", type:"town"},
    {lat:36.741, lng:3.050, name:"Kabylia rural (DZ)", difficulty:"hard", type:"village"},
    {lat:-34.905, lng:-56.190, name:"Canelones region (UY)", difficulty:"medium", type:"town"},
    {lat:-22.906, lng:-43.172, name:"Rio de Janeiro outskirts (BR)", difficulty:"medium", type:"city"},
    {lat:-26.204, lng:28.047, name:"Johannesburg periphery (ZA)", difficulty:"medium", type:"city"},
    {lat:8.506, lng:-13.234, name:"Sierra Leone inland (SL)", difficulty:"hard", type:"village"},
    {lat:52.377, lng:4.897, name:"Amsterdam outskirts (NL)", difficulty:"easy", type:"city"},
    {lat:60.391, lng:5.322, name:"Bergen rural (NO)", difficulty:"hard", type:"village"},
    {lat:59.329, lng:18.068, name:"Stockholm suburbs (SE)", difficulty:"easy", type:"city"},
    {lat:37.983, lng:23.727, name:"Athens periphery (GR)", difficulty:"medium", type:"city"},
    {lat:35.689, lng:139.691, name:"Tokyo suburbs (JP)", difficulty:"hard", type:"city"},
    {lat:25.276, lng:55.296, name:"Dubai outskirts (AE)", difficulty:"medium", type:"city"},
    {lat:48.856, lng:2.352, name:"Paris periphery (FR)", difficulty:"easy", type:"city"}
  ];

  // -------------- STATE --------------
  let state = {
    lang: localStorage.getItem("wg_lang") || "pt",
    online: navigator.onLine,
    mode: null, // easy|medium|hard|infinite|custom
    roundsTotal: 0,
    timePerRound: 60,
    currentRound: 0,
    roundActive: false,
    roundTimerId: null,
    remainingSeconds:0,
    score:0,
    usedIndices: new Set(),
    thisGameRounds: [], // {loc, guessLat, guessLng, distanceKm, points}
    panorama: null,
    googleLoaded:false,
    guessMarker: null,
    realMarker: null
  };

  // -------------- ELEMENTS --------------
  const el = id => document.getElementById(id);
  const startBtn = el('startBtn');
  const onlineState = el('onlineState');
  const uiLang = el('uiLang');
  const lblMode = el('statMode');
  const lblRounds = el('statRounds');
  const lblTime = el('statTime');
  const lblRemain = el('statRemain');
  const lblPoints = el('statPoints');
  const timerEl = el('timer');
  const finishBtn = el('finishBtn');
  const worldSvg = el('worldSvg');
  const markersGroup = el('markers');
  const minimapHint = el('minimapHint');
  const roundsList = el('roundsList');
  const endSummary = el('endSummary');
  const backToMenu = el('backToMenu');
  const btnEasy = el('btnEasy');
  const btnMedium = el('btnMedium');
  const btnHard = el('btnHard');
  const btnInfinite = el('btnInfinite');
  const btnCustom = el('btnCustom');
  const customArea = el('customArea');
  const customRounds = el('customRounds');
  const customSeconds = el('customSeconds');
  const difficultyFilter = el('difficultyFilter');
  const uiLangSelect = el('uiLang');
  const langPt = el('lang-pt');
  const langEn = el('lang-en');
  const startTitle = el('title');
  const statRounds = el('statRounds');
  const roundInfo = el('roundInfo');
  const locHint = el('locHint');
  const lastDistance = el('lastDistance');
  const lastPoints = el('lastPoints');
  const nextBtn = el('nextBtn');
  const revealBtn = el('revealBtn');
  const stopGameBtn = el('stopGameBtn');
  const scoresHistory = el('scoresHistory');
  const viewScoresBtn = el('viewScoresBtn');
  const resetScores = el('resetScores');
  const finishRevealBtn = el('revealBtn');

  // set initial language UI
  uiLangSelect.value = state.lang;

  // -------------- UTILITIES --------------
  function t(key){
    return UI_TEXTS[state.lang][key] || key;
  }

  function setOnlineFlag(){
    state.online = navigator.onLine;
    onlineState.textContent = state.online ? UI_TEXTS[state.lang].online : UI_TEXTS[state.lang].noonline;
    // Try to load Google Maps when online if not loaded
    if(state.online && !state.googleLoaded){
      loadGoogleMaps();
    }
  }
  window.addEventListener('online', setOnlineFlag);
  window.addEventListener('offline', setOnlineFlag);
  setOnlineFlag();

  function shuffle(arr){
    for(let i=arr.length-1;i>0;i--){
      const j = Math.floor(Math.random()*(i+1));
      [arr[i],arr[j]]=[arr[j],arr[i]];
    }
  }

  // haversine distance in kilometers
  function haversine(lat1,lon1,lat2,lon2){
    function toRad(v){return v*Math.PI/180;}
    const R=6371; // km
    const dLat=toRad(lat2-lat1);
    const dLon=toRad(lon2-lon1);
    const a=Math.sin(dLat/2)*Math.sin(dLat/2)+Math.cos(toRad(lat1))*Math.cos(toRad(lat2))*Math.sin(dLon/2)*Math.sin(dLon/2);
    const c=2*Math.atan2(Math.sqrt(a),Math.sqrt(1-a));
    return R*c;
  }

  // scoring: exponential decay (max 5000 points for exact)
  function scoreForDistance(km){
    // if within 0.5 km: near max.
    const max=5000;
    // scale parameter: distances larger than 20000 km -> 0
    const sigma = 2500; // tune: larger sigma -> slower drop
    const pts = Math.round(max * Math.exp(-0.5 * Math.pow(km / sigma, 2)));
    return pts;
  }

  function latlngFromSvgPoint(x,y){
    // svg viewBox 0..360 (x), 0..180 (y). Map x-> lon(-180..180), y-> lat(90..-90)
    const bbox = worldSvg.viewBox.baseVal;
    const vx = x * (bbox.width / worldSvg.clientWidth);
    const vy = y * (bbox.height / worldSvg.clientHeight);
    const lon = (vx / bbox.width) * 360 - 180;
    const lat = 90 - (vy / bbox.height) * 180;
    return {lat:lat, lng:lon};
  }

  function addMarkerToMinimap(lat,lng,cls, id){
    // convert lat/lng to svg coords
    const bbox = worldSvg.viewBox.baseVal;
    const x = ((lng + 180) / 360) * bbox.width;
    const y = ((90 - lat) / 180) * bbox.height;
    const existing = document.getElementById(id);
    if(existing) existing.remove();
    const g = document.createElementNS("http://www.w3.org/2000/svg","g");
    g.setAttribute("id", id);
    const circle = document.createElementNS("http://www.w3.org/2000/svg","circle");
    circle.setAttribute("cx", x);
    circle.setAttribute("cy", y);
    circle.setAttribute("r", 3.6);
    circle.setAttribute("class", cls === 'real' ? 'marker-real' : 'marker-guess');
    g.appendChild(circle);
    markersGroup.appendChild(g);
  }

  function clearMarkers(){ markersGroup.innerHTML=''; }

  // -------------- GAME FLOW --------------

  function prepareGame(mode){
    state.mode = mode;
    state.usedIndices = new Set();
    state.thisGameRounds = [];
    state.currentRound = 0;
    state.score = 0;
    state.googleLoaded = state.googleLoaded; // unchanged
    // configure rounds/time
    if(mode === 'easy'){ state.roundsTotal = 15; state.timePerRound = 120; }
    else if(mode === 'medium'){ state.roundsTotal = 15; state.timePerRound = 60; }
    else if(mode === 'hard'){ state.roundsTotal = 15; state.timePerRound = 30; }
    else if(mode === 'infinite'){ state.roundsTotal = Number.POSITIVE_INFINITY; state.timePerRound = 60; }
    else if(mode === 'custom'){ state.roundsTotal = parseInt(customRounds.value)||10; state.timePerRound = Math.min(180, parseInt(customSeconds.value)||60); }

    lblPoints.textContent = state.score;
    lblRounds.textContent = `${state.roundsTotal === Infinity ? '∞' : state.roundsTotal} rondas`;
    lblRemain.textContent = `${fmtTime(state.timePerRound)}`;
    roundInfo.textContent = `Ronda — 0/${state.roundsTotal === Infinity ? '∞' : state.roundsTotal}`;
    locHint.textContent = '';
    lastDistance.textContent = '—';
    lastPoints.textContent = '—';
    endSummary.style.display = 'none';
    roundsList.innerHTML = '';
    clearMarkers();
    // prepare list of indices filtered by difficulty
    state.availableIndices = LOCATIONS.map((l,i)=> ({i, d:l.difficulty, type:l.type})).filter(obj=>{
      const filter = difficultyFilter.value;
      if(filter === 'mixed') return true;
      if(filter === 'cities') return obj.type === 'city' || obj.type === 'town';
      if(filter === 'villages') return obj.type === 'village';
      return true;
    }).map(o => o.i);
    shuffle(state.availableIndices);
    // start first round
    nextRound();
  }

  function nextRound(){
    // end if reached total rounds
    if(state.roundsTotal !== Infinity && state.currentRound >= state.roundsTotal){
      endGame();
      return;
    }
    // pick next unused location
    if(state.availableIndices.length === 0){
      alert("Acabaram os locais disponíveis no pool para este filtro. O jogo vai terminar.");
      endGame();
      return;
    }
    const idx = state.availableIndices.pop();
    state.currentLocIndex = idx;
    state.currentRound++;
    state.roundActive = true;
    state.remainingSeconds = state.timePerRound;
    roundInfo.textContent = `Ronda — ${state.currentRound}/${state.roundsTotal === Infinity ? '∞' : state.roundsTotal}`;
    // show panorama (online: streetview, offline: static)
    const loc = LOCATIONS[idx];
    locHint.textContent = `${loc.name || '—'}`;

    if(state.online && state.googleLoaded && state.panorama){
      // show in StreetView
      try{
        state.panorama.setVisible(false);
        state.panorama.setPosition({lat:loc.lat, lng:loc.lng});
        // randomize POV a little
        state.panorama.setPov({heading: Math.floor(Math.random()*360), pitch:0});
        state.panorama.setVisible(true);
      }catch(e){
        // fallback to static
        showStaticPanorama(loc);
      }
    } else {
      showStaticPanorama(loc);
    }

    // reset minimap guess marker
    removeGuess();
    clearMarkers();
    // add real marker only after reveal
    state.roundTimerId && clearInterval(state.roundTimerId);
    startTimer();
  }

  function showStaticPanorama(loc){
    // static placeholder: simple background + label hidden until reveal
    const inner = document.getElementById('panoInner');
    inner.innerHTML = '';
    inner.style.display = 'flex';
    inner.style.alignItems = 'center';
    inner.style.justifyContent = 'center';
    inner.style.flexDirection = 'column';
    inner.style.color = '#9fbcd6';
    inner.style.fontSize = '14px';
    inner.style.padding = '18px';
    inner.style.textAlign = 'center';
    inner.style.backgroundImage = `radial-gradient(circle at 20% 20%, rgba(59,130,246,0.12), transparent 20%), linear-gradient(180deg, rgba(6,11,23,0.2), rgba(2,6,23,0.35)), url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="1200" height="600"><rect width="1200" height="600" fill="%23041a2b"/><g fill="%23063b5a" font-size="40" font-family="Arial"><text x="40" y="80">Panorâmica (offline)</text></g></svg>')`;
    inner.innerHTML = `<div style="font-size:20px; font-weight:700; color:#e6eef8; margin-bottom:6px;">Panorâmica</div>
      <div class="small" style="max-width:640px">${state.lang==='pt' ? 'Sem Internet — vista panorâmica estática. Não é possível mover-se.' : 'Offline — static panoramic view. Movement disabled.'}</div>`;
  }

  function removeGuess(){
    const g = document.getElementById('guessMarker');
    if(g) g.remove();
    state.guess = null;
  }

  function startTimer(){
    updateTimerDisplay();
    state.roundTimerId = setInterval(()=>{
      state.remainingSeconds--;
      updateTimerDisplay();
      if(state.remainingSeconds <= 0){
        // timeout
        clearInterval(state.roundTimerId);
        state.roundTimerId = null;
        state.roundActive = false;
        // automatically finalize with no guess => 0 points
        finalizeRoundNoGuess();
      }
    }, 1000);
  }

  function updateTimerDisplay(){
    timerEl.textContent = fmtTime(state.remainingSeconds);
    lblRemain.textContent = fmtTime(state.remainingSeconds);
  }

  function fmtTime(s){
    if(s === Infinity) return '∞';
    s = Math.max(0, Math.floor(s));
    const mm = String(Math.floor(s/60)).padStart(2,'0');
    const ss = String(s%60).padStart(2,'0');
    return `${mm}:${ss}`;
  }

  finishBtn.addEventListener('click', function(){
    if(!state.roundActive) return;
    // finalize round with current guess (or none)
    if(!state.guess){
      // no guess yet => treat as 0 (user clicked finish without guessing)
      finalizeRoundNoGuess();
    } else {
      finalizeRoundWithGuess(state.guess.lat, state.guess.lng);
    }
  });

  // minimap click handling
  worldSvg.addEventListener('click', function(ev){
    const rect = worldSvg.getBoundingClientRect();
    const x = ev.clientX - rect.left;
    const y = ev.clientY - rect.top;
    const bbox = worldSvg.viewBox.baseVal;
    // map x,y directly by proportion
    const lon = (x / rect.width) * 360 - 180;
    const lat = 90 - (y / rect.height) * 180;
    // store guess
    state.guess = {lat:lat, lng:lon};
    // place marker
    addMarkerToMinimap(lat, lng, 'guess', 'guessMarker');
  });

  function finalizeRoundNoGuess(){
    // add round record with 0 points
    const loc = LOCATIONS[state.currentLocIndex];
    const rec = {loc:loc, guessLat:null, guessLng:null, distanceKm:null, points:0};
    state.thisGameRounds.push(rec);
    state.roundActive = false;
    showRoundResult(rec);
    // proceed depending on infinite/others
  }

  function finalizeRoundWithGuess(guessLat, guessLng){
    const loc = LOCATIONS[state.currentLocIndex];
    const dist = haversine(loc.lat, loc.lng, guessLat, guessLng);
    const pts = scoreForDistance(dist);
    const rec = {loc:loc, guessLat, guessLng, distanceKm:dist, points:pts};
    state.score += pts;
    state.thisGameRounds.push(rec);
    state.roundActive = false;
    showRoundResult(rec);
    // mark real on minimap
    addMarkerToMinimap(loc.lat, loc.lng, 'real', 'realMarker');
    lblPoints.textContent = state.score;
    lastDistance.textContent = `${dist.toFixed(1)} km`;
    lastPoints.textContent = pts;
  }

  function showRoundResult(rec){
    // stop timer
    if(state.roundTimerId){ clearInterval(state.roundTimerId); state.roundTimerId = null; }
    // reveal real location: if Google loaded, pan to it briefly
    if(state.online && state.googleLoaded && state.panorama){
      try{
        state.panorama.setPosition({lat:rec.loc.lat, lng:rec.loc.lng});
        state.panorama.setVisible(true);
      }catch(e){ /* ignore */ }
    } else {
      // static: show label of real place
      const inner = document.getElementById('panoInner');
      inner.innerHTML = `<div style="text-align:center"><div style="font-weight:700">${rec.loc.name || 'Local real'}</div><div class="small">Coordenadas: ${rec.loc.lat.toFixed(4)}, ${rec.loc.lng.toFixed(4)}</div></div>`;
    }

    // show small overlay with results (we'll append to roundsList)
    const item = document.createElement('div');
    item.className = 'stat';
    const left = document.createElement('div');
    left.innerHTML = `<strong>${rec.loc.name || 'Local'}</strong><div class="small">${rec.loc.lat.toFixed(3)}, ${rec.loc.lng.toFixed(3)}</div>`;
    const right = document.createElement('div');
    right.style.textAlign = 'right';
    right.innerHTML = `<div>${rec.distanceKm===null? 'Sem palpite' : rec.distanceKm.toFixed(1)+' km'}</div><div style="font-weight:800">${rec.points} pts</div>`;
    item.appendChild(left); item.appendChild(right);
    roundsList.prepend(item);
    // after a short delay enable Next
  }

  nextBtn.addEventListener('click', function(){
    if(state.roundActive){
      if(!confirm((state.lang==='pt'?'Ronda ainda ativa. Deseja avançar para a próxima e perder pontos desta ronda?':'Round still active. Proceed to next and lose this round?'))){
        return;
      } else {
        // give zero points for this round
        finalizeRoundNoGuess();
      }
    }
    // if roundsTotal reached, end
    if(state.roundsTotal !== Infinity && state.currentRound >= state.roundsTotal){
      endGame();
      return;
    }
    nextRound();
  });

  revealBtn.addEventListener('click', function(){
    // reveal current real location without scoring (useful if user wants to see)
    const idx = state.currentLocIndex;
    const loc = LOCATIONS[idx];
    addMarkerToMinimap(loc.lat, loc.lng, 'real', 'realMarker');
    if(state.online && state.googleLoaded && state.panorama){
      try{ state.panorama.setPosition({lat:loc.lat, lng:loc.lng}); state.panorama.setVisible(true);}catch(e){}
    } else {
      showStaticPanorama(loc);
    }
  });

  stopGameBtn.addEventListener('click', function(){
    if(confirm((state.lang==='pt'?'Deseja terminar o jogo agora?':'Do you want to end the game now?'))){
      endGame();
    }
  });

  function endGame(){
    // stop timer
    if(state.roundTimerId){ clearInterval(state.roundTimerId); state.roundTimerId = null; }
    state.roundActive = false;
    // show summary
    endSummary.style.display = 'block';
    // compute stats
    const totalPoints = state.score;
    const totalRoundsPlayed = state.thisGameRounds.length;
    const avgDist = state.thisGameRounds.filter(r=>r.distanceKm!==null).reduce((a,b)=>a+b.distanceKm,0) / Math.max(1, state.thisGameRounds.filter(r=>r.distanceKm!==null).length);
    const best = state.thisGameRounds.reduce((m,r)=> Math.max(m, r.points||0),0);
    const summaryStats = el('summaryStats');
    summaryStats.innerHTML = `<div class="small">Pontuação total: <strong>${totalPoints}</strong></div>
      <div class="small">Rondas jogadas: <strong>${totalRoundsPlayed}</strong></div>
      <div class="small">Melhor ronda: <strong>${best}</strong></div>
      <div class="small">Média distância (onde houve palpite): <strong>${isNaN(avgDist)?'—': avgDist.toFixed(1)+' km'}</strong></div>`;

    // list rounds
    const list = el('roundsList'); list.innerHTML='';
    state.thisGameRounds.forEach((r,i)=>{
      const it = document.createElement('div');
      it.className='stat';
      it.innerHTML = `<div><strong>${i+1}. ${r.loc.name|| 'Local'}</strong><div class="small">${r.loc.lat.toFixed(3)}, ${r.loc.lng.toFixed(3)}</div></div>
        <div style="text-align:right"><div class="small">${r.distanceKm===null? 'Sem palpite' : r.distanceKm.toFixed(1)+' km'}</div><div style="font-weight:800">${r.points} pts</div></div>`;
      list.appendChild(it);
    });

    // save to localStorage history
    saveScore({
      date: new Date().toISOString(),
      totalPoints: totalPoints,
      rounds: totalRoundsPlayed,
      best: best,
      avgDistance: isNaN(avgDist)?null:avgDist
    });

    // refresh history panel
    renderScoresHistory();
  }

  backToMenu.addEventListener('click', function(){
    endSummary.style.display='none';
    // reset UI for new game
    state.score = 0;
    lblPoints.textContent = '0';
    roundsList.innerHTML = '';
    clearMarkers();
  });

  // -------------- STORAGE (history) --------------
  function saveScore(obj){
    const key = 'wg_scores_v1';
    const prev = JSON.parse(localStorage.getItem(key) || '[]');
    prev.unshift(obj);
    // limit history length to 50
    if(prev.length>50) prev.length=50;
    localStorage.setItem(key, JSON.stringify(prev));
  }
  function loadScores(){
    const key = 'wg_scores_v1';
    return JSON.parse(localStorage.getItem(key) || '[]');
  }
  function renderScoresHistory(){
    const arr = loadScores();
    scoresHistory.innerHTML = '';
    if(arr.length===0){ scoresHistory.innerHTML = `<div class="small">Sem histórico</div>`; return; }
    arr.slice(0,15).forEach(s=>{
      const d = new Date(s.date);
      const elNode = document.createElement('div');
      elNode.className = 'stat';
      elNode.innerHTML = `<div><strong>${state.lang==='pt'?'Jogo':'Game'} — ${d.toLocaleString()}</strong><div class="small">${state.lang==='pt'?'Rondas':'Rounds'}: ${s.rounds}</div></div>
        <div style="text-align:right"><div class="small">${state.lang==='pt'?'Média distância':'Avg dist'}: ${s.avgDistance?s.avgDistance.toFixed(1)+' km':'—'}</div><div style="font-weight:800">${s.totalPoints} pts</div></div>`;
      scoresHistory.appendChild(elNode);
    });
  }

  viewScoresBtn.addEventListener('click', function(){
    endSummary.style.display = 'block';
    renderScoresHistory();
    el('summaryStats').innerHTML = '<div class="small">Histórico de jogos guardados localmente.</div>';
    el('roundsList').innerHTML = '';
  });

  resetScores.addEventListener('click', function(){
    if(confirm((state.lang==='pt'?'Limpar histórico?':'Clear history?'))){
      localStorage.removeItem('wg_scores_v1');
      renderScoresHistory();
    }
  });

  renderScoresHistory();

  // -------------- UI wiring --------------
  btnEasy.addEventListener('click', ()=>{ prepareGame('easy'); });
  btnMedium.addEventListener('click', ()=>{ prepareGame('medium'); });
  btnHard.addEventListener('click', ()=>{ prepareGame('hard'); });
  btnInfinite.addEventListener('click', ()=>{ prepareGame('infinite'); });
  btnCustom.addEventListener('click', ()=>{ customArea.classList.toggle('hide'); prepareGame('custom'); });

  startBtn.addEventListener('click', function(){
    const modeSel = state.mode || 'easy';
    prepareGame(modeSel);
  });

  uiLangSelect.addEventListener('change', function(){
    state.lang = uiLangSelect.value;
    localStorage.setItem('wg_lang', state.lang);
    updateTexts();
  });
  langPt.addEventListener('click', ()=>{ uiLangSelect.value='pt'; uiLangSelect.dispatchEvent(new Event('change')); });
  langEn.addEventListener('click', ()=>{ uiLangSelect.value='en'; uiLangSelect.dispatchEvent(new Event('change')); });

  function updateTexts(){
    // update visible text according to language
    el('startBtn').textContent = state.lang==='pt' ? 'Começar' : 'Start';
    el('btnEasy').textContent = state.lang==='pt' ? 'Fácil / Easy' : 'Easy / Fácil';
    el('btnMedium').textContent = state.lang==='pt' ? 'Médio / Medium' : 'Medium / Médio';
    el('btnHard').textContent = state.lang==='pt' ? 'Difícil / Hard' : 'Hard / Difícil';
    el('btnInfinite').textContent = state.lang==='pt' ? 'Infinito / Infinite' : 'Infinite / Infinito';
    el('btnCustom').textContent = state.lang==='pt' ? 'Personalizado / Custom' : 'Custom / Personalizado';
    el('finishBtn').textContent = state.lang==='pt' ? 'Acabar' : 'Finish';
    el('stopGameBtn').textContent = state.lang==='pt' ? 'Parar Jogo / Stop' : 'Stop';
    document.querySelector('#subtitle').textContent = state.lang==='pt' ? 'Descobre onde no mundo estás — Street View / Panorâmico' : 'Find out where you are — Street View / Panoramic';
    setOnlineFlag();
    renderScoresHistory();
  }

  updateTexts();

  // -------------- GOOGLE MAPS LOADING & STREET VIEW --------------
  // We'll dynamically load the Google Maps JS API if online.
  function loadGoogleMaps(){
    if(state.googleLoaded) return;
    // create callback
    window._wg_init = function(){
      state.googleLoaded = true;
      initStreetView();
    };
    const s = document.createElement('script');
    s.src = `https://maps.googleapis.com/maps/api/js?key=${GOOGLE_API_KEY}&callback=_wg_init`;
    s.async = true; s.defer = true;
    s.onerror = function(){
      console.warn("Google Maps failed to load — continuing offline.");
      state.googleLoaded = false;
      setOnlineFlag();
    };
    document.head.appendChild(s);
  }
  // Try initial load if online
  if(state.online) loadGoogleMaps();

  function initStreetView(){
    // create panorama in div panoInner
    const elPano = document.getElementById('panoInner');
    elPano.innerHTML = ''; // clear static
    elPano.style.height='100%';
    elPano.style.padding='0';
    const panorama = new google.maps.StreetViewPanorama(elPano, {
      position: {lat:0, lng:0},
      pov: {heading: 165, pitch:0},
      visible:false,
      motionTracking:false,
      addressControl:false,
      showRoadLabels:false
    });
    state.panorama = panorama;
    // try to prevent initial errors when no valid position set
    panorama.addListener('pano_changed', function(){ /* ignore */});
  }

  // -------------- Misc helpers --------------
  // Expose minimal debug on console
  window.WG = {
    state, LOCATIONS, prepareGame, nextRound
  };

  // initial UI status
  setOnlineFlag();
})();
</script>
</body>
</html>
