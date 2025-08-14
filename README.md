<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>phik v0.8 — Comic & Storyboard Builder</title>
<style>
  :root{
    --bg:#0f1116; --fg:#e8eaf0; --muted:#9aa3b2;
    --accent:#7aa2f7; --accent2:#f7768e; --accent3:#9ece6a; --accent4:#e0af68; --accent5:#7dcfff;
    --panel:#151926; --panel-2:#1a1f2e; --panel-3:#0c0f16; --line:#2b3245;
    --warn:#ffcc66; --danger:#ff6b6b; --good:#48c774;
  }
  *{box-sizing:border-box;font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, "Helvetica Neue", Arial;}
  html,body{height:100%;margin:0;background:var(--bg);color:var(--fg);}
  #app{display:grid;grid-template-rows:auto 1fr auto; height:100%;}
  /* top bar */
  .topbar{display:flex;gap:.5rem;align-items:center;padding:.5rem;background:linear-gradient(180deg,var(--panel),var(--panel-2)); border-bottom:1px solid var(--line)}
  .brand{font-weight:700;letter-spacing:.5px;margin-right:.5rem}
  .spacer{flex:1}
  .btn{background:var(--panel-3);color:var(--fg);border:1px solid var(--line);padding:.45rem .6rem;border-radius:.4rem;cursor:pointer}
  .btn:hover{border-color:var(--accent)}
  .btn.small{padding:.25rem .45rem;font-size:.9rem}
  .btn.primary{background:var(--accent);border-color:transparent;color:#0b1020}
  .btn.warn{background:var(--warn);color:#000;border-color:#0002}
  .btn.danger{background:var(--danger);color:#fff;border-color:#0002}
  .seg{display:flex;border:1px solid var(--line);border-radius:.45rem;overflow:hidden}
  .seg .btn{border:0;border-right:1px solid var(--line);border-radius:0;background:transparent}
  .seg .btn:last-child{border-right:0}
  .kbd{background:#0008;border:1px solid var(--line);padding:0 .35rem;border-radius:.3rem;font-family:ui-monospace, SFMono-Regular, Menlo, Consolas, "Liberation Mono", monospace;font-size:.85rem}
  .pill{padding:.1rem .45rem;border:1px solid var(--line);border-radius:999px;background:#0006}
  .bubble{width:22px;height:22px;border-radius:50%;border:2px solid var(--line);cursor:pointer;position:relative}
  .bubble.active{outline:2px solid var(--accent)}
  .row{display:flex;gap:.5rem;align-items:center;flex-wrap:wrap}
  .hide{display:none!important}
  /* main area */
  .main{display:grid;grid-template-columns: 60px 1fr 320px; gap:.5rem; padding:.5rem}
  .panel{background:var(--panel);border:1px solid var(--line);border-radius:.6rem;overflow:hidden}
  .tools{padding:.5rem;display:flex;flex-direction:column;gap:.4rem}
  .tools .tbtn{display:flex;flex-direction:column;align-items:center;gap:.2rem;border:1px solid var(--line);background:var(--panel-2);padding:.35rem;border-radius:.4rem;cursor:pointer}
  .tools .tbtn.active{outline:2px solid var(--accent)}
  .canvasWrap{position:relative; overflow:hidden;
    background:
      conic-gradient(from 90deg at 1px 1px, #0000 90deg, #0002 0) 0 0/10px 10px;
  }
  canvas{display:block;background:transparent;width:100%;height:100%}
  .overlay{position:absolute;inset:0;pointer-events:none}
  .right{display:flex;flex-direction:column}
  .section{padding:.5rem;border-bottom:1px solid var(--line)}
  .section h3{margin:.2rem 0 .6rem 0;font-size:1rem}
  .section label{font-size:.85rem;color:var(--muted)}
  input[type="number"], input[type="text"], select, textarea{
    width:100%; background:#0006;color:var(--fg);border:1px solid var(--line);
    border-radius:.35rem;padding:.4rem .5rem;
  }
  textarea{min-height:72px;resize:vertical}
  input[type="range"]{width:100%}
  input[type="color"]{width:100%; height:2rem; padding:0; border:1px solid var(--line); background:#0000}
  .grid{display:grid;grid-template-columns:1fr 1fr;gap:.4rem}
  .stack{display:flex;flex-direction:column;gap:.35rem}
  /* layers list */
  #layerList{max-height:160px;overflow:auto;display:flex;flex-direction:column;gap:.25rem}
  .layerItem{display:flex;align-items:center;gap:.4rem;padding:.25rem .35rem;border:1px solid var(--line);border-radius:.35rem;background:#0003;cursor:pointer}
  .layerItem.active{outline:2px solid var(--accent)}
  .eye{width:18px;height:18px;border:1px solid var(--line);border-radius:.25rem;background:#0006;display:grid;place-items:center;font-size:12px}
  .layerItem input[type="text"]{flex:1;min-width:0;background:#0006;border:1px solid #0000}
  /* bottom nav */
  .bottombar{display:flex;gap:.5rem;align-items:center;padding:.5rem;background:linear-gradient(0deg,var(--panel),var(--panel-2)); border-top:1px solid var(--line)}
  /* floating UI toggle */
  #uiToggle{position:fixed;top:.5rem;left:.5rem;z-index:9999;background:var(--accent3);color:#0b1020;border:none;border-radius:999px; width:34px;height:34px;cursor:pointer;font-weight:900}
  .ui-hidden .topbar,.ui-hidden .main,.ui-hidden .bottombar,.ui-hidden .floaters{display:none!important}
  /* marquee + handles + page frame */
  .marquee{position:absolute;border:1px dashed var(--accent);background:rgba(122,162,247,.08);pointer-events:none}
  .handle{position:absolute;width:10px;height:10px;border:2px solid #000a;background:#fff;box-shadow:0 0 0 1px #0003;pointer-events:auto;cursor:nwse-resize}
  .handle[data-dir="n"], .handle[data-dir="s"]{cursor:ns-resize}
  .handle[data-dir="e"], .handle[data-dir="w"]{cursor:ew-resize}
  .pageFrame{position:absolute;border:2px dashed var(--accent5);pointer-events:none;box-shadow:0 0 0 9999px rgba(0,0,0,.25) inset}
  /* text editor overlay */
  #textEditor{position:absolute;min-width:40px;min-height:20px;background:#000a;border:1px dashed var(--accent);color:var(--fg);padding:.2rem .3rem;outline:none;white-space:pre-wrap;pointer-events:auto;display:none}
  .badge{font-size:.8rem;background:#0005;border:1px solid var(--line);border-radius:.35rem;padding:.1rem .4rem}
  .muted{color:var(--muted)}
  .note{font-size:.86rem;color:var(--muted)}
  .hr{height:1px;background:var(--line);margin:.25rem 0 .45rem}
  dialog{border:1px solid var(--line);background:#111827;color:var(--fg);border-radius:.6rem;max-width:min(680px,90vw);padding:1rem}
  /* error toast */
  #err{position:fixed;right:.75rem;bottom:.75rem;max-width:60ch;padding:.6rem .8rem;background:#2b1a1a;color:#ffd2d2;border:1px solid #5c2b2b;border-radius:.5rem;font-size:.9rem;display:none;z-index:99999;box-shadow:0 8px 24px #0005}
</style>
</head>
<body>
<button id="uiToggle" title="Hide/Show UI" type="button">?</button>
<div id="err"></div>

<div id="app">
  <!-- ===== TOPBAR ===== -->
  <div class="topbar">
    <div class="brand">??? <span id="appName">phik</span> <span class="pill" id="versionPill">v0.8</span> <span class="pill" id="modePill">PANEL MODE</span></div>

    <div class="seg" id="modeSeg">
      <button class="btn" data-mode="panel" type="button" title="Panel Mode">Panels</button>
      <button class="btn" data-mode="draw"  type="button" title="Draw Mode">Draw</button>
    </div>

    <div class="row">
      <button class="btn" id="btnNew"   type="button">New</button>
      <button class="btn" id="btnLoad"  type="button">Load</button>
      <button class="btn" id="btnSave"  type="button" title="Save Project"><span>Save</span> <span class="kbd" id="saveKbd">Ctrl+S</span></button>
      <button class="btn primary" id="btnExport" type="button" title="Export to standalone .html">Export HTML</button>
    </div>

    <div class="spacer"></div>

    <div class="row">
      <div class="seg">
        <button class="btn" id="btnUndo" type="button" title="Undo">? Undo</button>
        <button class="btn" id="btnRedo" type="button" title="Redo">? Redo</button>
      </div>
    </div>

    <!-- Always-visible palette -->
    <div class="row" id="paletteBar" title="Left-click to select, right-click to edit">
      <span>Palette:</span>
      <div id="palBubbles" class="row"></div>
      <span class="badge" id="activeColorBadge">—</span>
    </div>
  </div>

  <!-- ===== MAIN ===== -->
  <div class="main">
    <!-- TOOLS -->
    <div class="panel tools" id="toolPanel"></div>

    <!-- CANVAS -->
    <div class="panel canvasWrap" id="canvasPanel">
      <canvas id="canvas"></canvas>
      <div class="overlay" id="overlay"></div>
      <div class="marquee hide" id="marquee"></div>
      <div class="pageFrame" id="pageFrame"></div>
      <div id="textEditor" contenteditable="true"></div>
      <!-- handles get injected here -->
    </div>

    <!-- RIGHT SIDEBAR -->
    <div class="panel right">
      <div class="section">
        <h3 id="sidebarTitle">Panel Tools</h3>
        <div class="stack" id="dynamicControls"></div>
      </div>

      <div class="section">
        <h3>Page</h3>
        <div class="grid">
          <div class="stack">
            <label>Width</label>
            <input type="number" id="pageW" min="64" value="1024"/>
          </div>
          <div class="stack">
            <label>Height</label>
            <input type="number" id="pageH" min="64" value="576"/>
          </div>
        </div>
        <div class="grid" style="margin-top:.5rem">
          <div class="stack">
            <label>Page BG Color</label>
            <input type="color" id="pageBg" value="#000000">
          </div>
          <div class="stack">
            <label>Transparent BG</label>
            <select id="pageBgTransparent">
              <option value="true">Yes (transparent)</option>
              <option value="false">No (use color)</option>
            </select>
          </div>
        </div>
        <div class="stack" style="margin-top:.5rem">
          <label>Page Notes</label>
          <textarea id="pageNote" placeholder="Notes about this page..."></textarea>
        </div>
        <div class="row" style="margin-top:.5rem">
          <button class="btn" id="btnApplySize" type="button">Apply</button>
          <button class="btn" id="btnNewPage"   type="button">+ New Page</button>
        </div>
        <div class="row" style="margin-top:.5rem">
          <button class="btn" id="btnDupPage" type="button">Duplicate Page</button>
          <button class="btn danger" id="btnDelPage" type="button">Delete Page</button>
        </div>
        <div class="hr"></div>
        <div class="row wrap">
          <span class="note">Eraser uses transparent punch-through unless BG is set (then it erases to that color).</span>
        </div>
      </div>

      <div class="section">
        <h3>Selection</h3>
        <div class="row wrap" id="selButtons">
          <button class="btn" id="btnDeleteSel"    type="button">Delete</button>
          <button class="btn" id="btnCombineSel"   type="button">Combine</button>
          <button class="btn" id="btnToLayer"      type="button">Send to Layer…</button>
          <button class="btn" id="btnSaveAsPanel"  type="button">Save as Panel Template</button>
          <button class="btn" id="btnSaveAsStamp"  type="button">Save as Stamp</button>
        </div>
        <div class="row" style="margin-top:.4rem">
          <button class="btn small" id="btnSelectAll" type="button">Select All (visible)</button>
          <button class="btn small" id="btnDeselect"  type="button">Deselect</button>
        </div>
        <div class="stack" style="margin-top:.5rem">
          <label>Rotation</label>
          <input type="range" id="rotRange" min="-180" max="180" value="0"/>
        </div>
        <div id="textControls" class="stack" style="margin-top:.5rem; display:none">
          <div class="grid">
            <div class="stack"><label>Text</label><input type="text" id="textContent" placeholder="Edit text"></div>
            <div class="stack"><label>Size</label><input type="number" id="textSize" min="6" max="256" value="24"></div>
          </div>
          <span class="note">Tip: click a text object to edit inline. Shift+Enter for a newline.</span>
        </div>
        <div id="starControls" class="grid" style="margin-top:.5rem; display:none">
          <div class="stack"><label>Points</label><input id="uiStarPoints" type="number" min="3" max="24" value="5"/></div>
          <div class="stack"><label>Inner R</label><input id="uiStarInner" type="number" min="5" value="40"/></div>
          <div class="stack"><label>Outer R</label><input id="uiStarOuter" type="number" min="5" value="100"/></div>
        </div>
      </div>

      <div class="section">
        <h3>Stroke & Fill</h3>
        <div class="grid">
          <div class="stack">
            <label>Stroke</label>
            <input type="color" id="strokeColor" value="#e8eaf0"/>
          </div>
          <div class="stack">
            <label>Fill</label>
            <input type="color" id="fillColor" value="#00000000"/>
          </div>
          <div class="stack">
            <label>Width</label>
            <input type="range" id="strokeWidth" min="1" max="32" value="2"/>
          </div>
          <div class="stack">
            <label>Opacity</label>
            <input type="range" id="opacityRange" min="10" max="100" value="100"/>
          </div>
        </div>
        <div class="hr"></div>
        <div class="row">
          <span class="note">Active color:</span>
          <span class="badge" id="activeColorSmall">#e8eaf0</span>
        </div>
        <div class="hr"></div>
        <div class="row">
          <span class="note">Layer:</span>
          <div id="layerList" style="flex:1"></div>
        </div>
      </div>

      <div class="section">
        <h3>Library</h3>
        <div class="stack">
          <div class="row"><strong class="muted">Panel Templates</strong></div>
          <div id="libPanels" class="stack"></div>
          <div class="hr"></div>
          <div class="row"><strong class="muted">Stamps</strong></div>
          <div id="libStamps" class="stack"></div>
        </div>
      </div>

      <div class="section">
        <h3>Project</h3>
        <div class="row">
          <span class="note">Project:</span>
          <span class="badge" id="projName">—</span>
          <span class="note">Page:</span>
          <span class="badge" id="pageInfo">0 / 0</span>
        </div>
      </div>
    </div>
  </div>

  <!-- ===== BOTTOM BAR ===== -->
  <div class="bottombar">
    <div class="seg">
      <button class="btn" id="btnPrev" type="button">? Prev</button>
      <button class="btn" id="btnNext" type="button">Next ?</button>
    </div>
    <div class="spacer"></div>
    <div class="seg">
      <button class="btn" id="btnZoomOut" type="button">?</button>
      <button class="btn" id="btnZoomFit" type="button">Fit</button>
      <button class="btn" id="btnZoom100" type="button">100%</button>
      <button class="btn" id="btnZoomIn" type="button">+</button>
    </div>
    <div class="spacer"></div>
    <div class="row">
      <button class="btn" id="btnSettings" type="button">Settings</button>
      <span class="note">Tip: mid-mouse = pan. In Hatch tool, mid-drag sets angle with a live guide.</span>
    </div>
  </div>
</div>

<!-- SETTINGS (with Theme + Hotkeys) -->
<dialog id="dlgSettings">
  <form method="dialog">
    <h3>Settings</h3>
    <div class="section" style="border:0">
      <h4>Theme Colors</h4>
      <div class="grid">
        <div class="stack"><label>Accent 1</label><input type="color" id="th1" value="#7aa2f7"></div>
        <div class="stack"><label>Accent 2</label><input type="color" id="th2" value="#f7768e"></div>
        <div class="stack"><label>Accent 3</label><input type="color" id="th3" value="#9ece6a"></div>
        <div class="stack"><label>Accent 4</label><input type="color" id="th4" value="#e0af68"></div>
        <div class="stack"><label>Accent 5</label><input type="color" id="th5" value="#7dcfff"></div>
      </div>
    </div>
    <div class="section" style="border:0">
      <h4>Hotkeys</h4>
      <div class="stack">
        <label>Switch Mode</label>
        <input id="hkMode" type="text" placeholder="Key (e.g. Tab)" />
        <label>Save</label>
        <input id="hkSave" type="text" placeholder="Key (e.g. Ctrl+S)" />
        <label>Undo</label>
        <input id="hkUndo" type="text" placeholder="Ctrl+Z"/>
        <label>Redo</label>
        <input id="hkRedo" type="text" placeholder="Ctrl+Y / Ctrl+Shift+Z"/>
        <label>Tools (press in box then hit a key)</label>
        <div class="grid">
          <input id="hkBrush" type="text" placeholder="Brush"/>
          <input id="hkErase" type="text" placeholder="Erase"/>
          <input id="hkLine"  type="text" placeholder="Line"/>
          <input id="hkRect"  type="text" placeholder="Rect"/>
          <input id="hkCircle"type="text" placeholder="Circle"/>
          <input id="hkStar"  type="text" placeholder="Star"/>
          <input id="hkSelect"type="text" placeholder="Select"/>
          <input id="hkHatch" type="text" placeholder="Hatch"/>
          <input id="hkText"  type="text" placeholder="Text"/>
        </div>
        <div class="hr"></div>
        <div id="hkSummary" class="note"></div>
      </div>
    </div>
    <div class="hr"></div>
    <div class="row">
      <button class="btn" value="cancel" type="button" id="btnSettingsClose">Close</button>
      <button class="btn primary" id="btnSaveSettings" value="default" type="button">Save</button>
    </div>
  </form>
</dialog>

<!-- LOAD / NEW -->
<dialog id="dlgNew">
  <form method="dialog">
    <h3>New Project</h3>
    <div class="grid">
      <div class="stack"><label>Title</label><input type="text" id="newTitle" placeholder="phik project"></div>
      <div class="stack"><label>Default Width</label><input type="number" id="newW" value="1024"></div>
      <div class="stack"><label>Default Height</label><input type="number" id="newH" value="576"></div>
    </div>
    <div class="row" style="margin-top:.5rem">
      <button class="btn" value="cancel" type="button" id="btnNewClose">Cancel</button>
      <button class="btn primary" id="btnCreateProj" value="default" type="button">Create</button>
    </div>
  </form>
</dialog>

<dialog id="dlgLoad">
  <form method="dialog">
    <h3>Load Project</h3>
    <p class="note">Pick a “project.json” file exported by phik or one you previously saved.</p>
    <input type="file" id="loadFile" accept=".json"/>
    <div class="row" style="margin-top:.5rem">
      <button class="btn" value="cancel" type="button" id="btnLoadClose">Close</button>
      <button class="btn primary" id="btnDoLoad" value="default" type="button">Load</button>
    </div>
  </form>
</dialog>

<script>
(() => {
  /* ===== Helpers (safe bind + toast) ===== */
  const $ = s => document.querySelector(s);
  const $$ = s => Array.from(document.querySelectorAll(s));
  const bind = (sel, ev, fn) => { const el = $(sel); if(!el){ console.warn('Missing', sel); return; } el.addEventListener(ev, fn); };
  const set = (sel, prop, val) => { const el = $(sel); if(!el){ console.warn('Missing', sel); return; } el[prop] = val; };
  const toast = (msg)=>{ const e=$('#err'); e.textContent=msg; e.style.display='block'; clearTimeout(window.__errTo); window.__errTo=setTimeout(()=>e.style.display='none', 6000); };
  window.addEventListener('error', e=> toast('Script error: '+(e.message||e.error)));
  window.addEventListener('unhandledrejection', e=> toast('Unhandled: '+(e.reason?.message||e.reason)));

  /* ===== Utils ===== */
  const clamp=(v,lo,hi)=>Math.max(lo,Math.min(hi,v));
  const dist=(a,b)=>Math.hypot(a.x-b.x,a.y-b.y);
  const uid=()=>Math.random().toString(36).slice(2,9);
  const rgba=(hex,a=1)=>{ if(!hex) return `rgba(0,0,0,${a})`;
    if(hex.length===9){const r=parseInt(hex.slice(1,3),16),g=parseInt(hex.slice(3,5),16),b=parseInt(hex.slice(5,7),16),aa=parseInt(hex.slice(7,9),16)/255; return `rgba(${r},${g},${b},${(aa*a).toFixed(3)})`;}
    const r=parseInt(hex.slice(1,3),16),g=parseInt(hex.slice(3,5),16),b=parseInt(hex.slice(5,7),16); return `rgba(${r},${g},${b},${a})`; };
  const deepClone=o=>JSON.parse(JSON.stringify(o));
  const downloadFile=(name,text)=>{const blob=new Blob([text],{type:'text/plain'}); const a=document.createElement('a'); a.href=URL.createObjectURL(blob); a.download=name; a.click(); URL.revokeObjectURL(a.href); };
  const LS={set(k,v){localStorage.setItem(k,JSON.stringify(v));},get(k,d){try{const v=localStorage.getItem(k);return v?JSON.parse(v):d;}catch{ return d; }}};

  const TEMPLATES_KEY='PHIK_PANEL_TEMPLATES', STAMPS_KEY='PHIK_STAMPS', SETTINGS_KEY='PHIK_SETTINGS', THEME_KEY='PHIK_THEME', PALETTE_KEY='PHIK_PALETTE';

  /* ===== State ===== */
  const state = {
    mode:'panel',   // 'panel'|'draw'
    project:null,
    pageIndex:0,
    layer:0,
    tool:'panel-select',
    mouse:{x:0,y:0,down:false,ox:0,oy:0,button:0},
    camera:{x:0,y:0,scale:1},
    selection:new Set(),
    marquee:null,
    tmpPath:[],
    draw:{hatchAngle:45,hatchGap:8, hatchGuide:null},
    draggingSel:false, dragStartPositions:new Map(),
    panning:false, panStart:{x:0,y:0,cx:0,cy:0},
    resizing:null,      // {dir, startMouse, bbox0, items: Map(id,{...})}
    stroke:'#e8eaf0', fill:'#00000000', strokeWidth:2, opacity:100,
    currentColor:'#e8eaf0',
    templates: LS.get(TEMPLATES_KEY, []),
    stamps: LS.get(STAMPS_KEY, []),
    palette: LS.get(PALETTE_KEY, ['#e8eaf0','#f7768e','#9ece6a','#e0af68','#7aa2f7','#7dcfff']),
    settings: Object.assign({
      hotkeys:{ mode:'Tab', save:'Ctrl+S', undo:'Ctrl+Z', redo:'Ctrl+Y', brush:'B', erase:'E', line:'L', rect:'R', circle:'C', star:'S', select:'V', hatch:'H', text:'T' }
    }, LS.get(SETTINGS_KEY, {})),
    theme: Object.assign({ accent1:'#7aa2f7', accent2:'#f7768e', accent3:'#9ece6a', accent4:'#e0af68', accent5:'#7dcfff' }, LS.get(THEME_KEY, {})),
    history:{stack:[], idx:-1, lock:false},
    editingText:null // {id}
  };

  /* ===== Project schema ===== */
  function normalizeLayers(layers){
    // convert [0,1,2] -> [{id:0,name:'Layer 0',visible:true},...]
    if(Array.isArray(layers) && typeof layers[0]==='number'){
      return layers.map(id=>({id, name:`Layer ${id}`, visible:true}));
    }
    return layers && layers.length ? layers : [{id:0,name:'Layer 0',visible:true}];
  }
  function newProject(title,w=1024,h=576){
    return { title, defaultW:w, defaultH:h, pages:[newPage(w,h)], nextId:1, layers:normalizeLayers([0,1,2,3,4]) };
  }
  function newPage(w,h){ return { w,h, bgTransparent:true, bgColor:'#000000', note:'', objects:[] }; }
  function newObj(type,props={}){
    return Object.assign({
      id:uid(), type, kind: state.mode,
      x:0,y:0,w:0,h:0,r:0,
      points:[],
      stroke: state.stroke, fill: state.fill,
      strokeWidth: state.strokeWidth,
      opacity: state.opacity/100,
      layer: state.layer, group:null, meta:{} }, props);
  }
  const currPage = ()=> state.project.pages[state.pageIndex];
  const allObjects = ()=> currPage().objects;
  const getById = id => allObjects().find(o=>o.id===id);

  /* ===== History (Undo/Redo) ===== */
  function snapshot(){ return JSON.stringify(state.project); }
  function pushHistory(note=''){
    if(state.history.lock) return; // avoid recursive push during undo/redo
    const s=snapshot();
    // ignore if same as last
    if(state.history.stack[state.history.idx]===s) return;
    state.history.stack = state.history.stack.slice(0,state.history.idx+1);
    state.history.stack.push(s);
    state.history.idx++;
  }
  function undo(){
    if(state.history.idx<=0) return;
    state.history.lock=true;
    state.history.idx--;
    state.project = JSON.parse(state.history.stack[state.history.idx]);
    state.selection.clear(); state.resizing=null; state.draggingSel=false; state.drawing=null;
    refreshUIAfterProjectChange();
    state.history.lock=false;
  }
  function redo(){
    if(state.history.idx>=state.history.stack.length-1) return;
    state.history.lock=true;
    state.history.idx++;
    state.project = JSON.parse(state.history.stack[state.history.idx]);
    state.selection.clear(); state.resizing=null; state.draggingSel=false; state.drawing=null;
    refreshUIAfterProjectChange();
    state.history.lock=false;
  }

  /* ===== Canvas & render ===== */
  const canvas=$('#canvas'), ctx=canvas.getContext('2d'), overlay=$('#overlay'), marqueeEl=$('#marquee'), canvasPanel=$('#canvasPanel'), pageFrame=$('#pageFrame'), textEditor=$('#textEditor');
  function resizeCanvas(){
    const rect=canvasPanel.getBoundingClientRect();
    canvas.width = Math.max(200, Math.floor(rect.width));
    canvas.height= Math.max(200, Math.floor(rect.height));
    overlay.style.width=canvas.width+'px'; overlay.style.height=canvas.height+'px';
    drawPageFrame();
  }
  function drawPageFrame(){
    const p=currPage(); const tl=worldToScreen(0,0), br=worldToScreen(p.w,p.h);
    const x=Math.round(tl.x), y=Math.round(tl.y), w=Math.round(br.x-tl.x), h=Math.round(br.y-tl.y);
    pageFrame.style.left=x+'px'; pageFrame.style.top=y+'px'; pageFrame.style.width=w+'px'; pageFrame.style.height=h+'px';
  }
  const screenToWorld=(x,y)=>({x:x/state.camera.scale+state.camera.x, y:y/state.camera.scale+state.camera.y});
  const worldToScreen=(x,y)=>({x:(x-state.camera.x)*state.camera.scale, y:(y-state.camera.y)*state.camera.scale});

  function clearCanvas(){
    ctx.clearRect(0,0,canvas.width,canvas.height);
    // page background (just for preview inside page frame)
    const p=currPage();
    const a=worldToScreen(0,0), b=worldToScreen(p.w,p.h);
    if(!p.bgTransparent){
      ctx.save();
      ctx.fillStyle=p.bgColor||'#000000';
      ctx.fillRect(a.x,a.y,b.x-a.x,b.y-a.y);
      ctx.restore();
    }
    // draw page border
    ctx.save();
    ctx.strokeStyle='rgba(255,255,255,.4)';
    ctx.setLineDash([8,6]);
    ctx.lineWidth=1;
    ctx.strokeRect(a.x+0.5,a.y+0.5,(b.x-a.x)-1,(b.y-a.y)-1);
    ctx.setLineDash([]);
    ctx.restore();
  }

  function drawObject(o){
    if(!layerVisible(o.layer)) return;
    const s = state.camera.scale;
    const sc = worldToScreen(o.x, o.y);

    ctx.save();
    ctx.globalAlpha = (o.opacity ?? 1);

    // rotate about center (average center)
    const px = sc.x + (o.w*s)/2, py = sc.y + (o.h*s)/2;
    ctx.translate(px, py);
    ctx.rotate((o.r||0)*Math.PI/180);
    ctx.translate(-(o.w*s)/2, -(o.h*s)/2);

    if(o.type==='erase'){
      ctx.globalCompositeOperation='destination-out';
      ctx.lineWidth = Math.max(1, (o.strokeWidth||10)*s);
      ctx.strokeStyle = '#000';
      if(o.points.length>=2){
        ctx.beginPath();
        ctx.moveTo((o.points[0].x-o.x)*s,(o.points[0].y-o.y)*s);
        for(let i=1;i<o.points.length;i++){ const p=o.points[i]; ctx.lineTo((p.x-o.x)*s,(p.y-o.y)*s); }
        ctx.stroke();
      }
      ctx.restore(); return;
    }

    // normal drawing
    ctx.lineWidth = Math.max(1, (o.strokeWidth||1)*s);
    ctx.strokeStyle = rgba(o.stroke||'#e8eaf0', 1);
    ctx.fillStyle   = rgba(o.fill||'#00000000', 1);

    switch(o.type){
      case 'panel-rect': case 'rect':
        ctx.beginPath(); ctx.rect(0,0,o.w*s,o.h*s); if((o.fill??'')!=='#00000000') ctx.fill(); ctx.stroke(); break;
      case 'circle':
        ctx.beginPath(); ctx.ellipse((o.w*s)/2,(o.h*s)/2, Math.abs(o.w*s/2), Math.abs(o.h*s/2), 0, 0, Math.PI*2);
        if((o.fill??'')!=='#00000000') ctx.fill(); ctx.stroke(); break;
      case 'line':
        if(o.points.length>=2){ ctx.beginPath(); ctx.moveTo((o.points[0].x-o.x)*s,(o.points[0].y-o.y)*s); ctx.lineTo((o.points[1].x-o.x)*s,(o.points[1].y-o.y)*s); ctx.stroke(); }
        break;
      case 'path':
        if(o.points.length>=2){ ctx.beginPath(); ctx.moveTo((o.points[0].x-o.x)*s,(o.points[0].y-o.y)*s); for(let i=1;i<o.points.length;i++){ const p=o.points[i]; ctx.lineTo((p.x-o.x)*s,(p.y-o.y)*s);} if(o.meta.closed){ ctx.closePath(); if((o.fill??'')!=='#00000000') ctx.fill(); } ctx.stroke(); }
        break;
      case 'star':
        if(o.points.length>=2){ ctx.beginPath(); for(let i=0;i<o.points.length;i++){ const p=o.points[i]; const px=(p.x-o.x)*s, py=(p.y-o.y)*s; if(i===0) ctx.moveTo(px,py); else ctx.lineTo(px,py);} ctx.closePath(); if((o.fill??'')!=='#00000000') ctx.fill(); ctx.stroke(); }
        break;
      case 'group':
        ctx.setLineDash([6,6]); ctx.strokeRect(0,0,o.w*s,o.h*s); ctx.setLineDash([]); break;
      case 'text':
        ctx.fillStyle = o.stroke || state.currentColor;
        ctx.font = `${o.meta.size||24}px ${o.meta.family||'sans-serif'}`;
        ctx.textBaseline='top';
        (o.meta.text||'Text').split('\n').forEach((line,i)=> ctx.fillText(line, 0, i*(o.meta.size||24)));
        break;
    }
    ctx.restore();

    // selection box
    if(state.selection.has(o.id)){
      const p = worldToScreen(o.x,o.y);
      ctx.save(); ctx.setLineDash([6,6]); ctx.strokeStyle='#7aa2f7'; ctx.lineWidth=1;
      ctx.strokeRect(p.x+0.5,p.y+0.5,o.w*s-1,o.h*s-1); ctx.setLineDash([]); ctx.restore();
    }
  }

  function selectionBBox(){
    const items=[...state.selection].map(getById).filter(Boolean).filter(o=>o.kind===state.mode && layerVisible(o.layer));
    if(!items.length) return null;
    let minx=Infinity,miny=Infinity,maxx=-Infinity,maxy=-Infinity;
    for(const o of items){
      minx=Math.min(minx,o.x); miny=Math.min(miny,o.y);
      maxx=Math.max(maxx,o.x+o.w); maxy=Math.max(maxy,o.y+o.h);
    }
    return {x:minx,y:miny,w:maxx-minx,h:maxy-miny};
  }

  function drawHandles(){
    // remove old
    $$('#canvasPanel .handle').forEach(h=>h.remove());
    const bb=selectionBBox(); if(!bb) return;
    const points=[
      {dir:'nw', x:bb.x,           y:bb.y},
      {dir:'n',  x:bb.x+bb.w/2,    y:bb.y},
      {dir:'ne', x:bb.x+bb.w,      y:bb.y},
      {dir:'w',  x:bb.x,           y:bb.y+bb.h/2},
      {dir:'e',  x:bb.x+bb.w,      y:bb.y+bb.h/2},
      {dir:'sw', x:bb.x,           y:bb.y+bb.h},
      {dir:'s',  x:bb.x+bb.w/2,    y:bb.y+bb.h},
      {dir:'se', x:bb.x+bb.w,      y:bb.y+bb.h},
    ];
    for(const p of points){
      const sp=worldToScreen(p.x,p.y);
      const h=document.createElement('div'); h.className='handle'; h.dataset.dir=p.dir;
      h.style.left=(sp.x-5)+'px'; h.style.top=(sp.y-5)+'px';
      canvasPanel.appendChild(h);
      h.addEventListener('mousedown', e=>{
        e.stopPropagation(); e.preventDefault();
        startResize(p.dir);
      });
    }
  }

  function render(){
    if(!state.project) return requestAnimationFrame(render);
    clearCanvas();
    const objs = allObjects().slice().sort((a,b)=> a.layer-b.layer);
    for(const o of objs) drawObject(o);

    // hatch angle guide line (middle-drag in Hatch)
    if(state.draw.hatchGuide){
      const s=worldToScreen(state.draw.hatchGuide.a.x, state.draw.hatchGuide.a.y);
      const e=worldToScreen(state.draw.hatchGuide.b.x, state.draw.hatchGuide.b.y);
      ctx.save();
      ctx.strokeStyle=rgba(state.currentColor,.9);
      ctx.setLineDash([4,4]);
      ctx.lineWidth=2;
      ctx.beginPath(); ctx.moveTo(s.x,s.y); ctx.lineTo(e.x,e.y); ctx.stroke();
      ctx.setLineDash([]);
      ctx.restore();
    }

    // marquee
    if(state.marquee){
      const m=state.marquee;
      marqueeEl.style.left = Math.min(m.x,m.x+m.w)+'px';
      marqueeEl.style.top  = Math.min(m.y,m.y+m.h)+'px';
      marqueeEl.style.width= Math.abs(m.w)+'px';
      marqueeEl.style.height= Math.abs(m.h)+'px';
      marqueeEl.classList.remove('hide');
    } else marqueeEl.classList.add('hide');

    // handles (only in select)
    if(state.tool.endsWith('select') && state.selection.size){
      drawHandles();
    } else {
      $$('#canvasPanel .handle').forEach(h=>h.remove());
    }

    drawPageFrame();
    requestAnimationFrame(render);
  }

  /* ===== Hit / selection ===== */
  const aabb = o => ({x:o.x,y:o.y,w:o.w,h:o.h});
  const pointInAABB=(px,py,bb)=> px>=bb.x && py>=bb.y && px<=bb.x+bb.w && py<=bb.y+bb.h;
  const rectsIntersect = (A,B)=> !(B.x>A.x+A.w || B.x+B.w<A.x || B.y>A.y+A.h || B.y+B.h<A.y);

  function layerVisible(Lid){ const L = state.project.layers.find(l=>l.id===+Lid); return !L || L.visible!==false; }

  function hitTest(world, preferKind=state.mode){
    // Top-most in current mode & visible layer
    const objects = allObjects().slice().reverse();
    for(const o of objects){
      if(o.kind!==preferKind) continue;
      if(!layerVisible(o.layer)) continue;
      if(pointInAABB(world.x,world.y,aabb(o))) return o;
    }
    // fall back across modes
    for(const o of objects){
      if(!layerVisible(o.layer)) continue;
      if(pointInAABB(world.x,world.y,aabb(o))) return o;
    }
    return null;
  }

  function selectInMarquee_intersect(add=false){
    if(!state.marquee) return;
    const xm=Math.min(state.marquee.x,state.marquee.x+state.marquee.w);
    const ym=Math.min(state.marquee.y,state.marquee.y+state.marquee.h);
    const wm=Math.abs(state.marquee.w), hm=Math.abs(state.marquee.h);
    const topLeft=screenToWorld(xm,ym), bottomRight=screenToWorld(xm+wm,ym+hm);
    const box={x:topLeft.x,y:topLeft.y,w:bottomRight.x-topLeft.x,h:bottomRight.y-topLeft.y};
    if(!add) state.selection.clear();
    for(const o of allObjects()){
      if(o.kind!==state.mode) continue;
      if(!layerVisible(o.layer)) continue;
      const bb=aabb(o);
      if(rectsIntersect(bb, box)) state.selection.add(o.id);
    }
  }

  /* ===== Tools ===== */
  const TOOLS_PANEL = [
    {id:'panel-rect', label:'Panel Rect'},
    {id:'panel-star', label:'Star'},
    {id:'panel-line', label:'Line'},
    {id:'panel-select', label:'Select'}
  ];
  const TOOLS_DRAW = [
    {id:'draw-brush', label:'Brush'},
    {id:'draw-erase', label:'Erase'},
    {id:'draw-line',  label:'Line'},
    {id:'draw-rect',  label:'Rect'},
    {id:'draw-circle',label:'Circle'},
    {id:'draw-text',  label:'Text'},
    {id:'draw-hatch', label:'Hatch'},
    {id:'draw-select',label:'Select'}
  ];
  function toolLabel(id){ return ([...TOOLS_PANEL,...TOOLS_DRAW].find(t=>t.id===id)||{}).label||id; }

  function buildToolButtons(){
    const tpanel=$('#toolPanel'); tpanel.innerHTML='';
    const defs = (state.mode==='panel')? TOOLS_PANEL : TOOLS_DRAW;
    defs.forEach(d=>{
      const b=document.createElement('button'); b.type='button'; b.className='tbtn'+(state.tool===d.id?' active':''); b.textContent=d.label;
      b.addEventListener('click', ()=>{ state.tool=d.id; buildDynamicControls(); updateToolButtons(); });
      tpanel.appendChild(b);
    });
  }
  function updateToolButtons(){ $$('#toolPanel .tbtn').forEach(btn=> btn.classList.toggle('active', btn.textContent.trim().toLowerCase()===toolLabel(state.tool).toLowerCase())); }
  function setMode(mode){ state.mode=mode; $('#modePill').textContent = mode==='panel' ? 'PANEL MODE' : 'DRAW MODE'; $('#sidebarTitle').textContent = mode==='panel'? 'Panel Tools':'Draw Tools'; buildToolButtons(); buildDynamicControls(); state.selection.clear(); }

  function buildDynamicControls(){
    const wrap=$('#dynamicControls'); wrap.innerHTML='';
    const add=html=>wrap.insertAdjacentHTML('beforeend',html);

    if(state.mode==='panel'){
      if(state.tool==='panel-rect'){ add(`<div class="stack"><label>Panel Rect</label><p class="note">Drag to create rectangles. Use Select to move/scale.</p></div>`); }
      if(state.tool==='panel-star'){ add(`<div class="grid">
        <div class="stack"><label>Points</label><input id="starPoints" type="number" value="5" min="3" max="24"/></div>
        <div class="stack"><label>Inner Radius</label><input id="starInner" type="number" value="40" min="5" /></div>
        <div class="stack"><label>Outer Radius</label><input id="starOuter" type="number" value="100" min="5" /></div>
      </div><p class="note">Click an existing star to edit; click-drag to place a new star (scales while dragging).</p>`); }
      if(state.tool==='panel-line'){ add(`<p class="note">Drag to place a straight line. Hold Shift to snap angles.</p>`); }
      if(state.tool==='panel-select'){ add(`<p class="note">Click toggles selection. Drag to marquee (selects anything it touches). Drag inside to move; use handles to scale.</p>`); }
    } else {
      if(state.tool==='draw-brush' || state.tool==='draw-erase'){
        add(`<div class="grid">
          <div class="stack"><label>Size</label><input id="sizeBrush" type="range" min="1" max="64" value="${state.strokeWidth}"/></div>
          <div class="stack"><label>Round/Rect</label><select id="brushShape"><option>round</option><option>square</option></select></div>
        </div><p class="note">${state.tool==='draw-erase'?'Punches holes to transparency/BG.':'Freehand pen.'}</p>`);
      }
      if(state.tool==='draw-text'){
        add(`<div class="grid">
          <div class="stack"><label>Text</label><input id="toolTextContent" type="text" value="Text"></div>
          <div class="stack"><label>Size</label><input id="toolTextSize" type="number" min="6" max="256" value="24"></div>
        </div><p class="note">Click canvas to place text, then click it to edit inline.</p>`);
      }
      if(state.tool==='draw-hatch'){
        add(`<div class="grid">
          <div class="stack"><label>Angle</label><input id="hatchAngle" type="range" min="-90" max="90" value="${state.draw.hatchAngle}"/></div>
          <div class="stack"><label>Gap</label><input id="hatchGap" type="range" min="4" max="32" value="${state.draw.hatchGap}"/></div>
        </div><p class="note">Click-drag to shade. <strong>Middle-drag</strong> shows a guide and sets the hatch angle.</p>`);
      }
      if(state.tool==='draw-line' || state.tool==='draw-rect' || state.tool==='draw-circle'){ add(`<p class="note">Drag to place ${toolLabel(state.tool)}. Hold Shift to constrain.</p>`); }
      if(state.tool==='draw-select'){ add(`<p class="note">Click toggles selection. Drag to marquee (touch). Drag inside to move; handles to scale.</p>`); }
    }
  }

  /* ===== Theme & Palette ===== */
  function applyTheme(){
    const r=document.documentElement, t=state.theme;
    r.style.setProperty('--accent',t.accent1); r.style.setProperty('--accent2',t.accent2);
    r.style.setProperty('--accent3',t.accent3); r.style.setProperty('--accent4',t.accent4); r.style.setProperty('--accent5',t.accent5);
  }
  function buildPaletteBar(){
    const wrap=$('#palBubbles'); wrap.innerHTML='';
    state.palette.forEach((c,i)=>{
      const b=document.createElement('div'); b.className='bubble'+(c===state.currentColor?' active':''); b.style.background=c; b.title=`Color ${i+1}`;
      b.addEventListener('click', ()=>{ state.currentColor=c; state.stroke=c; $('#activeColorBadge').textContent=c; $('#activeColorSmall').textContent=c; buildPaletteBar(); });
      b.addEventListener('contextmenu', ev=>{
        ev.preventDefault();
        const inp=document.createElement('input'); inp.type='color'; inp.value=c; inp.style.position='fixed'; inp.style.left='-9999px'; document.body.appendChild(inp);
        inp.addEventListener('input', ()=>{ state.palette[i]=inp.value; LS.set(PALETTE_KEY, state.palette); buildPaletteBar(); });
        inp.addEventListener('change', ()=>{ document.body.removeChild(inp); });
        inp.click();
      });
      wrap.appendChild(b);
    });
    $('#activeColorBadge').textContent = state.currentColor;
    $('#activeColorSmall').textContent = state.currentColor;
    $('#strokeColor').value = state.currentColor;
  }

  /* ===== Layers UI ===== */
  function buildLayerList(){
    const list=$('#layerList'); list.innerHTML='';
    for(const L of state.project.layers){
      const row=document.createElement('div'); row.className='layerItem'+(state.layer===L.id?' active':''); row.dataset.id=L.id;
      const eye=document.createElement('div'); eye.className='eye'; eye.textContent = (L.visible===false)?'??':'???';
      const name=document.createElement('input'); name.type='text'; name.value=L.name||`Layer ${L.id}`;
      row.appendChild(eye); row.appendChild(name);
      list.appendChild(row);
      eye.addEventListener('click', e=>{ e.stopPropagation(); L.visible = !(L.visible!==false); buildLayerList(); });
      name.addEventListener('input', ()=>{ L.name=name.value; });
      row.addEventListener('click', ()=>{ state.layer=L.id; buildLayerList(); });
    }
  }
  function layerIdsVisible(){ return new Set(state.project.layers.filter(l=>l.visible!==false).map(l=>l.id)); }

  /* ===== Library ===== */
  function buildLibrary(){
    const P=$('#libPanels'); const S=$('#libStamps'); P.innerHTML=''; S.innerHTML='';
    const mk=(arr,wrap)=> arr.length? arr.map((t,i)=>{ const r=document.createElement('div'); r.className='row'; const b=document.createElement('button'); b.className='btn small'; b.textContent='Place'; const lbl=document.createElement('span'); lbl.className='note'; lbl.textContent=t.name; r.appendChild(b); r.appendChild(lbl); wrap.appendChild(r); b.addEventListener('click', ()=> placeTemplate(t.items)); }): (wrap.innerHTML='<span class="note">Empty — save some!</span>');
    mk(state.templates, P); mk(state.stamps, S);
  }
  function placeTemplate(items){
    const placed = items.map(o=>{
      const n=deepClone(o); n.id=uid(); currPage().objects.push(n); return n.id;
    });
    state.selection=new Set(placed);
    pushHistory('place template');
  }

  /* ===== Page/meta ===== */
  function refreshTopMeta(){
    set('#projName','textContent', state.project?.title || '—');
    set('#pageInfo','textContent', (state.pageIndex+1)+' / '+(state.project?.pages.length||0));
    $('#pageW').value=currPage().w; $('#pageH').value=currPage().h;
    $('#pageBg').value=currPage().bgColor||'#000000';
    $('#pageBgTransparent').value = currPage().bgTransparent?'true':'false';
    $('#pageNote').value = currPage().note||'';
    buildLayerList();
    buildLibrary();
  }

  /* ===== Resize / Scale ===== */
  function startResize(dir){
    const bb=selectionBBox(); if(!bb) return;
    const items=new Map();
    for(const id of state.selection){
      const o=getById(id); if(!o) continue;
      items.set(id, deepClone(o));
    }
    state.resizing={dir, bbox0:bb, items};
  }
  function applyResize(world){
    const R=state.resizing; if(!R) return;
    const bb0=R.bbox0;
    const anchors={
      'nw':{ax:bb0.x+bb0.w, ay:bb0.y+bb0.h},
      'n': {ax:bb0.x+bb0.w/2, ay:bb0.y+bb0.h},
      'ne':{ax:bb0.x, ay:bb0.y+bb0.h},
      'w': {ax:bb0.x+bb0.w, ay:bb0.y+bb0.h/2},
      'e': {ax:bb0.x, ay:bb0.y+bb0.h/2},
      'sw':{ax:bb0.x+bb0.w, ay:bb0.y},
      's': {ax:bb0.x+bb0.w/2, ay:bb0.y},
      'se':{ax:bb0.x, ay:bb0.y}
    };
    const {ax,ay}=anchors[R.dir];
    const nx=Math.min(ax,world.x), ny=Math.min(ay,world.y);
    const nw=Math.abs(world.x-ax), nh=Math.abs(world.y-ay);
    const sx = (nw||0.0001)/bb0.w, sy=(nh||0.0001)/bb0.h;

    for(const [id,base] of R.items){
      const o=getById(id); if(!o) continue;
      const relx = (base.x - bb0.x); const rely = (base.y - bb0.y);
      o.x = nx + relx*sx;
      o.y = ny + rely*sy;
      o.w = base.w * sx; o.h = base.h * sy;
      if(base.points?.length){
        o.points = base.points.map(p=>({ x: nx + (p.x - bb0.x)*sx, y: ny + (p.y - bb0.y)*sy }));
      }
    }
  }

  /* ===== Events ===== */
  // Settings + hotkeys UI
  bind('#btnSettings','click', ()=>{
    const HK=state.settings.hotkeys, t=state.theme;
    $('#hkMode').value=HK.mode; $('#hkSave').value=HK.save; $('#hkUndo').value=HK.undo; $('#hkRedo').value=HK.redo;
    $('#hkBrush').value=HK.brush; $('#hkErase').value=HK.erase; $('#hkLine').value=HK.line; $('#hkRect').value=HK.rect; $('#hkCircle').value=HK.circle; $('#hkStar').value=HK.star; $('#hkSelect').value=HK.select; $('#hkHatch').value=HK.hatch; $('#hkText').value=HK.text;
    $('#th1').value=t.accent1; $('#th2').value=t.accent2; $('#th3').value=t.accent3; $('#th4').value=t.accent4; $('#th5').value=t.accent5;
    updateHotkeySummary();
    $('#dlgSettings').showModal();
  });
  bind('#btnSettingsClose','click', ()=> $('#dlgSettings').close());
  const captureKeyIn = el => el.addEventListener('keydown', e=>{
    e.preventDefault(); const parts=[]; if(e.ctrlKey) parts.push('Ctrl'); if(e.altKey) parts.push('Alt'); if(e.shiftKey) parts.push('Shift');
    const key = e.key.length===1 ? e.key.toUpperCase() : e.key; if(!['Control','Alt','Shift'].includes(key)) parts.push(key); el.value=parts.join('+');
  });
  ['#hkMode','#hkSave','#hkUndo','#hkRedo','#hkBrush','#hkErase','#hkLine','#hkRect','#hkCircle','#hkStar','#hkSelect','#hkHatch','#hkText'].forEach(sel=>{ const el=$(sel); el && captureKeyIn(el); });
  function updateHotkeySummary(){
    const HK=state.settings.hotkeys;
    $('#hkSummary').innerHTML = `Mode: <b>${HK.mode}</b> • Save: <b>${HK.save}</b> • Undo: <b>${HK.undo}</b> • Redo: <b>${HK.redo}</b><br>
      Brush: <b>${HK.brush}</b> • Erase: <b>${HK.erase}</b> • Line: <b>${HK.line}</b> • Rect: <b>${HK.rect}</b> • Circle: <b>${HK.circle}</b> • Star: <b>${HK.star}</b> • Select: <b>${HK.select}</b> • Hatch: <b>${HK.hatch}</b> • Text: <b>${HK.text}</b>`;
  }
  bind('#btnSaveSettings','click', ()=>{
    const HK=state.settings.hotkeys, t=state.theme;
    Object.assign(HK, {
      mode:$('#hkMode').value||HK.mode, save:$('#hkSave').value||HK.save, undo:$('#hkUndo').value||HK.undo, redo:$('#hkRedo').value||HK.redo,
      brush:$('#hkBrush').value||HK.brush, erase:$('#hkErase').value||HK.erase, line:$('#hkLine').value||HK.line, rect:$('#hkRect').value||HK.rect, circle:$('#hkCircle').value||HK.circle, star:$('#hkStar').value||HK.star, select:$('#hkSelect').value||HK.select, hatch:$('#hkHatch').value||HK.hatch, text:$('#hkText').value||HK.text
    });
    state.theme.accent1=$('#th1').value; state.theme.accent2=$('#th2').value; state.theme.accent3=$('#th3').value; state.theme.accent4=$('#th4').value; state.theme.accent5=$('#th5').value;
    LS.set(SETTINGS_KEY,state.settings); LS.set(THEME_KEY,state.theme); applyTheme(); $('#saveKbd').textContent=HK.save; updateHotkeySummary(); $('#dlgSettings').close();
  });

  // project IO
  bind('#btnNew','click', ()=> $('#dlgNew').showModal());
  bind('#btnNewClose','click', ()=> $('#dlgNew').close());
  bind('#btnCreateProj','click', ()=>{
    const t=$('#newTitle').value.trim()||('phik-'+uid()); const w=+$('#newW').value||1024; const h=+$('#newH').value||576;
    state.project=newProject(t,w,h); state.pageIndex=0; state.layer=0; state.selection.clear();
    refreshUIAfterProjectChange(); pushHistory('new');
    $('#dlgNew').close();
  });
  bind('#btnLoad','click', ()=> $('#dlgLoad').showModal());
  bind('#btnLoadClose','click', ()=> $('#dlgLoad').close());
  bind('#btnDoLoad','click', async ()=>{
    const f=$('#loadFile').files[0]; if(!f) return alert('Pick a file');
    try{ const proj=JSON.parse(await f.text()); if(!proj.pages) throw new Error('Invalid project file'); proj.layers=normalizeLayers(proj.layers); state.project=proj; state.pageIndex=0; state.layer=proj.layers[0]?.id??0; state.selection.clear(); refreshUIAfterProjectChange(); pushHistory('load'); $('#dlgLoad').close(); }catch(err){ alert('Failed to load project: '+err.message); }
  });
  bind('#btnSave','click', ()=>{ if(!state.project) return alert('No project'); downloadFile((state.project.title||'phik')+'.project.json', JSON.stringify(state.project,null,2)); });
  bind('#btnExport','click', ()=>{ if(!state.project) return alert('No project'); const html=makeExportHTML(state.project); downloadFile((state.project.title||'comic')+'.html', html); });

  // undo/redo buttons
  bind('#btnUndo','click', undo);
  bind('#btnRedo','click', redo);

  // mode & tools
  bind('#uiToggle','click', ()=> document.body.classList.toggle('ui-hidden'));
  bind('#modeSeg [data-mode="panel"]','click', ()=> setMode('panel'));
  bind('#modeSeg [data-mode="draw"]','click',  ()=> setMode('draw'));

  // page controls
  bind('#btnNext','click', ()=>{ if(!state.project) return; state.pageIndex=(state.pageIndex+1)%state.project.pages.length; refreshUIAfterProjectChange(); });
  bind('#btnPrev','click', ()=>{ if(!state.project) return; state.pageIndex=(state.pageIndex-1+state.project.pages.length)%state.project.pages.length; refreshUIAfterProjectChange(); });
  bind('#btnApplySize','click', ()=>{ const w=+$('#pageW').value||currPage().w; const h=+$('#pageH').value||currPage().h; currPage().w=w; currPage().h=h; resizeCanvas(); pushHistory('page size'); });
  bind('#btnNewPage','click', ()=>{ const last=currPage(); state.project.pages.push(newPage(last.w,last.h)); state.pageIndex=state.project.pages.length-1; refreshUIAfterProjectChange(); pushHistory('new page'); });
  bind('#btnDupPage','click', ()=>{ const cp=currPage(); const dup=deepClone(cp); dup.objects.forEach(o=>o.id=uid()); state.project.pages.push(dup); state.pageIndex=state.project.pages.length-1; refreshUIAfterProjectChange(); pushHistory('dup page'); });
  bind('#btnDelPage','click', ()=>{ if(state.project.pages.length<=1) return alert('At least one page required.'); if(!confirm('Delete current page?')) return; state.project.pages.splice(state.pageIndex,1); state.pageIndex=Math.max(0,state.pageIndex-1); refreshUIAfterProjectChange(); pushHistory('del page'); });
  bind('#pageBg','input',  ()=>{ currPage().bgColor=$('#pageBg').value; });
  bind('#pageBgTransparent','change', ()=>{ currPage().bgTransparent = $('#pageBgTransparent').value==='true'; });
  bind('#pageNote','input', ()=>{ currPage().note=$('#pageNote').value; });

  // selection actions
  bind('#btnDeleteSel','click', ()=>{ const ids=[...state.selection]; currPage().objects = currPage().objects.filter(o=>!ids.includes(o.id)); state.selection.clear(); pushHistory('delete'); });
  bind('#btnCombineSel','click', ()=>{ if(state.selection.size<2) return; const items=[...state.selection].map(getById).filter(Boolean); const minx=Math.min(...items.map(o=>o.x)), miny=Math.min(...items.map(o=>o.y)); const maxx=Math.max(...items.map(o=>o.x+o.w)), maxy=Math.max(...items.map(o=>o.y+o.h)); const group=newObj('group',{x:minx,y:miny,w:maxx-minx,h:maxy-miny, meta:{children:items.map(i=>i.id)}}); currPage().objects.push(group); state.selection=new Set([group.id]); pushHistory('combine'); });
  bind('#btnToLayer','click', ()=>{ if(!state.selection.size) return; const L=prompt('Send selection to which layer id?', state.layer); if(L===null) return; for(const id of state.selection){ const o=getById(id); if(o) o.layer=+L; } pushHistory('to layer'); });
  bind('#btnSaveAsPanel','click', ()=>{ const items=[...state.selection].map(getById).filter(Boolean).filter(o=>o.kind==='panel'); if(!items.length) return alert('Select panels first'); const name=prompt('Template name?','tpl-'+uid()); if(!name) return; state.templates.push({name,items:items.map(deepClone)}); LS.set(TEMPLATES_KEY,state.templates); buildLibrary(); alert('Saved to panel_templates'); });
  bind('#btnSaveAsStamp','click', ()=>{ const items=[...state.selection].map(getById).filter(Boolean); if(!items.length) return alert('Select items first'); const name=prompt('Stamp name?','stamp-'+uid()); if(!name) return; state.stamps.push({name,items:items.map(deepClone)}); LS.set(STAMPS_KEY,state.stamps); buildLibrary(); alert('Saved to stamps'); });
  bind('#btnSelectAll','click', ()=>{ const vis=layerIdsVisible(); state.selection = new Set(allObjects().filter(o=>o.kind===state.mode && vis.has(o.layer)).map(o=>o.id)); });
  bind('#btnDeselect','click', ()=> state.selection.clear());
  bind('#rotRange','input', e=>{ for(const id of state.selection){ const o=getById(id); if(o) o.r=+e.target.value; } });

  // stroke/fill/live color
  bind('#strokeColor','input', e=>{ state.stroke = e.target.value; state.currentColor=e.target.value; buildPaletteBar(); });
  bind('#fillColor','input',   e=> state.fill   = e.target.value);
  bind('#strokeWidth','input', e=> state.strokeWidth = +e.target.value);
  bind('#opacityRange','input',e=> state.opacity = +e.target.value);

  // zoom
  function setZoom(scale, px=canvas.width/2, py=canvas.height/2){
    // zoom around screen point (px,py)
    const before=screenToWorld(px,py);
    state.camera.scale = clamp(scale, 0.1, 6);
    const after=screenToWorld(px,py);
    state.camera.x += (before.x-after.x);
    state.camera.y += (before.y-after.y);
  }
  bind('#btnZoomIn','click', ()=> setZoom(state.camera.scale*1.2));
  bind('#btnZoomOut','click', ()=> setZoom(state.camera.scale/1.2));
  bind('#btnZoom100','click', ()=> setZoom(1));
  bind('#btnZoomFit','click', ()=>{
    const p=currPage(), pad=40;
    const sx=(canvas.width-pad)/p.w, sy=(canvas.height-pad)/p.h;
    const s=clamp(Math.min(sx,sy),0.1,6);
    state.camera.scale=s;
    // center page
    const tl=worldToScreen(0,0), br=worldToScreen(p.w,p.h);
    const dx=((canvas.width - (br.x - tl.x))/2) - tl.x;
    const dy=((canvas.height - (br.y - tl.y))/2) - tl.y;
    state.camera.x -= dx/state.camera.scale;
    state.camera.y -= dy/state.camera.scale;
  });

  // canvas interactions
  function endInteractions(){
    state.mouse.down=false; state.draggingSel=false; state.resizing=null; state.marquee=null; state.panning=false;
    // commit history after interactions
    pushHistory('edit');
  }
  window.addEventListener('mouseup', endInteractions); // ensures release anywhere ends actions
  canvas.addEventListener('mousedown', e=>{
    if(!state.project) return;
    const rect=canvas.getBoundingClientRect();
    state.mouse.button=e.button;
    state.mouse.x=e.clientX-rect.left; state.mouse.y=e.clientY-rect.top; state.mouse.ox=state.mouse.x; state.mouse.oy=state.mouse.y; state.mouse.down=true;
    const world=screenToWorld(state.mouse.x,state.mouse.y);

    // middle button: pan (except hatch tool, where middle sets angle)
    if(e.button===1){
      if(!(state.tool==='draw-hatch')){
        state.panning=true; state.panStart={x:e.clientX,y:e.clientY,cx:state.camera.x,cy:state.camera.y}; return;
      } else {
        state.draw.hatchGuide={a:world,b:world}; return;
      }
    }

    // inline text editor commit if open
    if(state.editingText){ commitTextEdit(); }

    // handle resize grips
    if(state.tool.endsWith('select') && state.selection.size){
      const bb=selectionBBox();
      if(bb){
        const grips=[
          {dir:'nw',x:bb.x,y:bb.y},{dir:'n',x:bb.x+bb.w/2,y:bb.y},{dir:'ne',x:bb.x+bb.w,y:bb.y},
          {dir:'w',x:bb.x,y:bb.y+bb.h/2},{dir:'e',x:bb.x+bb.w,y:bb.y+bb.h/2},
          {dir:'sw',x:bb.x,y:bb.y+bb.h},{dir:'s',x:bb.x+bb.w/2,y:bb.y+bb.h},{dir:'se',x:bb.x+bb.w,y:bb.y+bb.h},
        ];
        for(const g of grips){
          const sp=worldToScreen(g.x,g.y);
          if(Math.abs(sp.x-state.mouse.x)<=8 && Math.abs(sp.y-state.mouse.y)<=8){ startResize(g.dir); return; }
        }
      }
    }

    if(state.mode==='panel'){
      if(state.tool==='panel-select'){
        const hit=hitTest(world,'panel');
        if(hit){
          // toggle selection (no Shift needed)
          if(state.selection.has(hit.id)) state.selection.delete(hit.id); else state.selection.add(hit.id);
          // start dragging if clicked inside selection but not handle
          state.draggingSel=true; state.dragStartPositions.clear();
          for(const id of state.selection){ const o=getById(id); if(o) state.dragStartPositions.set(id,{x:o.x,y:o.y}); }
          // inline text?
          maybeBeginTextEdit(hit, world);
        } else {
          // start marquee
          state.marquee={x:state.mouse.x,y:state.mouse.y,w:0,h:0};
          state.selection.clear();
        }
      }
      if(state.tool==='panel-rect'){
        const o=newObj('panel-rect',{x:world.x,y:world.y,w:1,h:1}); currPage().objects.push(o); state.selection=new Set([o.id]); state.drawing={id:o.id,start:world};
      }
      if(state.tool==='panel-line'){ state.tmpPath=[world]; }
      if(state.tool==='panel-star'){
        const hit=hitTest(world,'panel');
        if(hit?.type==='star'){ // edit existing
          state.selection=new Set([hit.id]);
          showStarControls(hit);
          // no drawing
        } else {
          // new star starts at click, scale while dragging
          const pts=+($('#starPoints')?.value||5), rin=+($('#starInner')?.value||10), rout=+($('#starOuter')?.value||10);
          const star=buildStar(world.x,world.y,pts,rin,rout);
          star.kind='panel'; star.meta={cx:world.x,cy:world.y,points:pts,inner:rin,outer:rout};
          currPage().objects.push(star);
          state.selection=new Set([star.id]);
          state.drawing={starId:star.id, center:{x:world.x,y:world.y}};
          showStarControls(star);
        }
      }
    } else {
      if(state.tool==='draw-select'){
        const hit=hitTest(world,'draw');
        if(hit){
          if(state.selection.has(hit.id)) state.selection.delete(hit.id); else state.selection.add(hit.id);
          state.draggingSel=true; state.dragStartPositions.clear(); for(const id of state.selection){ const o=getById(id); if(o) state.dragStartPositions.set(id,{x:o.x,y:o.y}); }
          maybeBeginTextEdit(hit, world);
        } else { state.marquee={x:state.mouse.x,y:state.mouse.y,w:0,h:0}; state.selection.clear(); }
      }
      if(state.tool==='draw-brush' || state.tool==='draw-erase'){
        const type = state.tool==='draw-erase' ? 'erase':'path';
        const p=newObj(type,{x:world.x,y:world.y, points:[world], stroke: state.tool==='draw-erase' ? '#000000' : state.currentColor, strokeWidth: +($('#sizeBrush')?.value || state.strokeWidth), meta:{closed:false}});
        p.fill='#00000000'; currPage().objects.push(p); state.drawing={id:p.id};
      }
      if(state.tool==='draw-line'){
        const o=newObj('line',{x:world.x,y:world.y, points:[world,world], stroke:state.currentColor}); currPage().objects.push(o); state.drawing={id:o.id,start:world};
      }
      if(state.tool==='draw-rect'){
        const o=newObj('rect',{x:world.x,y:world.y,w:1,h:1, stroke:state.currentColor, fill:state.fill}); currPage().objects.push(o); state.drawing={id:o.id,start:world};
      }
      if(state.tool==='draw-circle'){
        const o=newObj('circle',{x:world.x,y:world.y,w:1,h:1, stroke:state.currentColor, fill:state.fill}); currPage().objects.push(o); state.drawing={id:o.id,start:world};
      }
      if(state.tool==='draw-text'){
        const txt=($('#toolTextContent')?.value||'Text'); const size=+($('#toolTextSize')?.value||24);
        const o=newObj('text',{x:world.x,y:world.y,w:1,h:size, stroke:state.currentColor, meta:{text:txt,size:size,family:'sans-serif'}}); currPage().objects.push(o); state.selection=new Set([o.id]);
        // open inline editor right away
        beginTextEdit(o);
      }
      if(state.tool==='draw-hatch'){
        state.drawing={hatchStart:world};
      }
    }
  });

  canvas.addEventListener('mousemove', e=>{
    const rect=canvas.getBoundingClientRect();
    const sx=e.clientX-rect.left, sy=e.clientY-rect.top;
    state.mouse.x=sx; state.mouse.y=sy;
    const world=screenToWorld(state.mouse.x,state.mouse.y);
    if(!state.mouse.down) return;

    // panning
    if(state.panning){
      const dx=(e.clientX-state.panStart.x)/state.camera.scale;
      const dy=(e.clientY-state.panStart.y)/state.camera.scale;
      state.camera.x = state.panStart.cx - dx;
      state.camera.y = state.panStart.cy - dy;
      return;
    }
    // hatch angle middle guide
    if(state.draw.hatchGuide){
      state.draw.hatchGuide.b=world;
      return;
    }

    // resizing
    if(state.resizing){ applyResize(world); return; }

    if(state.mode==='panel'){
      if(state.tool==='panel-select' && state.marquee){
        state.marquee.w=state.mouse.x-state.mouse.ox; state.marquee.h=state.mouse.y-state.mouse.oy; selectInMarquee_intersect(false);
      }
      if(state.tool==='panel-select' && state.draggingSel){
        const dx=(state.mouse.x - state.mouse.ox)/state.camera.scale;
        const dy=(state.mouse.y - state.mouse.oy)/state.camera.scale;
        for(const id of state.selection){ const o=getById(id); if(!o) continue; const start=state.dragStartPositions.get(id); o.x=start.x+dx; o.y=start.y+dy; }
      }
      if(state.tool==='panel-rect' && state.drawing){
        const o=getById(state.drawing.id); if(!o) return;
        o.w = (world.x - state.drawing.start.x); o.h=(world.y - state.drawing.start.y);
        if(e.shiftKey){ const s=Math.max(Math.abs(o.w),Math.abs(o.h)); o.w=Math.sign(o.w)*s; o.h=Math.sign(o.h)*s; }
      }
      if(state.tool==='panel-line' && state.tmpPath.length){
        const last=state.tmpPath[state.tmpPath.length-1]; if(dist(last,world)>0.5) state.tmpPath.push(world);
      }
      if(state.tool==='panel-star' && state.drawing?.starId){
        const st=getById(state.drawing.starId); if(!st) return;
        const c=state.drawing.center;
        const rout=Math.hypot(world.x-c.x, world.y-c.y);
        const rin=rout*0.5;
        st.meta.outer=rout; st.meta.inner=rin;
        st.points = starPointsFromMeta(st.meta);
        // update bounding box
        const minx=Math.min(...st.points.map(p=>p.x)), miny=Math.min(...st.points.map(p=>p.y));
        const maxx=Math.max(...st.points.map(p=>p.x)), maxy=Math.max(...st.points.map(p=>p.y));
        st.x=minx; st.y=miny; st.w=maxx-minx; st.h=maxy-miny;
      }
    } else {
      if(state.tool==='draw-select' && state.marquee){
        state.marquee.w=state.mouse.x-state.mouse.ox; state.marquee.h=state.mouse.y-state.mouse.oy; selectInMarquee_intersect(false);
      }
      if(state.tool==='draw-select' && state.draggingSel){
        const dx=(state.mouse.x - state.mouse.ox)/state.camera.scale;
        const dy=(state.mouse.y - state.mouse.oy)/state.camera.scale;
        for(const id of state.selection){ const o=getById(id); if(!o) continue; const start=state.dragStartPositions.get(id); o.x=start.x+dx; o.y=start.y+dy; }
      }
      if((state.tool==='draw-brush' || state.tool==='draw-erase') && state.drawing){
        const o=getById(state.drawing.id); if(!o) return;
        const last=o.points[o.points.length-1]; if(dist(last,world)>0.25){ o.points.push(world); }
      }
      if(state.tool==='draw-line' && state.drawing){
        const o=getById(state.drawing.id); if(!o) return;
        o.points[1]=world;
        if(e.shiftKey){
          const dx=world.x - state.drawing.start.x, dy=world.y - state.drawing.start.y;
          const ang=Math.round(Math.atan2(dy,dx)/(Math.PI/4))*(Math.PI/4);
          const len=Math.hypot(dx,dy);
          o.points[1]={x: state.drawing.start.x + Math.cos(ang)*len, y: state.drawing.start.y + Math.sin(ang)*len};
        }
        o.x=Math.min(o.points[0].x,o.points[1].x);
        o.y=Math.min(o.points[0].y,o.points[1].y);
        o.w=Math.abs(o.points[1].x-o.points[0].x);
        o.h=Math.abs(o.points[1].y-o.points[0].y);
      }
      if((state.tool==='draw-rect' || state.tool==='draw-circle') && state.drawing){
        const o=getById(state.drawing.id); if(!o) return;
        o.w=(world.x - state.drawing.start.x); o.h=(world.y - state.drawing.start.y);
        if(e.shiftKey){ const s=Math.max(Math.abs(o.w),Math.abs(o.h)); o.w=Math.sign(o.w)*s; o.h=Math.sign(o.h)*s; }
      }
    }
  });

  canvas.addEventListener('mouseup', e=>{
    const world=screenToWorld(state.mouse.x,state.mouse.y);

    if(state.draw.hatchGuide){
      const a=state.draw.hatchGuide.a, b=state.draw.hatchGuide.b;
      const angRad=Math.atan2(b.y-a.y, b.x-a.x);
      state.draw.hatchAngle = Math.round(angRad*180/Math.PI);
      const inp=$('#hatchAngle'); if(inp) inp.value=state.draw.hatchAngle;
      state.draw.hatchGuide=null;
    }

    if(state.mode==='panel'){
      if(state.tool==='panel-line' && state.tmpPath.length>=2){
        const a=state.tmpPath[0], b=state.tmpPath[state.tmpPath.length-1];
        const o=newObj('path',{x:Math.min(a.x,b.x), y:Math.min(a.y,b.y), stroke:state.stroke, fill:'#00000000', points: state.tmpPath.slice(), meta:{closed:false}, kind:'panel'});
        o.kind='panel'; currPage().objects.push(o); state.tmpPath=[];
      }
      if(state.tool==='panel-rect' && state.drawing){
        const o=getById(state.drawing.id); if(o){ if(o.w<0){o.x+=o.w;o.w*=-1;} if(o.h<0){o.y+=o.h;o.h*=-1;} } state.drawing=null;
      }
      if(state.tool==='panel-star' && state.drawing?.starId){
        state.drawing=null;
      }
    } else {
      if(state.tool==='draw-hatch' && state.drawing?.hatchStart){
        const s=state.drawing.hatchStart, e2=screenToWorld(state.mouse.x,state.mouse.y);
        const ang=(+($('#hatchAngle')?.value||state.draw.hatchAngle))*Math.PI/180;
        const gap=+($('#hatchGap')?.value||state.draw.hatchGap);
        const minx=Math.min(s.x,e2.x), maxx=Math.max(s.x,e2.x), miny=Math.min(s.y,e2.y), maxy=Math.max(s.y,e2.y);
        const diag=Math.hypot(maxx-minx,maxy-miny);
        for(let d=-diag; d<diag; d+=gap){
          const x1=minx + d*Math.cos(ang), y1=miny + d*Math.sin(ang);
          const x2=x1 + (maxx-minx)*Math.sin(ang+Math.PI/2), y2=y1 - (maxx-minx)*Math.cos(ang+Math.PI/2);
          const line=newObj('line',{x:Math.min(x1,x2), y:Math.min(y1,y2), points:[{x:x1,y:y1},{x:x2,y:y2}], stroke: state.currentColor, fill:'#00000000', strokeWidth:1});
          currPage().objects.push(line);
        }
        state.drawing=null;
      }
      if(state.drawing && state.tool.startsWith('draw-')){
        const o=getById(state.drawing.id); if(o){ if(o.w<0){o.x+=o.w;o.w*=-1;} if(o.h<0){o.y+=o.h;o.h*=-1;} }
        state.drawing=null;
      }
    }
    // text controls toggle
    const onlyText = state.selection.size===1 && getById([...state.selection][0])?.type==='text';
    $('#textControls').style.display = onlyText ? 'flex':'none';
    if(onlyText){
      const t=getById([...state.selection][0]);
      $('#textContent').value=t.meta.text||'Text';
      $('#textSize').value=t.meta.size||24;
    }

    endInteractions();
  });

  // click empty space clears selection (when not dragging)
  canvas.addEventListener('click', e=>{
    if(state.mouse.button!==0) return;
    if(Math.hypot(state.mouse.x-state.mouse.ox, state.mouse.y-state.mouse.oy) < 2 && !state.resizing && !state.draggingSel){
      const world=screenToWorld(state.mouse.x,state.mouse.y);
      const hit=hitTest(world);
      if(!hit) state.selection.clear();
    }
  });

  // text edit via sidebar
  bind('#textContent','input', e=>{ if(state.selection.size!==1) return; const o=getById([...state.selection][0]); if(o?.type!=='text') return; o.meta.text=e.target.value; });
  bind('#textSize','input', e=>{ if(state.selection.size!==1) return; const o=getById([...state.selection][0]); if(o?.type!=='text') return; o.meta.size=+e.target.value; o.h=o.meta.size; });

  // hotkeys
  const keyString = e => { const parts=[]; if(e.ctrlKey) parts.push('Ctrl'); if(e.altKey) parts.push('Alt'); if(e.shiftKey) parts.push('Shift'); const key=e.key.length===1?e.key.toUpperCase():e.key; if(!['Control','Alt','Shift'].includes(key)) parts.push(key); return parts.join('+'); };
  addEventListener('keydown', e=>{
    if(document.querySelector('dialog[open]')) return;
    const K=keyString(e), HK=state.settings.hotkeys;
    if(K===HK.mode){ e.preventDefault(); setMode(state.mode==='panel'?'draw':'panel'); return; }
    if(K===HK.save){ e.preventDefault(); $('#btnSave').click(); return; }
    if(K===HK.undo){ e.preventDefault(); undo(); return; }
    if(K===HK.redo || (K==='Ctrl+Shift+Z')){ e.preventDefault(); redo(); return; }
    if(K===HK.brush){ setMode('draw'); state.tool='draw-brush'; buildDynamicControls(); updateToolButtons(); }
    if(K===HK.erase){ setMode('draw'); state.tool='draw-erase'; buildDynamicControls(); updateToolButtons(); }
    if(K===HK.line){ state.tool= state.mode==='panel'?'panel-line':'draw-line'; buildDynamicControls(); updateToolButtons(); }
    if(K===HK.rect){ state.tool= state.mode==='panel'?'panel-rect':'draw-rect'; buildDynamicControls(); updateToolButtons(); }
    if(K===HK.circle){ setMode('draw'); state.tool='draw-circle'; buildDynamicControls(); updateToolButtons(); }
    if(K===HK.star){ setMode('panel'); state.tool='panel-star'; buildDynamicControls(); updateToolButtons(); }
    if(K===HK.select){ state.tool= state.mode==='panel'?'panel-select':'draw-select'; buildDynamicControls(); updateToolButtons(); }
    if(K===HK.hatch){ setMode('draw'); state.tool='draw-hatch'; buildDynamicControls(); updateToolButtons(); }
    if(K===HK.text){ setMode('draw'); state.tool='draw-text'; buildDynamicControls(); updateToolButtons(); }
  });

  /* ===== Text inline editing ===== */
  function beginTextEdit(o){
    const s=worldToScreen(o.x,o.y);
    textEditor.style.left=s.x+'px'; textEditor.style.top=s.y+'px';
    textEditor.style.display='block';
    textEditor.style.minWidth=Math.max(40, o.w*state.camera.scale)+'px';
    textEditor.style.minHeight=Math.max(20, o.h*state.camera.scale)+'px';
    textEditor.textContent=o.meta.text||'';
    state.editingText={id:o.id};
    textEditor.focus();
  }
  function commitTextEdit(){
    if(!state.editingText) return;
    const o=getById(state.editingText.id); if(o){ o.meta.text=textEditor.textContent; o.h=o.meta.size||24; }
    textEditor.style.display='none'; textEditor.textContent='';
    state.editingText=null;
    pushHistory('text edit');
  }
  textEditor.addEventListener('keydown', e=>{
    if(e.key==='Enter' && !e.shiftKey){ e.preventDefault(); commitTextEdit(); }
  });
  textEditor.addEventListener('blur', commitTextEdit);
  function maybeBeginTextEdit(hit, world){
    if(hit?.type==='text'){ beginTextEdit(hit); }
  }

  /* ===== Stars ===== */
  function buildStar(cx,cy, points, innerR, outerR){
    const meta={cx,cy,points,inner:innerR,outer:outerR};
    const pts=starPointsFromMeta(meta);
    const minx=Math.min(...pts.map(p=>p.x)), miny=Math.min(...pts.map(p=>p.y));
    const maxx=Math.max(...pts.map(p=>p.x)), maxy=Math.max(...pts.map(p=>p.y));
    const o=newObj('star',{x:minx,y:miny,w:maxx-minx,h:maxy-miny, points:pts, stroke:state.currentColor, fill:state.fill, strokeWidth:state.strokeWidth, meta});
    return o;
  }
  function starPointsFromMeta(meta){
    const pts=[]; const step=Math.PI/meta.points;
    for(let i=0;i<2*meta.points;i++){ const r=(i%2===0)?meta.outer:meta.inner; const a=-Math.PI/2 + i*step; const x=meta.cx + r*Math.cos(a), y=meta.cy + r*Math.sin(a); pts.push({x,y}); }
    return pts;
  }
  function showStarControls(star){
    $('#starControls').style.display='grid';
    $('#uiStarPoints').value=star.meta.points||5;
    $('#uiStarInner').value=Math.round(star.meta.inner||40);
    $('#uiStarOuter').value=Math.round(star.meta.outer||100);
  }
  ['#uiStarPoints','#uiStarInner','#uiStarOuter'].forEach(sel=>{
    bind(sel,'input', ()=>{
      if(state.selection.size!==1) return; const st=getById([...state.selection][0]); if(!st || st.type!=='star') return;
      st.meta.points=+$('#uiStarPoints').value; st.meta.inner=+$('#uiStarInner').value; st.meta.outer=+$('#uiStarOuter').value;
      st.points=starPointsFromMeta(st.meta);
      const minx=Math.min(...st.points.map(p=>p.x)), miny=Math.min(...st.points.map(p=>p.y));
      const maxx=Math.max(...st.points.map(p=>p.x)), maxy=Math.max(...st.points.map(p=>p.y));
      st.x=minx; st.y=miny; st.w=maxx-minx; st.h=maxy-miny;
    });
  });

  /* ===== Export Viewer ===== */
  function makeExportHTML(project){
    const JSON_DATA=JSON.stringify(project);
    return `<!doctype html>
<html><head><meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1">
<title>${project.title} — Comic</title>
<style>
  body{margin:0;background:#0e1016;color:#eaeaf2;font-family:system-ui,sans-serif;height:100vh;display:grid;grid-template-rows:1fr auto}
  #wrap{display:flex;align-items:center;justify-content:center;padding:1rem}
  canvas{background:transparent;box-shadow:0 10px 40px #0008;border:1px solid #101828}
  .bar{display:flex;gap:.5rem;align-items:center;justify-content:center;padding:.5rem;background:#101423;border-top:1px solid #1a2235}
  .btn{background:#1b2235;color:#eaeaf2;border:1px solid #2a3551;border-radius:.45rem;padding:.45rem .7rem;cursor:pointer}
  .badge{background:#0b0f1a;border:1px solid #1a2235;border-radius:.35rem;padding:.2rem .5rem}
</style></head><body>
<div id="wrap"><canvas id="c"></canvas></div>
<div class="bar"><button class="btn" id="prev">? Prev</button><span class="badge" id="info"></span><button class="btn" id="next">Next ?</button></div>
<script>
  const project=${JSON_DATA}; let i=0;
  const c=document.getElementById('c'), ctx=c.getContext('2d'), info=document.getElementById('info');
  function drawObj(o,s){
    ctx.save(); ctx.globalAlpha=o.opacity??1;
    // rotate about center
    ctx.translate((o.x+o.w/2)*s,(o.y+o.h/2)*s); ctx.rotate((o.r||0)*Math.PI/180); ctx.translate(-(o.w/2)*s, -(o.h/2)*s);
    if(o.type==='erase'){ ctx.globalCompositeOperation='destination-out'; ctx.lineWidth=Math.max(1,(o.strokeWidth||10)*s);
      if(o.points?.length>=2){ ctx.beginPath(); ctx.moveTo((o.points[0].x-o.x)*s,(o.points[0].y-o.y)*s); for(let k=1;k<o.points.length;k++){const p=o.points[k]; ctx.lineTo((p.x-o.x)*s,(p.y-o.y)*s);} ctx.stroke(); }
      ctx.restore(); return; }
    ctx.lineWidth=Math.max(1,(o.strokeWidth||1)*s); ctx.strokeStyle=o.stroke||'#e8eaf0'; ctx.fillStyle=o.fill||'#00000000';
    if(o.type==='rect'||o.type==='panel-rect'){ ctx.beginPath(); ctx.rect(0,0,o.w*s,o.h*s); if((o.fill??'')!=='#00000000') ctx.fill(); ctx.stroke(); }
    else if(o.type==='circle'){ ctx.beginPath(); ctx.ellipse((o.w*s)/2,(o.h*s)/2,Math.abs(o.w*s/2),Math.abs(o.h*s/2),0,0,Math.PI*2); if((o.fill??'')!=='#00000000') ctx.fill(); ctx.stroke(); }
    else if(o.type==='line' && o.points?.length>=2){ ctx.beginPath(); ctx.moveTo((o.points[0].x-o.x)*s,(o.points[0].y-o.y)*s); ctx.lineTo((o.points[1].x-o.x)*s,(o.points[1].y-o.y)*s); ctx.stroke(); }
    else if((o.type==='path'||o.type==='star') && o.points?.length>=2){ ctx.beginPath(); ctx.moveTo((o.points[0].x-o.x)*s,(o.points[0].y-o.y)*s); for(let k=1;k<o.points.length;k++){ const p=o.points[k]; ctx.lineTo((p.x-o.x)*s,(p.y-o.y)*s);} if(o.type==='star'||o.meta?.closed){ ctx.closePath(); if((o.fill??'')!=='#00000000') ctx.fill(); } ctx.stroke(); }
    else if(o.type==='text'){ ctx.fillStyle=o.stroke||'#eaeaf2'; ctx.font=\`\${o.meta?.size||24}px \${o.meta?.family||'sans-serif'}\`; ctx.textBaseline='top'; (o.meta?.text||'Text').split('\\n').forEach((line,i)=> ctx.fillText(line,0,i*(o.meta?.size||24))); }
    ctx.restore();
  }
  function show(k){
    const page=project.pages[k];
    const scale=Math.min((innerWidth-40)/page.w,(innerHeight-100)/page.h);
    c.width=Math.floor(page.w*scale); c.height=Math.floor(page.h*scale);
    ctx.clearRect(0,0,c.width,c.height);
    if(!page.bgTransparent){ ctx.fillStyle=page.bgColor||'#000'; ctx.fillRect(0,0,c.width,c.height); }
    const objs=page.objects.slice().sort((a,b)=>a.layer-b.layer);
    for(const o of objs) drawObj(o, scale);
    info.textContent=(k+1)+"/"+project.pages.length+" — "+project.title;
  }
  document.getElementById('prev').onclick=()=>{ i=(i-1+project.pages.length)%project.pages.length; show(i); };
  document.getElementById('next').onclick=()=>{ i=(i+1)%project.pages.length; show(i); };
  addEventListener('keydown', e=>{ if(e.key==='ArrowLeft') document.getElementById('prev').click(); if(e.key==='ArrowRight') document.getElementById('next').click(); });
  show(0);
<\/script></body></html>`;
  }

  /* ===== Helpers after project change ===== */
  function refreshUIAfterProjectChange(){
    refreshTopMeta(); resizeCanvas(); $('#btnZoomFit').click(); // fit on page change
  }

  /* ===== Init ===== */
  function init(){
    applyTheme();
    state.project=newProject('phik',1024,576);
    setMode('panel'); buildToolButtons(); buildDynamicControls();
    refreshUIAfterProjectChange(); render();
    buildPaletteBar();
    $('#saveKbd').textContent=state.settings.hotkeys.save;
    pushHistory('init');
  }
  addEventListener('resize', resizeCanvas);
  init();
})();
</script>
</body>
</html>
