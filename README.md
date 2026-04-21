# chirushi.github.io
[NEURAL_MAP v1.0.html](https://github.com/user-attachments/files/26926011/NEURAL_MAP.v1.0.html)
<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>NEURAL_MAP</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Share+Tech+Mono&family=VT323&display=swap');

  :root { --g:#00ff41; --gd:#00cc34; --bg:#000d03; }
  * { margin:0; padding:0; box-sizing:border-box; }

  body {
    background:var(--bg); color:var(--g);
    font-family:'Share Tech Mono',monospace;
    overflow:hidden; width:100vw; height:100vh;
    cursor:crosshair; user-select:none;
  }

  /* scanlines */
  body::before {
    content:''; position:fixed; inset:0;
    background:repeating-linear-gradient(0deg,transparent,transparent 3px,rgba(0,0,0,0.05) 3px,rgba(0,0,0,0.05) 4px);
    pointer-events:none; z-index:9999;
  }
  /* vignette */
  body::after {
    content:''; position:fixed; inset:0;
    background:radial-gradient(ellipse at center,transparent 40%,rgba(0,0,0,0.7) 100%);
    pointer-events:none; z-index:9997;
  }

  /* glitch overlay */
  #glitch-overlay { position:fixed; inset:0; pointer-events:none; z-index:9996; opacity:0; }
  #glitch-overlay .gl { position:absolute; left:0; right:0; background:rgba(0,255,65,0.07); }

  /* BREADCRUMB */
  #bc {
    position:fixed; top:16px; left:20px;
    font-size:10px; color:rgba(0,255,65,0.6);
    letter-spacing:3px; z-index:200;
  }
  .bc-link { cursor:pointer; transition:color .2s; }
  .bc-link:hover { color:var(--g); }
  .bc-sep { opacity:.4; margin:0 6px; }
  .bc-cur { color:rgba(0,255,65,0.85); }

  /* BACK */
  #back {
    position:fixed; top:12px; right:20px;
    font-size:10px; color:rgba(0,255,65,0.55);
    letter-spacing:3px; cursor:pointer; z-index:200;
    display:none; background:none; border:none;
    font-family:'Share Tech Mono',monospace; transition:color .2s;
  }
  #back:hover { color:var(--g); }
  #back.show { display:block; }

  /* RECENTER BUTTON */
  #recenter {
    position:fixed; bottom:22px; left:50%;
    transform:translateX(-50%);
    font-size:9px; color:rgba(0,255,65,0.55);
    letter-spacing:3px; cursor:pointer; z-index:200;
    background:rgba(0,13,3,0.85); border:1px solid rgba(0,255,65,0.25);
    font-family:'Share Tech Mono',monospace;
    padding:5px 14px; transition:all .2s;
  }
  #recenter:hover { color:var(--g); border-color:rgba(0,255,65,0.7); }

  /* CANVAS */
  #canvas { position:fixed; inset:0; overflow:hidden; }
  #map {
    position:absolute; width:5000px; height:5000px;
    transform-origin:0 0;
    transition:transform .8s cubic-bezier(.16,1,.3,1);
  }
  #map.instant { transition:none; }

  /* GRID */
  #grid-svg { position:absolute; inset:0; width:100%; height:100%; opacity:.13; pointer-events:none; }
  #svg-layer { position:absolute; inset:0; width:100%; height:100%; pointer-events:none; }

  /* REGION */
  .region {
    position:absolute; transform:translate(-50%,-50%);
    cursor:pointer; z-index:10; transition:opacity .5s;
  }
  .region-body {
    border:1px solid rgba(0,255,65,0.55);
    background:rgba(0,255,65,0.04);
    position:relative; display:flex;
    flex-direction:column; align-items:center; justify-content:center;
    gap:5px; transition:all .3s;
    animation:soft-flicker 9s infinite;
  }
  .region:not(.faded):hover .region-body {
    border-color:rgba(0,255,65,1);
    background:rgba(0,255,65,0.08);
    box-shadow:0 0 30px rgba(0,255,65,0.15),inset 0 0 16px rgba(0,255,65,0.04);
  }
  @keyframes soft-flicker { 0%,93%,100%{opacity:1} 94%{opacity:.5} 95%{opacity:1} 97%{opacity:.65} }

  .rc { position:absolute; width:7px; height:7px; border-color:rgba(0,255,65,0.75); border-style:solid; }
  .rc.tl{top:-1px;left:-1px;border-width:1px 0 0 1px;}
  .rc.tr{top:-1px;right:-1px;border-width:1px 1px 0 0;}
  .rc.bl{bottom:-1px;left:-1px;border-width:0 0 1px 1px;}
  .rc.br{bottom:-1px;right:-1px;border-width:0 1px 1px 0;}

  .region-name {
    font-family:'VT323',monospace; font-size:26px;
    letter-spacing:4px; color:rgba(0,255,65,0.92);
    text-align:center; line-height:1;
  }
  .region-id    { font-size:8px; color:rgba(0,255,65,0.42); letter-spacing:2px; }
  .region-count { font-size:8px; color:rgba(0,255,65,0.38); letter-spacing:1px; }

  .region:not(.faded):hover .region-name { animation:name-glitch .12s steps(1) .05s 2; }
  @keyframes name-glitch {
    0%  { clip-path:inset(20% 0 55% 0); transform:translateX(-3px); }
    50% { clip-path:inset(55% 0 10% 0); transform:translateX(3px); color:#fff; }
    100%{ clip-path:none; transform:translateX(0); }
  }

  .region.faded { opacity:.07; pointer-events:none; }
  .region.focused .region-body { border-color:rgba(0,255,65,.85); }

  /* SUBNODE */
  .subnode {
    position:absolute; transform:translate(-50%,-50%);
    cursor:pointer; z-index:20; opacity:0; pointer-events:none;
    transition:opacity .5s; text-align:center;
  }
  .subnode.visible { opacity:1; pointer-events:all; }
  .sub-circle {
    width:62px; height:62px; border-radius:50%;
    border:1px solid rgba(0,255,65,0.55);
    background:radial-gradient(circle at 35% 35%,rgba(0,255,65,0.12),rgba(0,13,3,.97));
    display:flex; flex-direction:column; align-items:center; justify-content:center;
    margin:0 auto 5px; transition:all .3s;
    box-shadow:0 0 10px rgba(0,255,65,0.07);
  }
  .subnode:hover .sub-circle {
    border-color:rgba(0,255,65,1);
    box-shadow:0 0 24px rgba(0,255,65,0.38);
    transform:scale(1.1);
  }
  .sub-lang {
    font-family:'VT323',monospace; font-size:15px;
    color:rgba(0,255,65,0.9); letter-spacing:1px; line-height:1.1;
    text-align:center; padding:0 4px;
    text-shadow:0 0 8px rgba(0,255,65,.5);
  }
  .sub-label { font-size:8px; color:rgba(0,255,65,0.48); letter-spacing:1px; max-width:74px; line-height:1.3; }

  /* PANEL */
  #panel {
    position:fixed; right:-370px; top:50%; transform:translateY(-50%);
    width:345px; background:rgba(0,8,2,.97);
    border-left:1px solid rgba(0,255,65,0.3);
    border-top:1px solid rgba(0,255,65,0.12);
    border-bottom:1px solid rgba(0,255,65,0.12);
    padding:26px 22px; z-index:300;
    transition:right .45s cubic-bezier(.16,1,.3,1);
    backdrop-filter:blur(12px);
  }
  #panel.open { right:0; }
  #pcl {
    position:absolute; top:11px; right:14px;
    background:none; border:none; color:rgba(0,255,65,0.38);
    font-family:'Share Tech Mono',monospace; font-size:13px; cursor:pointer; transition:color .2s;
  }
  #pcl:hover { color:rgba(0,255,65,.85); }
  .pt   { font-size:8px; color:rgba(0,255,65,0.38); letter-spacing:3px; margin-bottom:5px; }
  .pn   { font-family:'VT323',monospace; font-size:36px; letter-spacing:2px; color:rgba(0,255,65,.93); line-height:1; margin-bottom:2px; text-shadow:0 0 12px rgba(0,255,65,.3); }
  .ps   { font-size:9px; color:rgba(0,255,65,0.48); letter-spacing:2px; margin-bottom:14px; }
  .pd   { border:none; border-top:1px solid rgba(0,255,65,0.1); margin:12px 0; }
  .pdesc{ font-size:10px; line-height:1.9; color:rgba(0,255,65,.65); }
  .pstat{ display:flex; justify-content:space-between; font-size:9px; margin:5px 0; color:rgba(0,255,65,0.45); }
  .pstat span:last-child { color:rgba(0,255,65,.85); }

  /* LOADING */
  #loading {
    position:fixed; inset:0; background:var(--bg);
    display:flex; flex-direction:column; align-items:center; justify-content:center;
    z-index:9000; transition:opacity 1s;
  }
  #loading.done { opacity:0; pointer-events:none; }
  #lt {
    font-family:'VT323',monospace; font-size:30px; letter-spacing:8px;
    color:rgba(0,255,65,0.72); margin-bottom:36px;
    animation:lt-flicker 2.5s infinite;
  }
  @keyframes lt-flicker { 0%,88%,100%{opacity:.72} 89%{opacity:.15} 91%{opacity:.72} 93%{opacity:.3} 94%{opacity:.72} }
  #lb-wrap { width:180px; height:1px; background:rgba(0,255,65,0.12); margin-bottom:14px; }
  #lb { height:100%; background:rgba(0,255,65,0.72); width:0%; transition:width .1s; box-shadow:0 0 6px rgba(0,255,65,.4); }
  #ltext { font-size:9px; color:rgba(0,255,65,0.45); letter-spacing:3px; }

  /* MINIMAP */
  #mm {
    position:fixed; bottom:52px; right:18px;
    width:100px; height:80px;
    border:1px solid rgba(0,255,65,0.22);
    background:rgba(0,8,2,.92); z-index:100;
    overflow:hidden; opacity:.65; transition:opacity .2s;
  }
  #mm:hover { opacity:1; }
  #mm canvas { width:100%; height:100%; display:block; }
  #mm-vp { position:absolute; border:1px solid rgba(0,255,65,.6); pointer-events:none; }

  /* COORDS — bien visibles */
  #coords {
    position:fixed; bottom:22px; left:20px;
    font-size:9px; color:rgba(0,255,65,0.72);
    z-index:200; pointer-events:none; letter-spacing:2px;
    text-shadow:0 0 8px rgba(0,255,65,0.4);
  }

  @keyframes dash-anim { from{stroke-dashoffset:20} to{stroke-dashoffset:0} }
</style>
</head>
<body>

<div id="glitch-overlay"></div>

<div id="loading">
  <div id="lt">NEURAL_MAP</div>
  <div id="lb-wrap"><div id="lb"></div></div>
  <div id="ltext">initialisation...</div>
</div>

<div id="bc"><span class="bc-cur">NEURAL_MAP</span></div>
<button id="back">← retour</button>

<div id="canvas">
  <div id="map">
    <svg id="grid-svg" xmlns="http://www.w3.org/2000/svg">
      <defs>
        <pattern id="sg" width="50" height="50" patternUnits="userSpaceOnUse">
          <path d="M50 0L0 0 0 50" fill="none" stroke="#00ff41" stroke-width="0.25"/>
        </pattern>
        <pattern id="lg" width="250" height="250" patternUnits="userSpaceOnUse">
          <rect width="250" height="250" fill="url(#sg)"/>
          <path d="M250 0L0 0 0 250" fill="none" stroke="#00ff41" stroke-width="0.65"/>
        </pattern>
      </defs>
      <rect width="5000" height="5000" fill="url(#lg)"/>
    </svg>
    <svg id="svg-layer" xmlns="http://www.w3.org/2000/svg"></svg>
    <div id="nodes-root"></div>
  </div>
</div>

<div id="panel">
  <button id="pcl">✕</button>
  <div class="pt" id="pt">—</div>
  <div class="pn" id="pn">—</div>
  <div class="ps" id="ps">—</div>
  <hr class="pd">
  <div class="pdesc" id="pdesc">—</div>
  <hr class="pd">
  <div id="pstats"></div>
</div>

<div id="mm"><canvas id="mm-canvas"></canvas><div id="mm-vp"></div></div>
<div id="coords">x:0000 y:0000</div>
<button id="recenter">⊕ recentrer</button>

<script>
// ─── DISPOSITION EN FORME DE CERVEAU ───────────────────────────────────────
// Vue de dessus d'un cerveau : lobe frontal en haut, occipital en bas,
// hémisphères gauche/droite, corps calleux au centre.
// 11 systèmes au total.

const REGIONS = [

  // ── LOBE FRONTAL (haut) ── planification, décision, personnalité
  { id:'CON', name:'CONSCIENCE', x:2500, y:900, w:340, h:230,
    subs:[
      {id:'UI',    lang:'UI',       label:'interface utilisateur', x:-120,y:-80, desc:'La surface du moi. Ce que tu perçois de toi-même — une interface partielle sur un système bien plus vaste.', stats:{'COUVERTURE':'~5%','LATENCE':'500ms','FIABLE':'Non'}},
      {id:'FOCUS', lang:'focus()',  label:'attention',             x:0,   y:-100,desc:'setFocus() — le cerveau alloue ses ressources CPU vers un seul thread. Le reste passe en arrière-plan.', stats:{'THREADS':'1 (conscient)','BG':'∞','DURÉE':'~20min max'}},
      {id:'META',  lang:'meta',     label:'métacognition',         x:120, y:-80, desc:'Penser sa propre pensée. Un processus qui s\'observe lui-même — récursion de la conscience.', stats:{'TYPE':'Observer pattern','COÛT':'Élevé','UNIQUE':'Humain (?)'}},
      {id:'PROC',  lang:'PID',      label:'process manager',       x:-70, y:90,  desc:'Gestionnaire de processus. Décide quelles tâches sont prioritaires, lesquelles sont kill -9.', stats:{'SCHEDULER':'Préemptif','PRIOR.':'Variable','KILL':'Possible'}},
      {id:'EGO',   lang:'self',     label:'identité',              x:70,  y:90,  desc:'La variable self. Référence à soi-même dans chaque appel de méthode. Persiste tant que le système tourne.', stats:{'TYPE':'Référence','MUTABLE':'OUI','SCOPE':'Global'}},
    ]},

  // ── LOBE PRÉFRONTAL GAUCHE ── logique, langage
  { id:'LOG', name:'LOGIQUE', x:1300, y:1150, w:320, h:220,
    subs:[
      {id:'ALGO',  lang:'algo',     label:'algorithmes',           x:-110,y:-80, desc:'Heuristiques de résolution. Rapides mais imprécis — optimisés pour la survie, pas pour la vérité.', stats:{'TYPE':'Heuristique','PRÉCISION':'Approx.','VITESSE':'Rapide'}},
      {id:'BOOL',  lang:'bool',     label:'conditions if/else',    x:20,  y:-95, desc:'Toute décision réduite à vrai ou faux. Trop simpliste pour un monde en nuances.', stats:{'VALEURS':'true/false','OPS':'&&  ||  !','LIMITES':'Réel ≠ binaire'}},
      {id:'REC',   lang:'recur.',   label:'récursion',             x:120, y:-75, desc:'La pensée qui s\'appelle elle-même. Rumination = récursion sans case de base. Stack overflow.', stats:{'BASE_CASE':'Souvent absent','PROFONDEUR':'Infinie','RISQUE':'Boucle'}},
      {id:'CPLX',  lang:'O(n)',     label:'complexité',            x:-60, y:85,  desc:'Certains problèmes sont exponentiellement difficiles. Le cerveau approxime — précision sacrifiée pour vitesse.', stats:{'MEILLEUR':'O(1)','PIRE':'O(∞)','STRATÉGIE':'Approx.'}},
      {id:'ASM',   lang:'ASM',      label:'réflexes bas niveau',   x:70,  y:85,  desc:'L\'assembly du cerveau. Réactions primaires 200ms avant la pensée consciente.', stats:{'NIVEAU':'Hardware','LATENCE':'< 20ms','OVERRIDE':'Difficile'}},
    ]},

  // ── LOBE PRÉFRONTAL DROIT ── créativité, imagination
  { id:'CRE', name:'CRÉATIVITÉ', x:3700, y:1150, w:320, h:220,
    subs:[
      {id:'RND',   lang:'random()', label:'aléatoire',             x:-120,y:-80, desc:'La créativité naît du bruit. Connexions inattendues entre données disparates.', stats:{'SEED':'Inconnue','RANGE':'[0,∞[','VRAI_ALÉA':'OUI'}},
      {id:'GEN',   lang:'gen_AI',   label:'génération',            x:10,  y:-98, desc:'Comme les modèles génératifs — produit du nouveau à partir de patterns absorbés. LLM biologique.', stats:{'MODÈLE':'Bio-neural','PARAMS':'86 Mrd','TEMP.':'Variable'}},
      {id:'ITR',   lang:'loop',     label:'itération',             x:120, y:-75, desc:'for(;;){ créer(); évaluer(); améliorer(); } La deadline est le seul break.', stats:{'BREAK':'Deadline','OUTPUT':'Progressif','COND.':'Jamais'}},
      {id:'FRK',   lang:'fork()',   label:'bifurcations',          x:-65, y:85,  desc:'Chaque idée fork en chemins parallèles. Certains sont kill -9, d\'autres deviennent des projets.', stats:{'PARENT':'Idée source','ENFANTS':'Multiples','MERGE':'Rare'}},
      {id:'NOIS',  lang:'noise',    label:'bruit créatif',         x:75,  y:85,  desc:'Perlin noise neuronal. Aléatoire mais cohérent — la texture de l\'espace imaginaire.', stats:{'TYPE':'Perlin/Simplex','USAGE':'Procédural','OCTAVES':'Variable'}},
    ]},

  // ── CORPS CALLEUX ── connexion hémisphères, centre
  { id:'MEM', name:'MÉMOIRE', x:2500, y:1500, w:320, h:220,
    subs:[
      {id:'RAM',   lang:'RAM',      label:'mémoire vive',          x:-110,y:-80, desc:'Volatile et rapide. Tout ce que tu penses maintenant. Effacé au redémarrage — le sommeil.', stats:{'VITESSE':'~50 Go/s','VOLATILE':'OUI','TAILLE':'Limitée'}},
      {id:'SQL',   lang:'SQL',      label:'base de données',       x:10,  y:-98, desc:'SELECT * FROM souvenirs WHERE émotion="forte". Mémoire long terme indexée par contexte.', stats:{'TYPE':'Relationnelle','QUERY':'SELECT *','INDEX':'Temporel'}},
      {id:'ROM',   lang:'ROM',      label:'mémoire long terme',    x:115, y:-78, desc:'Lecture seule. Souvenirs gravés — enfance, traumatismes, premières fois.', stats:{'VOLATILE':'NON','ACCÈS':'Lent','DURÉE':'Vie entière'}},
      {id:'CACHE', lang:'CACHE',    label:'mémoire cache',         x:-65, y:85,  desc:'Accès ultra-rapide aux données récentes. Le cerveau précharge ce dont il aura besoin.', stats:{'NIVEAU':'L1/L2/L3','LATENCE':'< 1ms','POLITIQUE':'LRU'}},
      {id:'GC',    lang:'GC',       label:'garbage collector',     x:70,  y:85,  desc:'L\'oubli comme libération. Le cerveau efface pour faire de la place — parfois trop vite.', stats:{'TRIGGER':'Auto','RÉCUPÈRE':'RAM','OUBLI':'Inévitable'}},
    ]},

  // ── LOBE TEMPORAL GAUCHE ── langage
  { id:'LANG', name:'LANGAGE', x:950, y:1900, w:310, h:215,
    subs:[
      {id:'STR',   lang:'string',   label:'les mots',              x:-110,y:-78, desc:'Pensée encodée en string. Chaque mot = un token. Compression avec perte de ~40%.', stats:{'ENCODING':'UTF-8','PERTE':'~40%','TOKENS':'~170k FR'}},
      {id:'API',   lang:'API',      label:'communication',         x:10,  y:-96, desc:'Le langage comme API entre cerveaux. Doc imparfaite, endpoints instables.', stats:{'PROTOCOLE':'Acoustique','REST':'Non','AUTH':'Confiance'}},
      {id:'SYN',   lang:'syntax',   label:'grammaire',             x:110, y:-76, desc:'Le parser humain est contextuel et flou. Violations tolérées.', stats:{'STRICT':'Non','TOLÉRANCE':'Haute','PARSER':'Contextuel'}},
      {id:'META2', lang:'metadata', label:'sous-texte',            x:-60, y:82,  desc:'Ce qui n\'est pas dit. Headers cachés — ton, intention, histoire.', stats:{'VOLUME':'>50% msg','LISIBLE':'Parfois','CRITIQUE':'OUI'}},
      {id:'LOG2',  lang:'log()',    label:'monologue intérieur',   x:68,  y:82,  desc:'console.log() permanent. Flux de conscience filtré par l\'humeur.', stats:{'NIVEAU':'DEBUG/INFO','FILTRE':'Humeur','STOCKAGE':'RAM'}},
    ]},

  // ── LOBE TEMPORAL DROIT ── émotions, reconnaissance
  { id:'EMO', name:'ÉMOTIONS', x:4050, y:1900, w:310, h:215,
    subs:[
      {id:'EXC',   lang:'try{}',    label:'exceptions',            x:-115,y:-78, desc:'Les émotions sont des erreurs non gérées. Certaines sont catchées — d\'autres crashent tout.', stats:{'TYPE':'RuntimeException','CATCH':'Rarement','FATAL':'Parfois'}},
      {id:'EVT',   lang:'event',    label:'événements',            x:10,  y:-96, desc:'Chaque stimulus déclenche un listener. Le cerveau est event-driven.', stats:{'PATTERN':'Observer','ASYNC':'OUI','QUEUE':'Illimitée'}},
      {id:'BUG',   lang:'bug',      label:'biais cognitifs',       x:112, y:-76, desc:'Bugs dans le code évolutif. Réactions disproportionnées — héritage non patché.', stats:{'ORIGINE':'Évolution','PATCH':'Thérapie','STATUT':'OPEN'}},
      {id:'OVF',   lang:'overflow', label:'débordement',           x:-62, y:82,  desc:'Stack overflow émotionnel. Le système se fige ou s\'effondre. Burnout = crash total.', stats:{'SEUIL':'Variable','RESET':'Sommeil','CONSÉQUENCE':'Burnout'}},
      {id:'NULL',  lang:'null',     label:'vide / absence',        x:68,  y:82,  desc:'NullPointerException affective — dépression, dissociation.', stats:{'VALEUR':'null','DANGER':'OUI','TRAITEMENT':'Requis'}},
    ]},

  // ── LOBE PARIÉTAL ── perception, intégration sensorielle
  { id:'PERC', name:'PERCEPTION', x:2500, y:2200, w:330, h:220,
    subs:[
      {id:'SENS',  lang:'sensor',   label:'capteurs sensoriels',   x:-115,y:-80, desc:'Les drivers des 5 sens + proprioception. Le cerveau ne perçoit jamais la réalité directement — seulement ses propres données.', stats:{'CANAUX':'5+','BANDE PASS.':'Limitée','FILTRE':'Actif'}},
      {id:'BUFF',  lang:'buffer',   label:'tampon sensoriel',      x:10,  y:-98, desc:'Input buffer. Les données arrivent plus vite que le cerveau ne les traite — certaines sont droppées.', stats:{'TAILLE':'~500ms','DROP':'Fréquent','PRIORITÉ':'Survie'}},
      {id:'REND',  lang:'render()', label:'rendu du réel',         x:115, y:-80, desc:'Le cerveau reconstruit la réalité à partir de données parcellaires. Ce que tu vois est une interpolation — pas la vérité.', stats:{'FPS':'~60','LAG':'~80ms','FIDÉLITÉ':'Partielle'}},
      {id:'MAP2',  lang:'mapping',  label:'cartographie spatiale', x:-65, y:85,  desc:'GPS interne. Le cerveau maintient une carte de l\'espace — relative, déformée, subjective.', stats:{'TYPE':'Relative','PRÉCISION':'Variable','RESET':'Sommeil'}},
      {id:'PTRN',  lang:'pattern',  label:'reconnaissance',        x:72,  y:85,  desc:'Pattern matching permanent. Le cerveau cherche des formes connues dans tout — même dans le bruit.', stats:{'FAUX_POS.':'Fréquents','BIAIS':'OUI','VITESSE':'Rapide'}},
    ]},

  // ── LOBE OCCIPITAL ── traitement visuel, profond
  { id:'APPR', name:'APPRENTISSAGE', x:2500, y:2950, w:340, h:230,
    subs:[
      {id:'TRAIN', lang:'train()',  label:'entraînement',          x:-120,y:-85, desc:'Chaque expérience ajuste les poids synaptiques. Le cerveau est en train() permanent — jamais en deploy fixe.', stats:{'MODE':'Continu','ÉPOQUE':'Une vie','LOSS':'Erreur = apprentissage'}},
      {id:'BACK',  lang:'backprop',label:'rétropropagation',       x:10,  y:-105,desc:'L\'erreur remonte le réseau pour ajuster les connexions. L\'échec est le signal d\'entraînement le plus efficace.', stats:{'SIGNAL':'Erreur','GRADIENT':'Douleur/Plaisir','VITESSE':'Variable'}},
      {id:'OVERF', lang:'overfit',  label:'surapprentissage',      x:120, y:-85, desc:'Trop optimisé sur les expériences passées. Ne généralise plus — rigidité, préjugés, trauma.', stats:{'CAUSE':'Répétition','SYMPTÔME':'Rigidité','REMÈDE':'Nouveauté'}},
      {id:'TRANS', lang:'transfer', label:'transfert',             x:-70, y:90,  desc:'Réutiliser ce qu\'on a appris dans un domaine pour en maîtriser un autre. Le cerveau excelle à ça.', stats:{'EFFICACITÉ':'Haute','ANALOGIE':'OUI','DOMAINE':'Cross'}},
      {id:'PLAST', lang:'plastic',  label:'plasticité',            x:75,  y:90,  desc:'Neuroplasticité = hot-reload du cerveau. Le hardware se reconfigure selon l\'usage. Use it or lose it.', stats:{'PEAK':'Enfance','ADULTE':'Diminuée','STIMULÉE':'Par l\'effort'}},
    ]},

  // ── CERVELET ── coordination, automatismes
  { id:'AUTO', name:'AUTOMATISMES', x:1400, y:3350, w:330, h:225,
    subs:[
      {id:'CRON',  lang:'cron',     label:'tâches planifiées',     x:-115,y:-80, desc:'Respiration, digestion, rythme cardiaque. Des cron jobs biologiques qui tournent sans intervention consciente.', stats:{'INTERVAL':'Continu','MODIF.':'Possible (yoga...)','PRIORITÉ':'Haute'}},
      {id:'HABIT', lang:'habit.js', label:'habitudes',             x:10,  y:-98, desc:'Scripts comportementaux mis en cache. Une fois compilés, ils s\'exécutent sans charge CPU consciente.', stats:{'FORMATION':'~66 jours','COUT_CPU':'Minimal','DÉMONTER':'Difficile'}},
      {id:'MACRO', lang:'macro',    label:'macros motrices',       x:115, y:-78, desc:'Séquences motrices automatisées. Taper au clavier, conduire, jouer de la musique — des macros enregistrées.', stats:{'VITESSE':'Rapide','ERREUR':'Rare','STOCKAGE':'Cervelet'}},
      {id:'DMON',  lang:'daemon',   label:'processus de fond',     x:-62, y:84,  desc:'Daemons système. Régulation thermique, immunitaire, endocrinienne — tournent en arrière-plan sans UI.', stats:{'VISIBILITÉ':'Nulle','CRITIQUE':'OUI','KILL':'Mortel'}},
      {id:'SCHED', lang:'sched',    label:'ordonnanceur',          x:68,  y:84,  desc:'Le système nerveux autonome ordonnance les processus sans consulter la conscience. Plus rapide, plus fiable.', stats:{'TYPE':'Préemptif','MANUEL':'Difficile','FIABILITÉ':'Haute'}},
    ]},

  // ── AMYGDALE ── instinct, survie
  { id:'INST', name:'INSTINCT', x:3600, y:3350, w:310, h:215,
    subs:[
      {id:'ROOT',  lang:'root',     label:'accès root',            x:-110,y:-78, desc:'PID 1. Override tout. Survie, reproduction — sudo permanent sans authentification.', stats:{'PID':'0x0001','SUDO':'Permanent','KILL':'Impossible'}},
      {id:'IRQ',   lang:'IRQ',      label:'interruptions',         x:10,  y:-96, desc:'Interrupt Request. Menace détectée = tout s\'arrête. Le kernel reprend le contrôle en < 50ms.', stats:{'PRIORITÉ':'MAX','LATENCE':'< 50ms','OVERRIDE':'Tout'}},
      {id:'KERN',  lang:'kernel',   label:'noyau ancestral',       x:110, y:-76, desc:'3.8 milliards d\'années de commits. Aucun changelog, aucune doc.', stats:{'COMMITS':'∞','VERSION':'Évolution 1.0','DOCS':'Aucune'}},
      {id:'BIOS',  lang:'BIOS',     label:'firmware vital',        x:-60, y:82,  desc:'Avant tout OS — respiration, battement, réflexes. Firmware immuable actif dès la naissance.', stats:{'MODIF.':'Impossible','BOOT':'Naissance','TYPE':'Firmware'}},
      {id:'DRV',   lang:'driver',   label:'drivers corporels',     x:66,  y:82,  desc:'Interface bas niveau cerveau–corps. Pilotes de la douleur, faim, fatigue.', stats:{'NIVEAU':'Bas','LATENCE':'Variable','IGNORER':'Dangereux'}},
    ]},

  // ── HIPPOCAMPE ── rêves, consolidation
  { id:'DRM', name:'RÊVES', x:2500, y:3750, w:320, h:220,
    subs:[
      {id:'SIM',   lang:'sim()',    label:'simulation',            x:-110,y:-80, desc:'Bac à sable sans moteur physique. Testing de réalités alternatives — sans risque, sans règles.', stats:{'PHYSIQUE':'OFF','RÈGLES':'Aucune','FIDÉLITÉ':'Variable'}},
      {id:'MLLOC', lang:'malloc',   label:'consolidation mém.',    x:10,  y:-98, desc:'malloc/free de la mémoire journalière. Le rêve trie, compresse, supprime. Sans ça — saturation.', stats:{'TIMING':'Sommeil REM','DURÉE':'90min','CRITIQUE':'OUI'}},
      {id:'THRD',  lang:'thread',   label:'processus parallèles',  x:110, y:-78, desc:'Threads de fond inaccessibles le jour. Ils remontent la nuit — l\'inconscient en surface.', stats:{'ACCÈS':'Indirect','PRIORITÉ':'Basse','VISIBILITÉ':'Rêves'}},
      {id:'SEED',  lang:'seed',     label:'seed inconnue',         x:-62, y:85,  desc:'Même cerveau, seeds différentes. Terrain différent chaque nuit.', stats:{'DÉTERMIN.':'Non','REPRODUCT.':'Jamais','SOURCE':'Inconscient'}},
      {id:'IDLE',  lang:'idle',     label:'veille active',         x:68,  y:85,  desc:'CPU actif, UI inaccessible. Le travail continue sans que tu le saches.', stats:{'CPU':'Actif','UI':'OFF','PID':'0xDREAM'}},
    ]},
];

// Connexions — suivent la forme du cerveau
const RCONNS=[
  ['CON','LOG'],['CON','CRE'],['CON','MEM'],
  ['LOG','MEM'],['LOG','LANG'],['LOG','PERC'],
  ['CRE','MEM'],['CRE','EMO'],['CRE','PERC'],
  ['MEM','LANG'],['MEM','EMO'],['MEM','PERC'],['MEM','APPR'],
  ['LANG','AUTO'],['EMO','INST'],['EMO','APPR'],
  ['PERC','APPR'],['PERC','INST'],
  ['APPR','AUTO'],['APPR','DRM'],
  ['AUTO','DRM'],['INST','DRM'],
  ['LOG','AUTO'],['CRE','DRM'],
];

const MAP_W=5000,MAP_H=5000;
let scale=.28,panX=0,panY=0,dragging=false,sx,sy,spx,spy,curRegion=null,trans=false;

function getCenter(){
  // Centroïde de tous les nœuds
  const xs=REGIONS.map(r=>r.x), ys=REGIONS.map(r=>r.y);
  return { cx:(Math.min(...xs)+Math.max(...xs))/2, cy:(Math.min(...ys)+Math.max(...ys))/2 };
}

function recenter(){
  const vw=window.innerWidth,vh=window.innerHeight;
  const {cx,cy}=getCenter();
  // Calcul du scale pour que tout rentre
  const allX=REGIONS.map(r=>r.x), allY=REGIONS.map(r=>r.y);
  const minX=Math.min(...allX)-200, maxX=Math.max(...allX)+200;
  const minY=Math.min(...allY)-200, maxY=Math.max(...allY)+200;
  const ns=Math.min((vw/(maxX-minX)),(vh/(maxY-minY)))*.85;
  scale=Math.max(.18,Math.min(.55,ns));
  panX=vw/2-cx*scale; panY=vh/2-cy*scale;
  applyT(true); updateMM();
}

/* GLITCH */
function startGlitch(){
  const ov=document.getElementById('glitch-overlay');
  function go(){
    ov.innerHTML='';
    for(let i=0;i<3;i++){
      const l=document.createElement('div'); l.className='gl';
      l.style.top=Math.random()*100+'vh';
      l.style.height=(1+Math.random()*4)+'px';
      ov.appendChild(l);
    }
    ov.style.opacity='1';
    const map=document.getElementById('map');
    map.style.transform=`translate(${panX+(Math.random()-.5)*4}px,${panY+(Math.random()-.5)*3}px) scale(${scale})`;
    setTimeout(()=>{applyT(false);ov.style.opacity='0';},72);
    setTimeout(go,4000+Math.random()*13000);
  }
  setTimeout(go,3000+Math.random()*4000);
}

function init(){
  recenter();
  drawConns(); buildAll(); setupEvents(); startGlitch(); updateMM();
}

function drawConns(){
  const svg=document.getElementById('svg-layer');
  const rm={};
  REGIONS.forEach(r=>rm[r.id]=r);
  RCONNS.forEach(([a,b],i)=>{
    const rA=rm[a],rB=rm[b];
    const mx=(rA.x+rB.x)/2+(Math.random()-.5)*80;
    const my=(rA.y+rB.y)/2+(Math.random()-.5)*80;
    const p=document.createElementNS('http://www.w3.org/2000/svg','path');
    p.setAttribute('d',`M${rA.x} ${rA.y} Q${mx} ${my} ${rB.x} ${rB.y}`);
    p.setAttribute('stroke','rgba(0,220,52,0.22)');
    p.setAttribute('stroke-width','0.9');
    p.setAttribute('fill','none');
    p.setAttribute('stroke-dasharray','5 7');
    p.style.animation=`dash-anim ${4+i*.4}s linear infinite`;
    p.id=`rc-${i}`; svg.appendChild(p);
    const d=document.createElementNS('http://www.w3.org/2000/svg','circle');
    d.setAttribute('r','2.5'); d.setAttribute('fill','rgba(0,255,65,0.7)');
    d.setAttribute('filter','drop-shadow(0 0 3px rgba(0,255,65,0.9))');
    svg.appendChild(d); animPulse(d,p,i);
  });
}

function animPulse(dot,path,i){
  const dur=5000+i*500,delay=i*700; let t0=null;
  (function step(ts){
    if(!t0)t0=ts+delay;
    const el=(ts-t0)%dur; if(el<0){requestAnimationFrame(step);return;}
    const t=el/dur,pt=path.getPointAtLength(t*path.getTotalLength());
    dot.setAttribute('cx',pt.x); dot.setAttribute('cy',pt.y);
    dot.setAttribute('opacity',Math.sin(t*Math.PI)*.68);
    requestAnimationFrame(step);
  })(0);
}

function buildAll(){
  const root=document.getElementById('nodes-root'),svg=document.getElementById('svg-layer');
  REGIONS.forEach(r=>{
    const el=document.createElement('div');
    el.className='region'; el.id=`r-${r.id}`;
    el.style.left=r.x+'px'; el.style.top=r.y+'px';
    el.innerHTML=`<div class="region-body" style="width:${r.w}px;height:${r.h}px">
      <div class="rc tl"></div><div class="rc tr"></div><div class="rc bl"></div><div class="rc br"></div>
      <div class="region-name">${r.name}</div>
      <div class="region-id">${r.id}_SYS</div>
      <div class="region-count">${r.subs.length} sous-systèmes</div>
    </div>`;
    el.addEventListener('click',()=>{ if(!curRegion) zoomIn(r); });
    root.appendChild(el);
    r.subs.forEach(sub=>{
      const s=document.createElement('div');
      s.className='subnode'; s.id=`s-${sub.id}`;
      s.style.left=(r.x+sub.x)+'px'; s.style.top=(r.y+sub.y)+'px';
      s.innerHTML=`<div class="sub-circle"><div class="sub-lang">${sub.lang}</div></div><div class="sub-label">${sub.label}</div>`;
      s.addEventListener('click',e=>{ e.stopPropagation(); openPanel(r,sub); });
      root.appendChild(s);
      const ln=document.createElementNS('http://www.w3.org/2000/svg','line');
      ln.setAttribute('x1',r.x); ln.setAttribute('y1',r.y);
      ln.setAttribute('x2',r.x+sub.x); ln.setAttribute('y2',r.y+sub.y);
      ln.setAttribute('stroke','rgba(0,255,65,0.28)');
      ln.setAttribute('stroke-width','0.6');
      ln.setAttribute('opacity','0');
      ln.setAttribute('stroke-dasharray','3 5');
      ln.id=`sl-${sub.id}`; svg.appendChild(ln);
    });
  });
}

function zoomIn(r){
  if(trans)return; trans=true; curRegion=r;
  REGIONS.forEach(reg=>{ const el=document.getElementById(`r-${reg.id}`); el.classList.toggle('faded',reg.id!==r.id); el.classList.toggle('focused',reg.id===r.id); });
  document.querySelectorAll('[id^="rc-"]').forEach(el=>{el.style.transition='opacity .5s';el.style.opacity='0';});
  r.subs.forEach((sub,i)=>{ setTimeout(()=>{ document.getElementById(`s-${sub.id}`).classList.add('visible'); document.getElementById(`sl-${sub.id}`).setAttribute('opacity','1'); },180+i*55); });
  const vw=window.innerWidth,vh=window.innerHeight;
  const ns=Math.max(.85,Math.min(2.5,Math.min((vw-80)/(r.w+280),(vh-80)/(r.h+250))*.85));
  panX=vw/2-r.x*ns; panY=vh/2-r.y*ns; scale=ns; applyT(true);
  document.getElementById('back').classList.add('show'); setBc(r);
  setTimeout(()=>{trans=false;},900); updateMM();
}

function zoomOut(){
  if(trans||!curRegion)return; trans=true;
  const r=curRegion; curRegion=null;
  document.getElementById('panel').classList.remove('open');
  REGIONS.forEach(reg=>{ document.getElementById(`r-${reg.id}`).classList.remove('faded','focused'); });
  document.querySelectorAll('[id^="rc-"]').forEach(el=>{el.style.opacity='.22';});
  r.subs.forEach(sub=>{ document.getElementById(`s-${sub.id}`).classList.remove('visible'); document.getElementById(`sl-${sub.id}`).setAttribute('opacity','0'); });
  recenter();
  document.getElementById('back').classList.remove('show'); setBc(null);
  setTimeout(()=>{trans=false;},900); updateMM();
}

function setBc(r){ document.getElementById('bc').innerHTML=r?`<span class="bc-link" onclick="zoomOut()">NEURAL_MAP</span><span class="bc-sep">›</span><span class="bc-cur">${r.name}</span>`:`<span class="bc-cur">NEURAL_MAP</span>`; }

function openPanel(r,sub){
  document.getElementById('pt').textContent=`${r.id}_SYS / ${sub.id}`;
  document.getElementById('pn').textContent=sub.lang;
  document.getElementById('ps').textContent=sub.label;
  document.getElementById('pdesc').textContent=sub.desc;
  document.getElementById('pstats').innerHTML=Object.entries(sub.stats).map(([k,v])=>`<div class="pstat"><span>${k}</span><span>${v}</span></div>`).join('');
  document.getElementById('panel').classList.add('open');
}

document.getElementById('pcl').addEventListener('click',()=>document.getElementById('panel').classList.remove('open'));
document.getElementById('back').addEventListener('click',zoomOut);
document.getElementById('recenter').addEventListener('click',()=>{ if(!curRegion) recenter(); else zoomOut(); });

function applyT(anim){
  const map=document.getElementById('map');
  map.classList.toggle('instant',!anim);
  map.style.transform=`translate(${panX}px,${panY}px) scale(${scale})`;
  document.getElementById('coords').textContent=`x:${Math.round(-panX/scale)}  y:${Math.round(-panY/scale)}  ·  zoom:${Math.round(scale*100)}%`;
  updateMM();
}

function setupEvents(){
  const cv=document.getElementById('canvas');
  cv.addEventListener('mousedown',e=>{ if(e.target.closest('.region')||e.target.closest('.subnode')||e.target.closest('#panel'))return; dragging=true;sx=e.clientX;sy=e.clientY;spx=panX;spy=panY;cv.style.cursor='grabbing'; });
  window.addEventListener('mousemove',e=>{ if(!dragging)return; panX=spx+(e.clientX-sx);panY=spy+(e.clientY-sy);applyT(false); });
  window.addEventListener('mouseup',()=>{ dragging=false;document.getElementById('canvas').style.cursor='crosshair'; });
  cv.addEventListener('wheel',e=>{ e.preventDefault();const d=e.deltaY>0?-.04:.04;const ns=Math.max(.1,Math.min(4,scale+d));panX=e.clientX-(e.clientX-panX)*(ns/scale);panY=e.clientY-(e.clientY-panY)*(ns/scale);scale=ns;applyT(false); },{passive:false});
  let ld=0;
  cv.addEventListener('touchstart',e=>{ if(e.touches.length===1){dragging=true;sx=e.touches[0].clientX;sy=e.touches[0].clientY;spx=panX;spy=panY;}else if(e.touches.length===2)ld=Math.hypot(e.touches[0].clientX-e.touches[1].clientX,e.touches[0].clientY-e.touches[1].clientY); });
  cv.addEventListener('touchmove',e=>{ e.preventDefault();if(e.touches.length===1&&dragging){panX=spx+(e.touches[0].clientX-sx);panY=spy+(e.touches[0].clientY-sy);applyT(false);}else if(e.touches.length===2){const d=Math.hypot(e.touches[0].clientX-e.touches[1].clientX,e.touches[0].clientY-e.touches[1].clientY);scale=Math.max(.1,Math.min(4,scale+(d-ld)*.004));ld=d;applyT(false);} },{passive:false});
  cv.addEventListener('touchend',()=>{dragging=false;});
}

function updateMM(){
  const mm=document.getElementById('mm'),c=document.getElementById('mm-canvas');
  const ctx=c.getContext('2d');
  const W=mm.offsetWidth,H=mm.offsetHeight; c.width=W;c.height=H;
  const sx2=W/MAP_W,sy2=H/MAP_H;
  ctx.fillStyle='#000d03';ctx.fillRect(0,0,W,H);
  const rm={};REGIONS.forEach(r=>rm[r.id]=r);
  RCONNS.forEach(([a,b])=>{ const rA=rm[a],rB=rm[b];ctx.strokeStyle='rgba(0,200,50,0.25)';ctx.lineWidth=.4;ctx.beginPath();ctx.moveTo(rA.x*sx2,rA.y*sy2);ctx.lineTo(rB.x*sx2,rB.y*sy2);ctx.stroke(); });
  REGIONS.forEach(r=>{ ctx.fillStyle=curRegion?.id===r.id?'rgba(0,255,65,1)':'rgba(0,210,55,.65)';ctx.shadowColor='rgba(0,255,65,.6)';ctx.shadowBlur=3;ctx.fillRect(r.x*sx2-2.5,r.y*sy2-2.5,5,5);ctx.shadowBlur=0; });
  const vw=window.innerWidth,vh=window.innerHeight;
  const vp=document.getElementById('mm-vp');
  vp.style.left=Math.max(0,(-panX/scale)*sx2)+'px';
  vp.style.top=Math.max(0,(-panY/scale)*sy2)+'px';
  vp.style.width=Math.min((vw/scale)*sx2,W)+'px';
  vp.style.height=Math.min((vh/scale)*sy2,H)+'px';
}

const LL=['boot...','chargement des régions...','mapping des connexions...','injection des sous-systèmes...','en ligne.'];
let prog=0,li=0;
const lb=document.getElementById('lb'),lt2=document.getElementById('ltext');
(function runLoad(){
  if(prog>=100){setTimeout(()=>{document.getElementById('loading').classList.add('done');init();},500);return;}
  prog=Math.min(100,prog+Math.random()*20+5);
  lb.style.width=prog+'%';
  if(li<LL.length)lt2.textContent=LL[li++];
  setTimeout(runLoad,130+Math.random()*120);
})();
</script>
</body>
</html>
