<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>phik v0.82 — Comic & Storyboard Builder</title>
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
  .row{display:flex;gap:.5rem;align-items:center;flex-wrap:wrap}
  .hide{display:none!important}

  /* main area */
  .main{
    display:grid;
    grid-template-columns: 100px minmax(480px, 1fr) 420px;
    gap:.5rem; padding:.5rem;
  }
  @media (min-width:1600px){
    .main{ grid-template-columns: 110px minmax(800px, 1fr) 480px; }
  }
  .panel{background:var(--panel);border:1px solid var(--line);border-radius:.6rem;overflow:hidden}
  .tools{padding:.5rem;display:flex;flex-direction:column;gap:.4rem}
  .tbtn{display:flex;flex-direction:column;align-items:center;gap:.2rem;border:1px solid var(--line);background:var(--panel-2);padding:.35rem;border-radius:.4rem;cursor:pointer}
  .tbtn.active{outline:2px solid var(--accent)}

  /* small color cluster on the left */
  .colors{margin-top:.4rem; display:flex; flex-direction:column; gap:.35rem}
  .palHeader{display:flex;align-items:center;justify-content:space-between;gap:.25rem}
  .palHeader .bank{display:flex;gap:.25rem}
  .palHeader .bank .btn{padding:.2rem .4rem}
  .bubbles{display:grid; grid-template-columns:repeat(3, 18px); gap:6px; justify-content:start}
  .bubble{width:18px;height:18px;border-radius:50%;border:2px solid var(--line);cursor:pointer}
  .bubble.active{outline:2px solid var(--accent)}
  .sf-mini{display:flex; gap:.35rem; align-items:center}
  .swatch{width:18px;height:18px;border-radius:4px;border:1px solid var(--line);cursor:pointer}
  .snapControls{margin-top:.5rem;display:flex;gap:.35rem;align-items:center}
  .snapControls label{display:flex;align-items:center;gap:.25rem;font-size:.85rem}
  .snapControls input[type="number"]{width:48px}

  .canvasWrap{position:relative; overflow:hidden;
    background: conic-gradient(from 90deg at 1px 1px, #0000 90deg, #0002 0) 0 0/10px 10px;
  }
  /* make canvas layer explicit so handles can sit above */
  .canvasWrap canvas{display:block;background:transparent;width:100%;height:100%;position:relative;z-index:1}

  .right{display:flex;flex-direction:column}
  .section{padding:.5rem;border-bottom:1px solid var(--line)}
  .section h3{margin:.2rem 0 .6rem 0;font-size:1rem}
  .section label{font-size:.85rem;color:var(--muted)}
  input[type="number"], input[type="text"], select, textarea{
    width:100%; background:#0006;color:var(--fg);border:1px solid var(--line);
    border-radius:.35rem;padding:.4rem .5rem;
  }
  textarea{min-height:140px;resize:vertical}
  input[type="range"]{width:100%}
  input[type="color"]{width:100%; height:2rem; padding:0; border:1px solid var(--line); background:#0000}
  .grid{display:grid;grid-template-columns:1fr 1fr;gap:.4rem}
  .stack{display:flex;flex-direction:column;gap:.35rem}

  /* layers list */
  #layerList{max-height:220px;overflow:auto;display:flex;flex-direction:column;gap:.25rem}
  .layerItem{display:flex;align-items:center;gap:.35rem;padding:.25rem .35rem;border:1px solid var(--line);border-radius:.35rem;background:#0003;cursor:pointer}
  .layerItem.active{outline:2px solid var(--accent)}
  .eye{min-width:24px;display:grid;place-items:center;border:1px solid var(--line);border-radius:.25rem;background:#0006;font-family:ui-monospace}
  .thumb{width:48px;height:34px;border:1px solid var(--line);border-radius:.25rem;background:#000;flex:0 0 auto}
  .layerItem input[type="text"]{flex:1;min-width:0;background:#0006;border:1px solid #0000}

  .bottombar{display:flex;gap:.5rem;align-items:center;padding:.5rem;background:linear-gradient(0deg,var(--panel),var(--panel-2)); border-top:1px solid var(--line)}
  #uiToggle{position:fixed;top:.5rem;left:.5rem;z-index:9999;background:var(--accent3);color:#0b1020;border:none;border-radius:999px; width:34px;height:34px;cursor:pointer;font-weight:900}
  .ui-hidden .topbar,.ui-hidden .main,.ui-hidden .bottombar{display:none!important}

  /* marquee + handles + page frame + crop mask */
  .marquee{position:absolute;border:1px dashed var(--accent);background:rgba(122,162,247,.08);pointer-events:none;z-index:3}
  .handle{position:absolute;width:10px;height:10px;border:2px solid #000a;background:#fff;box-shadow:0 0 0 1px #0003;pointer-events:auto;z-index:5}
  .handle[data-dir="move"]{width:12px;height:12px;border-radius:50%; cursor:move}
  .handle[data-dir="n"], .handle[data-dir="s"]{cursor:ns-resize}
  .handle[data-dir="e"], .handle[data-dir="w"]{cursor:ew-resize}
  .handle[data-dir="nw"], .handle[data-dir="se"]{cursor:nwse-resize}
  .handle[data-dir="ne"], .handle[data-dir="sw"]{cursor:nesw-resize}
  .pageFrame{position:absolute;border:2px dashed var(--accent5);pointer-events:none;box-shadow:0 0 0 9999px rgba(0,0,0,.25) inset;z-index:2}
  .cropMask{position:absolute;border:2px dashed var(--accent2);pointer-events:none;background:rgba(247,118,142,.08);z-index:2}

  /* inline text editor */
  #textEditor{position:absolute;min-width:40px;min-height:20px;background:#000a;border:1px dashed var(--accent);color:var(--fg);padding:.2rem .3rem;outline:none;white-space:pre-wrap;pointer-events:auto;display:none;z-index:4}

  .badge{font-size:.8rem;background:#0005;border:1px solid var(--line);border-radius:.35rem;padding:.1rem .4rem}
  .note{font-size:.86rem;color:var(--muted)}
  .hr{height:1px;background:var(--line);margin:.25rem 0 .45rem}
  dialog{border:1px solid var(--line);background:#111827;color:var(--fg);border-radius:.6rem;max-width:min(680px,90vw);padding:1rem}
  #err{position:fixed;right:.75rem;bottom:.75rem;max-width:60ch;padding:.6rem .8rem;background:#2b1a1a;color:#ffd2d2;border:1px solid #5c2b2b;border-radius:.5rem;font-size:.9rem;display:none;z-index:99999;box-shadow:0 8px 24px #0005}

  /* Palettes manager (right sidebar) */
  .palBank{border:1px solid var(--line);border-radius:.4rem;background:#0003;padding:.4rem;margin-bottom:.4rem}
  .palRow{display:flex;gap:.35rem;align-items:center;flex-wrap:wrap}
  .miniCells{display:flex;gap:4px}
  .miniCells div{width:14px;height:14px;border-radius:3px;border:1px solid #0006}
</style>
</head>
<body>
<button id="uiToggle" title="Hide/Show UI" type="button">?</button>
<div id="err"></div>

<div id="app">
  <!-- ===== TOPBAR ===== -->
  <div class="topbar">
    <div class="brand"><span id="appName">phik</span> <span class="pill" id="versionPill">v0.82</span></div>

    <div class="row">
      <button class="btn" id="btnNew"   type="button">New</button>
      <button class="btn" id="btnLoad"  type="button">Load</button>
      <button class="btn" id="btnSave"  type="button" title="Save Project"><span>Save</span> <span class="kbd" id="saveKbd">Ctrl+S</span></button>
      <button class="btn primary" id="btnExport" type="button" title="Export to standalone .html">Export HTML</button>
      <button class="btn" id="btnFont" type="button" title="Import custom font">Import Font</button>
    </div>

    <input type="file" id="fontFile" accept=".ttf,.otf,.woff,.woff2" style="display:none" />

    <div class="spacer"></div>

    <div class="row">
      <div class="seg">
        <button class="btn" id="btnUndo" type="button" title="Undo">Undo</button>
        <button class="btn" id="btnRedo" type="button" title="Redo">Redo</button>
      </div>
      <div class="seg">
        <button class="btn" id="btnZoomOut" type="button">-</button>
        <button class="btn" id="btnZoomFit" type="button">Fit</button>
        <button class="btn" id="btnZoom100" type="button">100%</button>
        <button class="btn" id="btnZoomIn" type="button">+</button>
      </div>
      <button class="btn" id="btnSettings" type="button">Settings</button>
    </div>
  </div>

  <!-- ===== MAIN ===== -->
  <div class="main">
    <!-- LEFT: TOOLS + mini color cluster -->
    <div class="panel tools">
      <div id="toolPanel" class="stack">
        <button class="tbtn" data-tool="panel-rect">Panel Rect</button>
        <button class="tbtn" data-tool="panel-star">Panel Star</button>
        <button class="tbtn" data-tool="panel-line">Panel Line</button>
        <button class="tbtn" data-tool="panel-select">Panel Select</button>
        <button class="tbtn" data-tool="draw-brush">Brush</button>
        <button class="tbtn" data-tool="draw-erase">Erase</button>
        <button class="tbtn" data-tool="draw-line">Line</button>
        <button class="tbtn" data-tool="draw-rect">Rect</button>
        <button class="tbtn" data-tool="draw-circle">Circle</button>
        <button class="tbtn" data-tool="draw-image">Image</button>
        <button class="tbtn" data-tool="draw-bucket">Bucket</button>
        <button class="tbtn" data-tool="draw-crop">Crop</button>
        <button class="tbtn" data-tool="draw-text">Text</button>
        <button class="tbtn" data-tool="draw-hatch">Hatch</button>
        <button class="tbtn" data-tool="draw-select">Select</button>
      </div>

      <div class="colors">
        <div class="palHeader">
          <div class="bank">
            <button class="btn small" id="bankA" title="Use Bank A">A</button>
            <button class="btn small" id="bankB" title="Use Bank B">B</button>
            <button class="btn small" id="bankC" title="Use Bank C">C</button>
          </div>
          <div class="bank">
            <button class="btn small" id="palPrev" title="Prev page">&lt;</button>
            <span class="pill" id="palPageLabel">1/3</span>
            <button class="btn small" id="palNext" title="Next page">&gt;</button>
          </div>
        </div>
        <div class="bubbles" id="palBubbles"></div>
        <div class="sf-mini">
          <div id="swStroke" class="swatch" title="Stroke"></div>
          <div id="swFill" class="swatch" title="Fill"></div>
        </div>
      </div>
      <div class="snapControls">
        <label><input type="checkbox" id="snapToggle"> Snap</label>
        <input type="number" id="snapSize" min="1" value="8" title="Grid size (px)">
      </div>
    </div>

    <!-- CANVAS -->
    <div class="panel canvasWrap" id="canvasPanel">
      <canvas id="canvas"></canvas>
      <!-- overlay removed (caused ghost transparent rect) -->
      <div class="marquee hide" id="marquee"></div>
      <div class="pageFrame" id="pageFrame"></div>
      <div class="cropMask hide" id="cropMask"></div>
      <div id="textEditor" contenteditable="true"></div>
    </div>

    <!-- RIGHT SIDEBAR -->
    <div class="panel right">
      <div class="section">
        <h3 id="sidebarTitle">Tools</h3>
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

        <!-- universal style controls -->
        <div class="grid" style="margin-top:.5rem">
          <div class="stack"><label>Stroke</label><input type="color" id="selStroke" value="#e8eaf0"></div>
          <div class="stack"><label>Fill</label><input type="color" id="selFill" value="#000000"></div>
        </div>
        <div class="grid" style="margin-top:.5rem">
          <div class="stack"><label>Stroke Width</label><input type="number" id="selStrokeW" min="1" max="64" value="2"></div>
          <div class="stack"><label>Opacity %</label><input type="number" id="selOpacity" min="1" max="100" value="100"></div>
        </div>

        <!-- text controls -->
        <div id="textControls" class="stack" style="margin-top:.5rem; display:none">
          <div class="grid">
            <div class="stack"><label>Text</label><input type="text" id="textContent" placeholder="Edit text"></div>
            <div class="stack"><label>Size</label><input type="number" id="textSize" min="6" max="256" value="24"></div>
          </div>
          <div class="stack"><label>Font</label><select id="textFont"></select></div>
          <div class="grid">
            <div class="stack"><label>Text Color</label><input type="color" id="textFill" value="#e8eaf0"></div>
            <div class="stack"><label>Outline Color</label><input type="color" id="textOutlineColor" value="#000000"></div>
          </div>
          <div class="grid">
            <div class="stack"><label>Outline?</label><select id="textOutline"><option value="off">Off</option><option value="on">On</option></select></div>
            <div class="stack"><label>Outline Width</label><input type="number" id="textOutlineWidth" value="2" min="1" max="16"></div>
          </div>
          <span class="note">Click a text object to edit inline. <b>Enter</b> inserts a newline. <b>Ctrl+Enter</b> finishes.</span>
        </div>

        <!-- star controls -->
        <div id="starControls" class="grid" style="margin-top:.5rem; display:none">
          <div class="stack"><label>Points</label><input id="uiStarPoints" type="number" min="3" max="24" value="5"/></div>
          <div class="stack"><label>Inner R</label><input id="uiStarInner" type="number" min="5" value="40"/></div>
          <div class="stack"><label>Outer R</label><input id="uiStarOuter" type="number" min="5" value="100"/></div>
        </div>

        <!-- image controls -->
        <div id="imgControls" class="grid" style="margin-top:.5rem; display:none">
          <div class="stack"><label>Left Offset</label><input id="imgOffL" type="number" value="0"/></div>
          <div class="stack"><label>Right Offset</label><input id="imgOffR" type="number" value="0"/></div>
          <div class="stack"><label>Top Offset</label><input id="imgOffT" type="number" value="0"/></div>
          <div class="stack"><label>Bottom Offset</label><input id="imgOffB" type="number" value="0"/></div>
        </div>
      </div>

      <div class="section">
        <h3>Layers</h3>
        <div id="layerList"></div>
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
        <h3>Palettes (Banks A/B/C)</h3>
        <div id="palBanksUI" class="stack"></div>
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
      <button class="btn" id="btnPrev" type="button">Prev</button>
      <button class="btn" id="btnNext" type="button">Next</button>
    </div>
  </div>
</div>

<!-- SETTINGS -->
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
        <label>Save</label>
        <input id="hkSave" type="text" placeholder="Key (e.g. Ctrl+S)" />
        <label>Undo</label>
        <input id="hkUndo" type="text" placeholder="Ctrl+Z"/>
        <label>Redo</label>
        <input id="hkRedo" type="text" placeholder="Ctrl+Y / Ctrl+Shift+Z"/>
        <label>Tools</label>
        <div class="grid">
          <input id="hkBrush" type="text" placeholder="Brush"/>
          <input id="hkErase" type="text" placeholder="Erase"/>
          <input id="hkLine"  type="text" placeholder="Line"/>
          <input id="hkRect"  type="text" placeholder="Rect"/>
          <input id="hkCircle"type="text" placeholder="Circle"/>
          <input id="hkStar"  type="text" placeholder="Star"/>
          <input id="hkSelect"type="text" placeholder="Select"/>
          <input id="hkHatch" type="text" placeholder="Hatch"/>
          <input id="hkBucket"type="text" placeholder="Bucket"/>
          <input id="hkCrop"  type="text" placeholder="Crop"/>
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
      <div class="stack"><label>Title</label><input type="text" id="newTitle" placeholder="project_#"></div>
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
  /* ===== Helpers ===== */
  const $ = s => document.querySelector(s);
  const $$ = s => Array.from(document.querySelectorAll(s));
  const bind = (sel, ev, fn) => { const el = $(sel); if(!el){ console.warn('Missing', sel); return; } el.addEventListener(ev, fn); };
  const set = (sel, prop, val) => { const el = $(sel); if(!el){ console.warn('Missing', sel); return; } el[prop] = val; };
  const toast = (msg)=>{ const e=$('#err'); e.textContent=msg; e.style.display='block'; clearTimeout(window.__errTo); window.__errTo=setTimeout(()=>e.style.display='none', 6000); };
  window.addEventListener('error', e=> toast('Script error: '+(e.message||e.error)));
  window.addEventListener('unhandledrejection', e=> toast('Unhandled: '+(e.reason?.message||e.reason)));

  const clamp=(v,lo,hi)=>Math.max(lo,Math.min(hi,v));
  const dist=(a,b)=>Math.hypot(a.x-b.x,a.y-b.y);
  const uid=()=>Math.random().toString(36).slice(2,9);
  const rgba=(hex,a=1)=>{ if(!hex) return `rgba(0,0,0,${a})`;
    if(hex.length===9){const r=parseInt(hex.slice(1,3),16),g=parseInt(hex.slice(3,5),16),b=parseInt(hex.slice(5,7),16),aa=parseInt(hex.slice(7,9),16)/255; return `rgba(${r},${g},${b},${(aa*a).toFixed(3)})`;}
    const r=parseInt(hex.slice(1,3),16),g=parseInt(hex.slice(3,5),16),b=parseInt(hex.slice(5,7),16); return `rgba(${r},${g},${b},${a})`; };
  const deepClone=o=>JSON.parse(JSON.stringify(o));
  const downloadFile=(name,text)=>{const blob=new Blob([text],{type:'text/plain'}); const a=document.createElement('a'); a.href=URL.createObjectURL(blob); a.download=name; a.click(); URL.revokeObjectURL(a.href); };
  const LS={set(k,v){localStorage.setItem(k,JSON.stringify(v));},get(k,d){try{const v=localStorage.getItem(k);return v?JSON.parse(v):d;}catch{ return d; }}};

  const TEMPLATES_KEY='PHIK_PANEL_TEMPLATES', STAMPS_KEY='PHIK_STAMPS', SETTINGS_KEY='PHIK_SETTINGS', THEME_KEY='PHIK_THEME';
  const GLOBAL_PALS_KEY='PHIK_GLOBAL_PALETTES';
  const PROJ_BANKS_KEY='palBanks'; // stored on project
  const PROJECT_COUNT_KEY='PHIK_PROJECT_COUNT';

  /* ===== Default palettes ===== */
  const DEFAULT_PAL_PAGES=[
    // Page 1: transparent, black, white, then the rainbow
    ['#00000000','#000000','#ffffff','#ff0000','#ffa500','#ffff00','#00ff00','#0000ff','#4b0082','#ee82ee'],
    // Page 2: metals followed by light-to-dark skin tones
    ['#ffd700','#c0c0c0','#cd7f32','#ffdbac','#f1c27d','#e0ac69','#c68642','#8d5524','#7d3f15','#603914'],
    // Page 3: additional skin tones to round out the default set
    ['#4a2511','#3b200c','#2b1709','#ffe0bd','#ffcd94','#eac086','#d8a577','#bf8a60','#a16b46','#7f4a25']
  ];

  function placeholderPalette(){
    return Array(10).fill('#00000000');
  }

  /* ===== State ===== */
  const defaultsHK = { save:'Ctrl+S', undo:'Ctrl+Z', redo:'Ctrl+Y', brush:'B', erase:'E', line:'L', rect:'R', circle:'C', star:'S', select:'V', hatch:'H', bucket:'K', crop:'X', text:'T' };
  const state = {
    project:null,
    pageIndex:0,
    layer:0,
    tool:'panel-select',
    mouse:{x:0,y:0,down:false,ox:0,oy:0,button:0},
    camera:{x:0,y:0,scale:1,fit:true},
    selection:new Set(),
    marquee:null,
    tmpPath:[],
    draw:{hatchAngle:45,hatchGap:8, hatchGuide:null, hatchPreview:null, hatchOrigin:null},
    bucketBox:null,
    draggingSel:false, dragStartPositions:new Map(),
    panning:false, panStart:{x:0,y:0,cx:0,cy:0},
    resizing:null,
    stroke:'#e8eaf0', fill:'#00000000', strokeWidth:2, opacity:100,
    currentColor:'#e8eaf0',
    snap:{enabled:false,size:8},
    templates: LS.get(TEMPLATES_KEY, []),
    stamps: LS.get(STAMPS_KEY, []),
    // Palettes: 3 banks, each up to 10 pages, each page is array of colors
    palBanks: null, activeBank:0, activePage:[0,0,0],
    settings: (()=>{ const s=LS.get(SETTINGS_KEY, {}); if(!s.hotkeys) s.hotkeys=deepClone(defaultsHK); return s; })(),
    theme: Object.assign({ accent1:'#7aa2f7', accent2:'#f7768e', accent3:'#9ece6a', accent4:'#e0af68', accent5:'#7dcfff' }, LS.get(THEME_KEY, {})),
    history:{stack:[], idx:-1, lock:false},
    editingText:null,
    cropMask:null, // first step rectangle in world space
    textFont:'sans-serif',
    fonts:['sans-serif','serif','monospace'],
    projectCounter: LS.get(PROJECT_COUNT_KEY,1)
  };

  /* ===== Palettes ===== */
  function ensurePalBanks(banks){
    if(banks && banks.length===3) return banks.map(b=>{
      const pages = Array.isArray(b.pages)? b.pages.slice(0,3):[];
      while(pages.length<3) pages.push(placeholderPalette());
      return {
        pages: pages.map(p=>{
          const arr=p.slice(0,10);
          while(arr.length<10) arr.push('#00000000');
          return arr;
        })
      };
    });
    return [
      {pages: DEFAULT_PAL_PAGES.map(p=>p.slice())},
      {pages: DEFAULT_PAL_PAGES.map(p=>p.slice())},
      {pages: DEFAULT_PAL_PAGES.map(p=>p.slice())}
    ];
  }
  function activePalette(){
    const b = state.palBanks[state.activeBank];
    const p = state.activePage[state.activeBank] ?? 0;
    if(!b.pages[p]) b.pages[p] = placeholderPalette();
    return b.pages[p];
  }
  function setActivePalette(arr){
    const b=state.palBanks[state.activeBank], p=state.activePage[state.activeBank];
    b.pages[p] = arr.slice();
  }
  function saveGlobalPalette(name, colors){
    const store = LS.get(GLOBAL_PALS_KEY, {});
    store[name]=colors.slice();
    LS.set(GLOBAL_PALS_KEY, store);
    toast('Saved palette "'+name+'"');
  }
  function loadGlobalPalette(name){
    const store = LS.get(GLOBAL_PALS_KEY, {});
    return store[name]? store[name].slice(): null;
  }
  function listGlobalPaletteNames(){
    const store = LS.get(GLOBAL_PALS_KEY, {});
    return Object.keys(store);
  }

  /* ===== Project ===== */
  function normalizeLayers(layers){
    if(Array.isArray(layers) && typeof layers[0]==='number'){
      return layers.map(id=>({id, name:`Layer ${id}`, visible:true}));
    }
    return layers && layers.length ? layers : [{id:0,name:'Layer 0',visible:true}];
  }
  function newProject(title,w=1024,h=576){
    const palBanks = ensurePalBanks(null);
    return { title, defaultW:w, defaultH:h, pages:[newPage(w,h)], nextId:1, layers:normalizeLayers([0,1,2,3,4]), [PROJ_BANKS_KEY]:palBanks };
  }
  function newPage(w,h){ return { w,h, bgTransparent:true, bgColor:'#000000', note:'', objects:[] }; }
  function newObj(type,props={}){
    return Object.assign({
      id:uid(), type, kind: state.tool.startsWith('panel-')?'panel':'draw',
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

  /* ===== History ===== */
  function snapshot(){ return JSON.stringify(state.project); }
  function pushHistory(){ if(state.history.lock) return; const s=snapshot(); if(state.history.stack[state.history.idx]===s) return; state.history.stack = state.history.stack.slice(0,state.history.idx+1); state.history.stack.push(s); state.history.idx++; }
  function undo(){ if(state.history.idx<=0) return; const cam=deepClone(state.camera);
    state.history.lock=true; state.history.idx--; state.project = JSON.parse(state.history.stack[state.history.idx]);
    // restore palbanks into state
    state.palBanks = ensurePalBanks(state.project[PROJ_BANKS_KEY]);
    state.selection.clear(); state.resizing=null; state.draggingSel=false; state.drawing=null;
    refreshUIAfterProjectChange({preserveCamera:true}); state.camera = cam; state.history.lock=false;
  }
  function redo(){ if(state.history.idx>=state.history.stack.length-1) return; const cam=deepClone(state.camera);
    state.history.lock=true; state.history.idx++; state.project = JSON.parse(state.history.stack[state.history.idx]);
    state.palBanks = ensurePalBanks(state.project[PROJ_BANKS_KEY]);
    state.selection.clear(); state.resizing=null; state.draggingSel=false; state.drawing=null;
    refreshUIAfterProjectChange({preserveCamera:true}); state.camera = cam; state.history.lock=false;
  }

  /* ===== Canvas ===== */
  const canvas=$('#canvas'), ctx=canvas.getContext('2d'), marqueeEl=$('#marquee'), canvasPanel=$('#canvasPanel'), pageFrame=$('#pageFrame'), textEditor=$('#textEditor'), cropMaskEl=$('#cropMask');

  function resizeCanvas(){
    const rect=canvasPanel.getBoundingClientRect();
    canvas.width = Math.max(200, Math.floor(rect.width));
    canvas.height= Math.max(200, Math.floor(rect.height));
    if(state.camera.fit) fitPage(false);
    drawPageFrame(); drawCropMask();
  }
  function drawPageFrame(){
    if(!state.project) return;
    const p=currPage(); const tl=worldToScreen(0,0), br=worldToScreen(p.w,p.h);
    const x=Math.round(tl.x), y=Math.round(tl.y), w=Math.round(br.x-tl.x), h=Math.round(br.y-tl.y);
    pageFrame.style.left=x+'px'; pageFrame.style.top=y+'px'; pageFrame.style.width=w+'px'; pageFrame.style.height=h+'px';
  }
  function drawCropMask(){
    const m=state.cropMask; if(!m){ cropMaskEl.classList.add('hide'); return; }
    const a=worldToScreen(m.x,m.y), b=worldToScreen(m.x+m.w, m.y+m.h);
    cropMaskEl.style.left=Math.min(a.x,b.x)+'px';
    cropMaskEl.style.top=Math.min(a.y,b.y)+'px';
    cropMaskEl.style.width=Math.abs(b.x-a.x)+'px';
    cropMaskEl.style.height=Math.abs(b.y-a.y)+'px';
    cropMaskEl.classList.remove('hide');
  }
  const screenToWorld=(x,y)=>({x:x/state.camera.scale+state.camera.x, y:y/state.camera.scale+state.camera.y});
  const worldToScreen=(x,y)=>({x:(x-state.camera.x)*state.camera.scale, y:(y-state.camera.y)*state.camera.scale});
  const snapValue = v => { const s=state.snap.size; return Math.round(v/s)*s; };
  const snapPoint = p => state.snap.enabled ? {x:snapValue(p.x), y:snapValue(p.y)} : p;

  function clearCanvas(){
    ctx.clearRect(0,0,canvas.width,canvas.height);
    const p=currPage();
    const a=worldToScreen(0,0), b=worldToScreen(p.w,p.h);
    if(!p.bgTransparent){
      ctx.save(); ctx.fillStyle=p.bgColor||'#000000'; ctx.fillRect(a.x,a.y,b.x-a.x,b.y-a.y); ctx.restore();
    }
    ctx.save(); ctx.strokeStyle='rgba(255,255,255,.4)'; ctx.setLineDash([8,6]); ctx.lineWidth=1;
    ctx.strokeRect(a.x+0.5,a.y+0.5,(b.x-a.x)-1,(b.y-a.y)-1); ctx.setLineDash([]); ctx.restore();
  }

  function layerVisible(Lid){ const L = state.project.layers.find(l=>l.id===+Lid); return !L || L.visible!==false; }

  function drawObject(o){
    if(!layerVisible(o.layer)) return;
    const s = state.camera.scale;
    const sc = worldToScreen(o.x, o.y);
    if(o.type==='image'){
      if(!o.meta.img && o.meta.src){
        const img=new Image(); img.src=o.meta.src;
        Object.defineProperty(o.meta,'img',{value:img, enumerable:false});
        img.onload=()=>{ o.w=img.width - (o.meta.left||0) - (o.meta.right||0); o.h=img.height - (o.meta.top||0) - (o.meta.bottom||0); };
      }
      if(o.meta.img){
        ctx.save();
        ctx.globalAlpha = (o.opacity ?? 1);
        const px = sc.x + (o.w*s)/2, py = sc.y + (o.h*s)/2;
        ctx.translate(px, py); ctx.rotate((o.r||0)*Math.PI/180); ctx.translate(-(o.w*s)/2, -(o.h*s)/2);
        const img=o.meta.img;
        const sx=o.meta.left||0, sy=o.meta.top||0;
        const sw=img.width - (o.meta.left||0) - (o.meta.right||0);
        const sh=img.height - (o.meta.top||0) - (o.meta.bottom||0);
        ctx.drawImage(img, sx, sy, sw, sh, 0, 0, o.w*s, o.h*s);
        ctx.restore();
      }
      if(state.selection.has(o.id)){
        const p=worldToScreen(o.x,o.y);
        ctx.save(); ctx.setLineDash([6,6]); ctx.strokeStyle='#7aa2f7'; ctx.lineWidth=1;
        ctx.strokeRect(p.x+0.5,p.y+0.5,o.w*s-1,o.h*s-1); ctx.setLineDash([]); ctx.restore();
      }
      return;
    }

    ctx.save();
    ctx.globalAlpha = (o.opacity ?? 1);

    // rotate about center
    const px = sc.x + (o.w*s)/2, py = sc.y + (o.h*s)/2;
    ctx.translate(px, py); ctx.rotate((o.r||0)*Math.PI/180); ctx.translate(-(o.w*s)/2, -(o.h*s)/2);

    ctx.lineWidth = Math.max(1, (o.strokeWidth||1)*s);
    ctx.strokeStyle = rgba(o.stroke||'#e8eaf0', 1);
    ctx.fillStyle   = rgba(o.fill||'#00000000', 1);

    switch(o.type){
      case 'panel-rect': case 'rect':
        ctx.beginPath(); ctx.rect(0,0,o.w*s,o.h*s); if((o.fill??'')!=='#00000000') ctx.fill(); ctx.stroke(); break;
      case 'circle':
      case 'ellipse':
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
        const fillCol = o.meta.fill || o.stroke || '#e8eaf0';
        const outlineOn = !!o.meta.outline;
        const outlineCol = o.meta.outlineColor || '#000';
        const outlineW = o.meta.outlineWidth || 2;
        ctx.font = `${o.meta.size||24}px ${o.meta.family||'sans-serif'}`;
        ctx.textBaseline='top';
        ctx.fillStyle = fillCol;
        const lines=(o.meta.text||'Text').split('\n');
        lines.forEach((line,i)=>{
          if(outlineOn){ ctx.lineWidth = Math.max(1, outlineW*s); ctx.strokeStyle=outlineCol; ctx.strokeText(line, 0, i*(o.meta.size||24)); }
          ctx.fillText(line, 0, i*(o.meta.size||24));
        });
        break;
    }
    ctx.restore();

    if(state.selection.has(o.id)){
      const p = worldToScreen(o.x,o.y);
      ctx.save(); ctx.setLineDash([6,6]); ctx.strokeStyle='#7aa2f7'; ctx.lineWidth=1;
      ctx.strokeRect(p.x+0.5,p.y+0.5,o.w*s-1,o.h*s-1); ctx.setLineDash([]); ctx.restore();
    }
  }

  function selectionBBox(){
    const items=[...state.selection].map(getById).filter(Boolean).filter(o=>layerVisible(o.layer));
    if(!items.length) return null;
    let minx=Infinity,miny=Infinity,maxx=-Infinity,maxy=-Infinity;
    for(const o of items){
      minx=Math.min(minx,o.x); miny=Math.min(miny,o.y);
      maxx=Math.max(maxx,o.x+o.w); maxy=Math.max(maxy,o.y+o.h);
    }
    return {x:minx,y:miny,w:maxx-minx,h:maxy-miny};
  }

  function drawHandles(){
    $$('#canvasPanel .handle').forEach(h=>h.remove());
    const bb=selectionBBox(); if(!bb) return;
    const pts=[
      {dir:'nw', x:bb.x,           y:bb.y},
      {dir:'ne', x:bb.x+bb.w,      y:bb.y},
      {dir:'sw', x:bb.x,           y:bb.y+bb.h},
      {dir:'se', x:bb.x+bb.w,      y:bb.y+bb.h},
      {dir:'move',x:bb.x+bb.w/2,   y:bb.y+bb.h/2}
    ];
    for(const p of pts){
      const sp=worldToScreen(p.x,p.y);
      const h=document.createElement('div'); h.className='handle'; h.dataset.dir=p.dir;
      h.style.left=(sp.x-6)+'px'; h.style.top=(sp.y-6)+'px';
      canvasPanel.appendChild(h);
      h.addEventListener('mousedown', e=>{
        e.stopPropagation(); e.preventDefault();
        if(p.dir==='move'){
          state.draggingSel=true; state.dragStartPositions.clear();
          for(const id of state.selection){ const o=getById(id); if(o) state.dragStartPositions.set(id,{x:o.x,y:o.y}); }
        } else {
          startResize(p.dir);
        }
      });
    }
  }

  function render(){
    if(!state.project) return requestAnimationFrame(render);
    clearCanvas();
    const objs = allObjects().slice().sort((a,b)=> a.layer-b.layer);
    for(const o of objs) drawObject(o);

    // hatch preview (during drag) — now circular from origin
    if(state.draw.hatchPreview && state.draw.hatchOrigin){
      const o = state.draw.hatchOrigin;
      const r = state.draw.hatchPreview.r;
      const minx=o.x-r, maxx=o.x+r, miny=o.y-r, maxy=o.y+r;
      const gap=+($('#hatchGap')?.value||state.draw.hatchGap);
      const ang=(+($('#hatchAngle')?.value||state.draw.hatchAngle))*Math.PI/180;
      const diag=r*2*Math.SQRT2;
      ctx.save(); ctx.strokeStyle=rgba(state.currentColor,.7); ctx.lineWidth=1; ctx.setLineDash([6,6]);
      for(let d=-diag; d<diag; d+=gap){
        const x1=o.x + d*Math.cos(ang), y1=o.y + d*Math.sin(ang);
        const x2=x1 + (maxx-minx)*Math.sin(ang+Math.PI/2), y2=y1 - (maxx-minx)*Math.cos(ang+Math.PI/2);
        const s1=worldToScreen(x1,y1), s2=worldToScreen(x2,y2);
        ctx.beginPath(); ctx.moveTo(s1.x,s1.y); ctx.lineTo(s2.x,s2.y); ctx.stroke();
      }
      ctx.setLineDash([]); ctx.restore();
      // show circle guide
      const sc=worldToScreen(o.x,o.y);
      ctx.save(); ctx.strokeStyle=rgba('#ffffff',.35);
      ctx.beginPath(); ctx.arc(sc.x,sc.y,r*state.camera.scale,0,Math.PI*2); ctx.stroke(); ctx.restore();
    }

    // hatch angle guide
    if(state.draw.hatchGuide){
      const s=worldToScreen(state.draw.hatchGuide.a.x, state.draw.hatchGuide.a.y);
      const e=worldToScreen(state.draw.hatchGuide.b.x, state.draw.hatchGuide.b.y);
      ctx.save(); ctx.strokeStyle=rgba(state.currentColor,.9); ctx.setLineDash([4,4]); ctx.lineWidth=2;
      ctx.beginPath(); ctx.moveTo(s.x,s.y); ctx.lineTo(e.x,e.y); ctx.stroke(); ctx.setLineDash([]); ctx.restore();
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

    if(state.tool.endsWith('select') && state.selection.size) drawHandles();
    else $$('#canvasPanel .handle').forEach(h=>h.remove());

    drawPageFrame(); drawCropMask();
    requestAnimationFrame(render);
  }

  /* ===== Hit / select ===== */
  const aabb = o => ({x:o.x,y:o.y,w:o.w,h:o.h});
  const pointInAABB=(px,py,bb)=>{
    const x1=Math.min(bb.x, bb.x+bb.w), x2=Math.max(bb.x, bb.x+bb.w);
    const y1=Math.min(bb.y, bb.y+bb.h), y2=Math.max(bb.y, bb.y+bb.h);
    return px>=x1 && py>=y1 && px<=x2 && py<=y2;
  };
  const rectsIntersect = (A,B)=> !(B.x>A.x+A.w || B.x+B.w<A.x || B.y>A.y+A.h || B.y+B.h<A.y);

  function hitTest(world, kind=null){
    const objects = allObjects().slice().reverse();
    for(const o of objects){
      if(kind && o.kind!==kind) continue;
      if(!layerVisible(o.layer)) continue;
      if(pointInAABB(world.x,world.y,aabb(o))) return o;
    }
    return null;
  }

  function selectInMarquee_intersect(add=false, kind=null){
    if(!state.marquee) return;
    const xm=Math.min(state.marquee.x,state.marquee.x+state.marquee.w);
    const ym=Math.min(state.marquee.y,state.marquee.y+state.marquee.h);
    const wm=Math.abs(state.marquee.w), hm=Math.abs(state.marquee.h);
    const topLeft=screenToWorld(xm,ym), bottomRight=screenToWorld(xm+wm,ym+hm);
    const box={x:topLeft.x,y:topLeft.y,w:bottomRight.x-topLeft.x,h:bottomRight.y-topLeft.y};
    if(!add) state.selection.clear();
    for(const o of allObjects()){
      if(kind && o.kind!==kind) continue;
      if(!layerVisible(o.layer)) continue;
      if(rectsIntersect(aabb(o), box)) state.selection.add(o.id);
    }
  }

  /* ===== Tools ===== */
  const TOOLS_PANEL = [
    {id:'panel-rect', label:'Panel Rect'},
    {id:'panel-star', label:'Panel Star'},
    {id:'panel-line', label:'Panel Line'},
    {id:'panel-select', label:'Panel Select'}
  ];
  // tools available while drawing
  const TOOLS_DRAW = [
    // freehand brush for sketching
    {id:'draw-brush', label:'Brush'},
    {id:'draw-erase', label:'Erase'},
    {id:'draw-line',  label:'Line'},
    // rectangle drawing tool
    {id:'draw-rect',  label:'Rect'},
    {id:'draw-circle',label:'Circle'},
    {id:'draw-ellipse',label:'Ellipse'},
    {id:'draw-star', label:'Star'},
    {id:'draw-image', label:'Image'},
    {id:'draw-bucket',label:'Bucket'},
    {id:'draw-crop',  label:'Crop'},
    {id:'draw-text',  label:'Text'},
    {id:'draw-hatch', label:'Hatch'},
    {id:'draw-select',label:'Select'}
  ];
  function toolLabel(id){ return ([...TOOLS_PANEL,...TOOLS_DRAW].find(t=>t.id===id)||{}).label||id; }

  function buildToolButtons(){
    const tpanel=$('#toolPanel');
    if(!tpanel) return;
    if(!tpanel.children.length){
      const defs=[...TOOLS_PANEL, ...TOOLS_DRAW];
      defs.forEach(d=>{
        const b=document.createElement('button');
        b.type='button';
        b.className='tbtn';
        b.dataset.tool=d.id;
        b.textContent=d.label;
        tpanel.appendChild(b);
      });
    }
    $$('#toolPanel .tbtn').forEach(btn=>{
      const id=btn.dataset.tool;
      btn.onclick=()=>{ state.tool=id; buildDynamicControls(); updateToolButtons(); };
    });
    updateToolButtons();
    buildPaletteBar();
  }
  function updateToolButtons(){
    $$('#toolPanel .tbtn').forEach(btn=>{
      btn.classList.toggle('active', btn.dataset.tool===state.tool);
    });
  }

  function buildDynamicControls(){
    const wrap=$('#dynamicControls'); wrap.innerHTML='';
    const add=html=>wrap.insertAdjacentHTML('beforeend',html);
    if(state.tool.startsWith('panel-')){
      if(state.tool==='panel-rect'){ add(`<p class="note">Drag to create rectangles. Use Select to move/scale.</p>`); }
      if(state.tool==='panel-star'){ add(`<div class="grid">
        <div class="stack"><label>Points</label><input id="starPoints" type="number" value="5" min="3" max="24"/></div>
        <div class="stack"><label>Inner Radius</label><input id="starInner" type="number" value="40" min="5" /></div>
        <div class="stack"><label>Outer Radius</label><input id="starOuter" type="number" value="100" min="5" /></div>
      </div><p class="note">Click an existing star to edit; click-drag to place a new star (scales while dragging).</p>`); }
      if(state.tool==='panel-line'){ add(`<p class="note">Click-drag to place a line. Hold Shift to lock to 0°/90°.</p>`); }
      if(state.tool==='panel-select'){ add(`<p class="note">Click toggles selection. Drag to marquee (selects anything it touches). Drag inside to move; handles to scale.</p>`); }
    } else {
      if(state.tool==='draw-brush'){
        add(`<div class="grid">
          <div class="stack"><label>Size</label><input id="sizeBrush" type="range" min="1" max="64" value="${state.strokeWidth}"/></div>
          <div class="stack"><label>Round/Rect</label><select id="brushShape"><option>round</option><option>square</option></select></div>
        </div>`);
      }
      if(state.tool==='draw-text'){
        add(`<div class="grid">
          <div class="stack"><label>Text</label><input id="toolTextContent" type="text" value="Text"></div>
          <div class="stack"><label>Size</label><input id="toolTextSize" type="number" min="6" max="256" value="24"></div>
        </div>
        <div class="stack"><label>Font</label><select id="toolTextFont"></select></div>
        <p class="note">Click canvas to place text; click a text while in Text tool to select/edit it.</p>`);
        buildFontSelect($('#toolTextFont'));
        $('#toolTextFont').value=state.textFont;
        bind('#toolTextFont','change', e=>{ state.textFont=e.target.value; });
      }
        if(state.tool==='draw-image'){
          add(`<div class="stack"><label>Image File</label><input type="file" id="imgFile" accept="image/*"/></div>
          <p class="note">Choose an image to place on the canvas.</p>`);
          bind('#imgFile','change', e=>{
            const file=e.target.files[0]; if(!file) return;
            const reader=new FileReader();
            reader.onload=ev=>{
              const img=new Image();
              img.onload=()=>{
                const o=newObj('image',{x:0,y:0,w:img.width,h:img.height, meta:{src:ev.target.result,left:0,top:0,right:0,bottom:0}});
                Object.defineProperty(o.meta,'img',{value:img, enumerable:false});
                currPage().objects.push(o); state.selection=new Set([o.id]); pushHistory();
                state.tool='draw-select';
                updateToolButtons();
                buildDynamicControls();
              };
              img.src=ev.target.result;
            };
            reader.readAsDataURL(file);
          });
        }
      if(state.tool==='draw-star'){
        add(`<div class="grid">
        <div class="stack"><label>Points</label><input id="starPoints" type="number" value="5" min="3" max="24"/></div>
        <div class="stack"><label>Inner Radius</label><input id="starInner" type="number" value="40" min="5" /></div>
        <div class="stack"><label>Outer Radius</label><input id="starOuter" type="number" value="100" min="5" /></div>
      </div><p class="note">Click-drag to place a star (scales while dragging).</p>`);
      }
      if(state.tool==='draw-hatch'){
        add(`<div class="grid">
          <div class="stack"><label>Angle</label><input id="hatchAngle" type="range" min="-180" max="180" value="${state.draw.hatchAngle}"/></div>
          <div class="stack"><label>Gap</label><input id="hatchGap" type="range" min="4" max="32" value="${state.draw.hatchGap}"/></div>
        </div><p class="note">Drag from an origin: size is radius from click. <b>Middle-drag</b> shows a guide and sets the angle.</p>`);
      }
      if(['draw-brush','draw-line','draw-rect','draw-circle','draw-ellipse','draw-star','draw-bucket','draw-crop','draw-erase'].includes(state.tool)){
        const tips={
          'draw-brush':'Freehand pen.',
          'draw-line':'Click-drag; free angle. Hold Shift to lock to 0°/90°.',
          'draw-rect':'Drag from corner.',
          'draw-circle':'Drag from center.',
          'draw-ellipse':'Drag from corner.',
          'draw-star':'Drag from center.',
          'draw-bucket':'Click one; or drag a box to recolor everything it touches.',
          'draw-crop':'Two-step: first choose mask; second choose keep-area (deletes ring).',
          'draw-erase':'Click deletes brush strokes; drag a box to delete all strokes it touches.'
        };
        add(`<p class="note">${tips[state.tool]}</p>`);
      }
      if(state.tool==='draw-select'){ add(`<p class="note">Click toggles selection. Drag to marquee (touch). Drag inside to move; corner handles to scale; center handle to grab.</p>`); }
    }
    updateTextControls();
  }

  function buildFontSelect(el){
    if(!el) return;
    el.innerHTML='';
    state.fonts.forEach(f=>{
      const opt=document.createElement('option');
      opt.value=f; opt.textContent=f;
      el.appendChild(opt);
    });
  }

  /* ===== Theme & Palette (UI) ===== */
  function applyTheme(){
    const r=document.documentElement, t=state.theme;
    r.style.setProperty('--accent',t.accent1); r.style.setProperty('--accent2',t.accent2);
    r.style.setProperty('--accent3',t.accent3); r.style.setProperty('--accent4',t.accent4); r.style.setProperty('--accent5',t.accent5);
  }
  function buildPaletteBar(){
    const wrap=$('#palBubbles'); wrap.innerHTML='';
    const pal = activePalette();
    pal.forEach((c,i)=>{
      const b=document.createElement('div');
      b.className='bubble'+(c===state.currentColor?' active':'');
      b.style.background=c;
      b.title=`Color ${i+1}`;
      b.addEventListener('click', ()=>{
        state.currentColor=c;
        state.stroke=c;
        updateMiniSwatches();
        buildPaletteBar();
      });
      b.addEventListener('contextmenu', ev=>{
        ev.preventDefault();
        state.fill=c;
        updateMiniSwatches();
      });
      b.addEventListener('dblclick', ev=>{
        ev.preventDefault();
        const inp=document.createElement('input');
        inp.type='color';
        inp.value=c;
        inp.style.position='fixed';
        inp.style.left='-9999px';
        document.body.appendChild(inp);
        inp.addEventListener('input', ()=>{
          pal[i]=inp.value;
          setActivePalette(pal);
          buildPaletteBar();
          buildPalBanksUI();
        });
        inp.addEventListener('change', ()=>{ document.body.removeChild(inp); });
        inp.click();
      });
      wrap.appendChild(b);
    });
    $('#palPageLabel').textContent = (state.activePage[state.activeBank]+1) + '/3';
    updateMiniSwatches();
  }
  function updateMiniSwatches(){
    $('#swStroke').style.background=state.stroke=state.currentColor;
    $('#swFill').style.background=state.fill;
    $('#swStroke').onclick=()=>{ const inp=document.createElement('input'); inp.type='color'; inp.value=state.stroke; inp.style.position='fixed'; inp.style.left='-9999px'; document.body.appendChild(inp);
      inp.addEventListener('input', ()=>{ state.stroke=state.currentColor=inp.value; buildPaletteBar(); });
      inp.addEventListener('change', ()=>document.body.removeChild(inp)); inp.click();
    };
    $('#swFill').onclick=()=>{ const inp=document.createElement('input'); inp.type='color'; inp.value=(state.fill==='#00000000')?'#000000':state.fill; inp.style.position='fixed'; inp.style.left='-9999px'; document.body.appendChild(inp);
      inp.addEventListener('input', ()=>{ state.fill=inp.value; $('#swFill').style.background=state.fill; });
      inp.addEventListener('change', ()=>document.body.removeChild(inp)); inp.click();
    };
  }
  function buildPalBanksUI(){
    const host = $('#palBanksUI'); host.innerHTML='';
    const labels=['A','B','C'];
    state.palBanks.forEach((bank,bi)=>{
      const page = state.activePage[bi]??0;
      const wrap=document.createElement('div'); wrap.className='palBank';
      const row=document.createElement('div'); row.className='palRow';
      const title=document.createElement('strong'); title.textContent=`Bank ${labels[bi]} — Page ${page+1}/3`;
      const useBtn=document.createElement('button'); useBtn.className='btn small'; useBtn.textContent = (state.activeBank===bi)?'Using':'Use';
      useBtn.addEventListener('click', ()=>{ state.activeBank=bi; buildPaletteBar(); buildPalBanksUI(); });
      const prev=document.createElement('button'); prev.className='btn small'; prev.textContent='<';
      prev.addEventListener('click', ()=>{ state.activePage[bi]=Math.max(0,(state.activePage[bi]||0)-1); buildPalBanksUI(); if(bi===state.activeBank) buildPaletteBar(); });
      const next=document.createElement('button'); next.className='btn small'; next.textContent='>';
      next.addEventListener('click', ()=>{ state.activePage[bi]=Math.min(2,(state.activePage[bi]||0)+1); if(!bank.pages[state.activePage[bi]]) bank.pages[state.activePage[bi]]=placeholderPalette(); buildPalBanksUI(); if(bi===state.activeBank) buildPaletteBar(); });
      const load=document.createElement('button'); load.className='btn small'; load.textContent='Load';
      load.addEventListener('click', ()=>{
        const names=listGlobalPaletteNames();
        const name=prompt('Load palette by name:\n'+(names.length?names.join(', '):'(none saved yet)'));
        if(!name) return;
        const pal=loadGlobalPalette(name);
        if(!pal) return alert('Not found');
        while(pal.length<10) pal.push('#00000000');
        bank.pages[state.activePage[bi]||0]=pal.slice(0,10);
        if(bi===state.activeBank) buildPaletteBar();
        buildPalBanksUI();
      });
      const save=document.createElement('button'); save.className='btn small'; save.textContent='Save';
      save.addEventListener('click', ()=>{
        const pal=(bank.pages[state.activePage[bi]||0]||placeholderPalette()).slice(0,10);
        const name=prompt('Name for palette to save globally:', 'palette-'+labels[bi]+'-'+(state.activePage[bi]+1));
        if(!name) return;
        saveGlobalPalette(name, pal);
      });

      const minis=document.createElement('div'); minis.className='miniCells';
      (bank.pages[state.activePage[bi]||0]||placeholderPalette()).slice(0,10).forEach(c=>{ const d=document.createElement('div'); d.style.background=c; minis.appendChild(d); });

      row.appendChild(title); row.appendChild(useBtn); row.appendChild(prev); row.appendChild(next); row.appendChild(load); row.appendChild(save);
      wrap.appendChild(row); wrap.appendChild(minis);
      host.appendChild(wrap);
    });
  }

  /* ===== Layers UI ===== */
  function buildLayerList(){
    const list=$('#layerList'); list.innerHTML='';
    for(const L of state.project.layers){
      const row=document.createElement('div'); row.className='layerItem'+(state.layer===L.id?' active':''); row.dataset.id=L.id;
      const eye=document.createElement('div'); eye.className='eye'; eye.textContent = (L.visible===false)?'--':'O';
      const name=document.createElement('input'); name.type='text'; name.value=L.name||`Layer ${L.id}`;
      const thumb=document.createElement('canvas'); thumb.width=96; thumb.height=68; thumb.className='thumb';
      row.appendChild(eye); row.appendChild(thumb); row.appendChild(name);
      list.appendChild(row);
      drawLayerThumb(thumb, L.id);
      eye.addEventListener('click', e=>{ e.stopPropagation(); L.visible = !(L.visible!==false); buildLayerList(); });
      name.addEventListener('input', ()=>{ L.name=name.value; });
      row.addEventListener('click', ()=>{ state.layer=L.id; buildLayerList(); });
      row.addEventListener('mousedown', e=>{ if(e.button===1){ e.preventDefault(); }});
      row.addEventListener('auxclick', e=>{ if(e.button===1){ e.preventDefault(); }});
    }
  }
  function drawLayerThumb(cv, lid){
    const tctx=cv.getContext('2d');
    tctx.clearRect(0,0,cv.width,cv.height);
    const p=currPage();
    const s=Math.min((cv.width-6)/p.w, (cv.height-6)/p.h);
    const ox=(cv.width - p.w*s)/2, oy=(cv.height - p.h*s)/2;
    tctx.fillStyle='#000'; tctx.fillRect(0,0,cv.width,cv.height);
    if(!p.bgTransparent){ tctx.fillStyle=p.bgColor; tctx.fillRect(ox,oy,p.w*s,p.h*s); }
    const objs=allObjects().filter(o=>o.layer===lid);
    const drawObj=(o)=>{
      tctx.save();
      tctx.globalAlpha=o.opacity??1;
      tctx.translate(ox+(o.x+o.w/2)*s, oy+(o.y+o.h/2)*s); tctx.rotate((o.r||0)*Math.PI/180); tctx.translate(-(o.w/2)*s, -(o.h/2)*s);
      tctx.lineWidth=Math.max(1,(o.strokeWidth||1)*s); tctx.strokeStyle=o.stroke||'#e8eaf0'; tctx.fillStyle=o.fill||'#0000';
      if(o.type==='rect'||o.type==='panel-rect'){ tctx.beginPath(); tctx.rect(0,0,o.w*s,o.h*s); if((o.fill??'')!=='#00000000') tctx.fill(); tctx.stroke(); }
      else if(o.type==='circle' || o.type==='ellipse'){ tctx.beginPath(); tctx.ellipse((o.w*s)/2,(o.h*s)/2,Math.abs(o.w*s/2),Math.abs(o.h*s/2),0,0,Math.PI*2); if((o.fill??'')!=='#00000000') tctx.fill(); tctx.stroke(); }
      else if(o.type==='line' && o.points?.length>=2){ tctx.beginPath(); tctx.moveTo((o.points[0].x-o.x)*s,(o.points[0].y-o.y)*s); tctx.lineTo((o.points[1].x-o.x)*s,(o.points[1].y-o.y)*s); tctx.stroke(); }
      else if((o.type==='path'||o.type==='star') && o.points?.length>=2){ tctx.beginPath(); tctx.moveTo((o.points[0].x-o.x)*s,(o.points[0].y-o.y)*s); for(let k=1;k<o.points.length;k++){ const p=o.points[k]; tctx.lineTo((p.x-o.x)*s,(p.y-o.y)*s);} if(o.type==='star'||o.meta?.closed){ tctx.closePath(); if((o.fill??'')!=='#00000000') tctx.fill(); } tctx.stroke(); }
      else if(o.type==='text'){ tctx.font=`${o.meta?.size||24}px ${o.meta?.family||'sans-serif'}`; tctx.textBaseline='top'; tctx.fillStyle=o.meta?.fill||o.stroke||'#eaeaf2'; tctx.fillText((o.meta?.text||'T').slice(0,1),0,0); }
      tctx.restore();
    };
    objs.forEach(drawObj);
    tctx.strokeStyle='#3a455f'; tctx.strokeRect(0.5,0.5,cv.width-1,cv.height-1);
  }

  /* ===== Library ===== */
  function buildLibrary(){
    const P=$('#libPanels'); const S=$('#libStamps'); P.innerHTML=''; S.innerHTML='';
    const mk=(arr,wrap)=> arr.length? arr.map((t)=>{ const r=document.createElement('div'); r.className='row'; const b=document.createElement('button'); b.className='btn small'; b.textContent='Place'; const lbl=document.createElement('span'); lbl.className='note'; lbl.textContent=t.name; r.appendChild(b); r.appendChild(lbl); wrap.appendChild(r); b.addEventListener('click', ()=> placeTemplate(t.items)); }): (wrap.innerHTML='<span class="note">Empty — save some!</span>');
    mk(state.templates, P); mk(state.stamps, S);
  }
  function placeTemplate(items){
    const minx=Math.min(...items.map(o=>o.x)), miny=Math.min(...items.map(o=>o.y));
    const maxx=Math.max(...items.map(o=>o.x+o.w)), maxy=Math.max(...items.map(o=>o.y+o.h));
    const cx=(minx+maxx)/2, cy=(miny+maxy)/2;
    const center=screenToWorld(canvas.width/2, canvas.height/2);
    const dx=center.x-cx, dy=center.y-cy;
    const placed = items.map(o=>{
      const n=deepClone(o);
      n.id=uid();
      n.x+=dx;
      n.y+=dy;
      currPage().objects.push(n);
      return n.id;
    });
    state.selection=new Set(placed);
    pushHistory();
  }

  /* ===== Meta/UI ===== */
  function refreshTopMeta(){
    set('#projName','textContent', state.project?.title || '—');
    set('#pageInfo','textContent', (state.pageIndex+1)+' / '+(state.project?.pages.length||0));
    $('#pageW').value=currPage().w; $('#pageH').value=currPage().h;
    $('#pageBg').value=currPage().bgColor||'#000000';
    $('#pageBgTransparent').value = currPage().bgTransparent?'true':'false';
    $('#pageNote').value = currPage().note||'';
    buildLayerList(); buildLibrary();
  }

  /* ===== Resizing ===== */
  function startResize(dir){
    const bb=selectionBBox(); if(!bb) return;
    const items=new Map();
    for(const id of state.selection){ const o=getById(id); if(!o) continue; items.set(id, deepClone(o)); }
    state.resizing={dir, bbox0:bb, items};
  }
  function applyResize(world){
    const R=state.resizing; if(!R) return; const bb0=R.bbox0;
    const anchors={ 'nw':{ax:bb0.x+bb0.w, ay:bb0.y+bb0.h}, 'ne':{ax:bb0.x, ay:bb0.y+bb0.h}, 'sw':{ax:bb0.x+bb0.w, ay:bb0.y}, 'se':{ax:bb0.x, ay:bb0.y} };
    const {ax,ay}=anchors[R.dir]||anchors.se;
    const nx=Math.min(ax,world.x), ny=Math.min(ay,world.y);
    const nw=Math.abs(world.x-ax), nh=Math.abs(world.y-ay);
    const sx = (nw||0.0001)/bb0.w, sy=(nh||0.0001)/bb0.h;

    for(const [id,base] of R.items){
      const o=getById(id); if(!o) continue;
      const relx = (base.x - bb0.x); const rely = (base.y - bb0.y);
      o.x = nx + relx*sx; o.y = ny + rely*sy;
      o.w = base.w * sx; o.h = base.h * sy;
      if(base.points?.length){ o.points = base.points.map(p=>({ x: nx + (p.x - bb0.x)*sx, y: ny + (p.y - bb0.y)*sy })); }
    }
  }

  /* ===== Zoom / Fit ===== */
  function setZoom(scale, px=canvas.width/2, py=canvas.height/2){
    const before=screenToWorld(px,py);
    state.camera.scale = clamp(scale, 0.1, 6);
    const after=screenToWorld(px,py);
    state.camera.x += (before.x-after.x);
    state.camera.y += (before.y-after.y);
    state.camera.fit=false;
  }
  function fitPage(setFlag=true){
    const p=currPage(), pad=40;
    const sx=(canvas.width-pad)/p.w, sy=(canvas.height-pad)/p.h;
    const s=clamp(Math.min(sx,sy),0.1,6);
    state.camera.scale=s;
    const tl=worldToScreen(0,0), br=worldToScreen(p.w,p.h);
    const dx=((canvas.width - (br.x - tl.x))/2) - tl.x;
    const dy=((canvas.height - (br.y - tl.y))/2) - tl.y;
    state.camera.x -= dx/state.camera.scale;
    state.camera.y -= dy/state.camera.scale;
    if(setFlag) state.camera.fit=true;
    drawPageFrame();
  }

  /* ===== Events ===== */
  const preventMMB = el => { el.addEventListener('mousedown', e=>{ if(e.button===1){ e.preventDefault(); }}); el.addEventListener('auxclick', e=>{ if(e.button===1){ e.preventDefault(); }}); };
  preventMMB(canvasPanel); preventMMB($('#layerList'));

  // mouse wheel zoom to cursor
  canvasPanel.addEventListener('wheel', (e)=>{
    e.preventDefault();
    const rect=canvasPanel.getBoundingClientRect();
    const px=e.clientX-rect.left, py=e.clientY-rect.top;
    const factor = e.deltaY>0 ? 1/1.1 : 1.1;
    setZoom(state.camera.scale*factor, px, py);
  }, {passive:false});

  // Settings dialog
  bind('#btnSettings','click', ()=>{
    const HK={...defaultsHK, ...(state.settings?.hotkeys||{})}, t=state.theme;
    const ids=['hkSave','hkUndo','hkRedo','hkBrush','hkErase','hkLine','hkRect','hkCircle','hkStar','hkSelect','hkHatch','hkBucket','hkCrop','hkText'];
    const vals=[HK.save,HK.undo,HK.redo,HK.brush,HK.erase,HK.line,HK.rect,HK.circle,HK.star,HK.select,HK.hatch,HK.bucket,HK.crop,HK.text];
    ids.forEach((id,i)=>{ const el=$('#'+id); if(el) el.value=vals[i]; });
    $('#th1').value=t.accent1; $('#th2').value=t.accent2; $('#th3').value=t.accent3; $('#th4').value=t.accent4; $('#th5').value=t.accent5;
    updateHotkeySummary(HK); $('#dlgSettings').showModal();
  });
  bind('#btnSettingsClose','click', ()=> $('#dlgSettings').close());
  const captureKeyIn = el => el.addEventListener('keydown', e=>{
    e.preventDefault(); const parts=[]; if(e.ctrlKey) parts.push('Ctrl'); if(e.altKey) parts.push('Alt'); if(e.shiftKey) parts.push('Shift');
    const key = e.key.length===1 ? e.key.toUpperCase() : e.key; if(!['Control','Alt','Shift'].includes(key)) parts.push(key); el.value=parts.join('+');
  });
  ['#hkSave','#hkUndo','#hkRedo','#hkBrush','#hkErase','#hkLine','#hkRect','#hkCircle','#hkStar','#hkSelect','#hkHatch','#hkBucket','#hkCrop','#hkText'].forEach(sel=>{ const el=$(sel); el && captureKeyIn(el); });
  function updateHotkeySummary(HK=state.settings.hotkeys){
    $('#hkSummary').innerHTML = `Save: <b>${HK.save}</b> • Undo: <b>${HK.undo}</b> • Redo: <b>${HK.redo}</b><br>
      Brush: <b>${HK.brush}</b> • Erase: <b>${HK.erase}</b> • Line: <b>${HK.line}</b> • Rect: <b>${HK.rect}</b> • Circle: <b>${HK.circle}</b> • Star: <b>${HK.star}</b> • Select: <b>${HK.select}</b> • Hatch: <b>${HK.hatch}</b> • Bucket: <b>${HK.bucket}</b> • Crop: <b>${HK.crop}</b> • Text: <b>${HK.text}</b>`;
  }
  bind('#btnSaveSettings','click', ()=>{
    const HK=state.settings.hotkeys || (state.settings.hotkeys={...defaultsHK}), t=state.theme;
    Object.assign(HK, {
      save:$('#hkSave').value||HK.save, undo:$('#hkUndo').value||HK.undo, redo:$('#hkRedo').value||HK.redo,
      brush:$('#hkBrush').value||HK.brush, erase:$('#hkErase').value||HK.erase, line:$('#hkLine').value||HK.line, rect:$('#hkRect').value||HK.rect, circle:$('#hkCircle').value||HK.circle, star:$('#hkStar').value||HK.star, select:$('#hkSelect').value||HK.select, hatch:$('#hkHatch').value||HK.hatch, bucket:$('#hkBucket').value||HK.bucket, crop:$('#hkCrop').value||HK.crop, text:$('#hkText').value||HK.text
    });
    state.theme.accent1=$('#th1').value; state.theme.accent2=$('#th2').value; state.theme.accent3=$('#th3').value; state.theme.accent4=$('#th4').value; state.theme.accent5=$('#th5').value;
    LS.set(SETTINGS_KEY,state.settings); LS.set(THEME_KEY,state.theme); applyTheme(); $('#saveKbd').textContent=HK.save; updateHotkeySummary(); $('#dlgSettings').close();
  });

  // mini palette bank/page controls (left)
  bind('#bankA','click', ()=>{ state.activeBank=0; buildPaletteBar(); buildPalBanksUI(); });
  bind('#bankB','click', ()=>{ state.activeBank=1; buildPaletteBar(); buildPalBanksUI(); });
  bind('#bankC','click', ()=>{ state.activeBank=2; buildPaletteBar(); buildPalBanksUI(); });
  bind('#palPrev','click', ()=>{ state.activePage[state.activeBank]=Math.max(0,(state.activePage[state.activeBank]||0)-1); buildPaletteBar(); buildPalBanksUI(); });
  bind('#palNext','click', ()=>{ state.activePage[state.activeBank]=Math.min(2,(state.activePage[state.activeBank]||0)+1); const b=state.palBanks[state.activeBank]; if(!b.pages[state.activePage[state.activeBank]]) b.pages[state.activePage[state.activeBank]]=placeholderPalette(); buildPaletteBar(); buildPalBanksUI(); });

  // project IO
  bind('#btnNew','click', ()=>{ $('#newTitle').placeholder = `project_${state.projectCounter}`; $('#dlgNew').showModal(); });
  bind('#btnNewClose','click', ()=> $('#dlgNew').close());
  bind('#btnCreateProj','click', ()=>{
    let t=$('#newTitle').value.trim();
    if(!t){ t=`project_${state.projectCounter}`; state.projectCounter++; LS.set(PROJECT_COUNT_KEY,state.projectCounter); }
    const w=+$('#newW').value||1024; const h=+$('#newH').value||576;
    state.project=newProject(t,w,h); state.pageIndex=0; state.layer=0; state.selection.clear();
    state.palBanks = ensurePalBanks(state.project[PROJ_BANKS_KEY]);
    refreshUIAfterProjectChange(); pushHistory(); $('#dlgNew').close();
  });
  bind('#btnLoad','click', ()=> $('#dlgLoad').showModal());
  bind('#btnLoadClose','click', ()=> $('#dlgLoad').close());
  bind('#btnDoLoad','click', async ()=>{
    const f=$('#loadFile').files[0]; if(!f) return alert('Pick a file');
    try{
      const proj=JSON.parse(await f.text()); if(!proj.pages) throw new Error('Invalid project file'); proj.layers=normalizeLayers(proj.layers);
      state.project=proj; state.pageIndex=0; state.layer=proj.layers[0]?.id??0; state.selection.clear();
      state.palBanks = ensurePalBanks(proj[PROJ_BANKS_KEY]);
      refreshUIAfterProjectChange(); pushHistory(); $('#dlgLoad').close();
    }catch(err){ alert('Failed to load project: '+err.message); }
  });
  bind('#btnSave','click', ()=>{
    if(!state.project) return alert('No project');
    // persist banks into project before save
    state.project[PROJ_BANKS_KEY] = deepClone(state.palBanks);
    downloadFile((state.project.title||'phik')+'.project.json', JSON.stringify(state.project,null,2));
  });
  bind('#btnExport','click', ()=>{ if(!state.project) return alert('No project'); const html=makeExportHTML(state.project); downloadFile((state.project.title||'comic')+'.html', html); });
  bind('#btnFont','click', ()=> $('#fontFile').click());
  bind('#fontFile','change', e=>{
    const file=e.target.files[0]; if(!file) return;
    const name=file.name.replace(/\.[^/.]+$/,'').replace(/\W+/g,'_');
    const url=URL.createObjectURL(file);
    const style=document.createElement('style');
    style.textContent=`@font-face{font-family:'${name}';src:url('${url}');}`;
    document.head.appendChild(style);
    state.fonts.push(name);
    buildFontSelect($('#textFont'));
    buildFontSelect($('#toolTextFont'));
    state.textFont=name;
    $('#textFont').value=name;
    $('#toolTextFont')?.value=name;
    e.target.value='';
    toast('Loaded font '+file.name);
  });

  // undo/redo buttons
  bind('#btnUndo','click', undo); bind('#btnRedo','click', redo);
  // snap controls
  bind('#snapToggle','change', e=>{ state.snap.enabled = e.target.checked; });
  bind('#snapSize','change', e=>{ const v=Math.max(1,+e.target.value||1); state.snap.size = v; });

  // tools
  bind('#uiToggle','click', ()=> document.body.classList.toggle('ui-hidden'));

  // page controls
  bind('#btnNext','click', ()=>{ if(!state.project) return; state.pageIndex=(state.pageIndex+1)%state.project.pages.length; refreshUIAfterProjectChange({preserveCamera:true}); });
  bind('#btnPrev','click', ()=>{ if(!state.project) return; state.pageIndex=(state.pageIndex-1+state.project.pages.length)%state.project.pages.length; refreshUIAfterProjectChange({preserveCamera:true}); });
  bind('#btnApplySize','click', ()=>{ const w=+$('#pageW').value||currPage().w; const h=+$('#pageH').value||currPage().h; currPage().w=w; currPage().h=h; resizeCanvas(); pushHistory(); });
  bind('#btnNewPage','click', ()=>{ const last=currPage(); state.project.pages.push(newPage(last.w,last.h)); state.pageIndex=state.project.pages.length-1; refreshUIAfterProjectChange({preserveCamera:true}); pushHistory(); });
  bind('#btnDupPage','click', ()=>{ const cp=currPage(); const dup=deepClone(cp); dup.objects.forEach(o=>o.id=uid()); state.project.pages.push(dup); state.pageIndex=state.project.pages.length-1; refreshUIAfterProjectChange({preserveCamera:true}); pushHistory(); });
  bind('#btnDelPage','click', ()=>{ if(state.project.pages.length<=1) return alert('At least one page required.'); if(!confirm('Delete current page?')) return; state.project.pages.splice(state.pageIndex,1); state.pageIndex=Math.max(0,state.pageIndex-1); refreshUIAfterProjectChange({preserveCamera:true}); pushHistory(); });
  bind('#pageBg','input',  ()=>{ currPage().bgColor=$('#pageBg').value; });
  bind('#pageBgTransparent','change', ()=>{ currPage().bgTransparent = $('#pageBgTransparent').value==='true'; });
  bind('#pageNote','input', ()=>{ currPage().note=$('#pageNote').value; });

  // selection actions
  function deleteSelection(){
    if(!state.selection.size) return;
    const ids=[...state.selection];
    currPage().objects = currPage().objects.filter(o=>!ids.includes(o.id));
    state.selection.clear();
    pushHistory();
  }
  bind('#btnDeleteSel','click', deleteSelection);
  bind('#btnCombineSel','click', ()=>{ if(state.selection.size<2) return; const items=[...state.selection].map(getById).filter(Boolean); const minx=Math.min(...items.map(o=>o.x)), miny=Math.min(...items.map(o=>o.y)); const maxx=Math.max(...items.map(o=>o.x+o.w)), maxy=Math.max(...items.map(o=>o.y+o.h)); const group=newObj('group',{x:minx,y:miny,w:maxx-minx,h:maxy-miny, meta:{children:items.map(i=>i.id)}}); currPage().objects.push(group); state.selection=new Set([group.id]); pushHistory(); });
  bind('#btnToLayer','click', ()=>{ if(!state.selection.size) return; const L=prompt('Send selection to which layer id?', state.layer); if(L===null) return; for(const id of state.selection){ const o=getById(id); if(o) o.layer=+L; } pushHistory(); });
  bind('#btnSaveAsPanel','click', ()=>{ const items=[...state.selection].map(getById).filter(Boolean).filter(o=>o.kind==='panel'); if(!items.length) return alert('Select panels first'); const name=prompt('Template name?','tpl-'+uid()); if(!name) return; state.templates.push({name,items:items.map(deepClone)}); LS.set(TEMPLATES_KEY,state.templates); buildLibrary(); alert('Saved to panel_templates'); });
  bind('#btnSaveAsStamp','click', ()=>{ const items=[...state.selection].map(getById).filter(Boolean); if(!items.length) return alert('Select items first'); const name=prompt('Stamp name?','stamp-'+uid()); if(!name) return; state.stamps.push({name,items:items.map(deepClone)}); LS.set(STAMPS_KEY,state.stamps); buildLibrary(); alert('Saved to stamps'); });
  bind('#btnSelectAll','click', ()=>{ const vis=new Set(state.project.layers.filter(l=>l.visible!==false).map(l=>l.id)); state.selection = new Set(allObjects().filter(o=>vis.has(o.layer)).map(o=>o.id)); });
  bind('#btnDeselect','click', ()=> state.selection.clear());
  bind('#rotRange','input', e=>{ for(const id of state.selection){ const o=getById(id); if(o) o.r=+e.target.value; } });

  // universal selection style controls
  bind('#selStroke','input', e=>{ for(const id of state.selection){ const o=getById(id); if(o){ o.stroke=e.target.value; } } });
  bind('#selFill','input', e=>{ for(const id of state.selection){ const o=getById(id); if(o){ o.fill=e.target.value; } } });
  bind('#selStrokeW','input', e=>{ const v=+e.target.value; for(const id of state.selection){ const o=getById(id); if(o){ o.strokeWidth=v; } } });
  bind('#selOpacity','input', e=>{ const v=clamp(+e.target.value,1,100)/100; for(const id of state.selection){ const o=getById(id); if(o){ o.opacity=v; } } });

  // zoom buttons
  bind('#btnZoomIn','click', ()=> setZoom(state.camera.scale*1.2));
  bind('#btnZoomOut','click', ()=> setZoom(state.camera.scale/1.2));
  bind('#btnZoom100','click', ()=> { setZoom(1); });
  bind('#btnZoomFit','click', ()=> fitPage(true));

  /* ===== Canvas interactions ===== */
  function endInteractions(commitHist=true){
    state.mouse.down=false; state.draggingSel=false; state.resizing=null; state.marquee=null; state.panning=false;
    state.draw.hatchPreview=null; state.draw.hatchOrigin=null;
    if(commitHist) pushHistory();
  }
  window.addEventListener('mouseup', ()=> endInteractions(true));

  canvas.addEventListener('mousedown', e=>{
    if(!state.project) return;
    const rect=canvasPanel.getBoundingClientRect();
    state.mouse.button=e.button;
    state.mouse.x=e.clientX-rect.left; state.mouse.y=e.clientY-rect.top; state.mouse.ox=state.mouse.x; state.mouse.oy=state.mouse.y; state.mouse.down=true;
    let world=snapPoint(screenToWorld(state.mouse.x,state.mouse.y));

    if(e.button===1){
      if(!(state.tool==='draw-hatch')){
        state.panning=true; state.panStart={x:e.clientX,y:e.clientY,cx:state.camera.x,cy:state.camera.y}; return;
      } else {
        state.draw.hatchGuide={a:world,b:world}; return;
      }
    }

    if(state.editingText){ commitTextEdit(); }

    // handles (z-index fixed)
    if(state.tool.endsWith('select') && state.selection.size){
      const bb=selectionBBox();
      if(bb){
        const grips=[
          {dir:'nw',x:bb.x,y:bb.y},{dir:'ne',x:bb.x+bb.w,y:bb.y},{dir:'sw',x:bb.x,y:bb.y+bb.h},{dir:'se',x:bb.x+bb.w,y:bb.y+bb.h},
          {dir:'move',x:bb.x+bb.w/2,y:bb.y+bb.h/2}
        ];
        for(const g of grips){
          const sp=worldToScreen(g.x,g.y);
          if(Math.abs(sp.x-state.mouse.x)<=8 && Math.abs(sp.y-state.mouse.y)<=8){
            if(g.dir==='move'){ state.draggingSel=true; state.dragStartPositions.clear(); for(const id of state.selection){ const o=getById(id); if(o) state.dragStartPositions.set(id,{x:o.x,y:o.y}); } }
            else { startResize(g.dir); }
            return;
          }
        }
      }
    }

    if(state.tool.startsWith('panel-')){
      if(state.tool==='panel-select'){
        const hit=hitTest(world,'panel');
        if(hit){ if(state.selection.has(hit.id)) state.selection.delete(hit.id); else state.selection.add(hit.id);
          state.draggingSel=true; state.dragStartPositions.clear(); for(const id of state.selection){ const o=getById(id); if(o) state.dragStartPositions.set(id,{x:o.x,y:o.y}); }
          if(hit.type==='text'){ beginTextEdit(hit); }
        } else { state.marquee={x:state.mouse.x,y:state.mouse.y,w:0,h:0}; state.selection.clear(); }
      }
      if(state.tool==='panel-rect'){
        const o=newObj('panel-rect',{x:world.x,y:world.y,w:1,h:1}); currPage().objects.push(o); state.selection=new Set([o.id]); state.drawing={id:o.id,start:world};
      }
      if(state.tool==='panel-line'){
        const o=newObj('line',{x:world.x,y:world.y, points:[world,world], stroke:state.currentColor, kind:'panel'}); o.kind='panel'; currPage().objects.push(o); state.drawing={id:o.id,start:world};
      }
      if(state.tool==='panel-star'){
        const hit=hitTest(world,'panel');
        if(hit?.type==='star'){ state.selection=new Set([hit.id]); showStarControls(hit); }
        else{
          const pts=+($('#starPoints')?.value||5), rin=+($('#starInner')?.value||10), rout=+($('#starOuter')?.value||10);
          const star=buildStar(world.x,world.y,pts,rin,rout); star.kind='panel'; currPage().objects.push(star);
          state.selection=new Set([star.id]); state.drawing={starId:star.id, center:{x:world.x,y:world.y}}; showStarControls(star);
        }
      }
    } else {
      if(state.tool==='draw-select'){
        const hit=hitTest(world,'draw');
        if(hit){ if(state.selection.has(hit.id)) state.selection.delete(hit.id); else state.selection.add(hit.id);
          state.draggingSel=true; state.dragStartPositions.clear(); for(const id of state.selection){ const o=getById(id); if(o) state.dragStartPositions.set(id,{x:o.x,y:o.y}); }
          if(hit.type==='text'){ beginTextEdit(hit); }
        } else { state.marquee={x:state.mouse.x,y:state.mouse.y,w:0,h:0}; state.selection.clear(); }
      }
      if(state.tool==='draw-erase'){
        const hit=hitTest(world,'draw');
        if(hit && hit.kind==='draw'){
          currPage().objects = currPage().objects.filter(o=>o.id!==hit.id);
          pushHistory();
          endInteractions(false);
        } else if(!hit){
          state.marquee={x:state.mouse.x,y:state.mouse.y,w:0,h:0, erase:true};
        }
        return;
      }
      if(state.tool==='draw-brush'){
        const p=newObj('path',{x:world.x,y:world.y, points:[world], stroke: state.currentColor, strokeWidth: +($('#sizeBrush')?.value || state.strokeWidth), meta:{closed:false}});
        p.fill='#00000000'; currPage().objects.push(p); state.drawing={id:p.id};
      }
      if(state.tool==='draw-line'){
        const o=newObj('line',{x:world.x,y:world.y, points:[world,world], stroke:state.currentColor}); currPage().objects.push(o); state.drawing={id:o.id,start:world};
      }
      if(state.tool==='draw-rect'){
        const o=newObj('rect',{x:world.x,y:world.y,w:1,h:1, stroke:state.currentColor, fill:state.fill}); currPage().objects.push(o); state.drawing={id:o.id,start:world};
      }
      if(state.tool==='draw-circle'){
        const o=newObj('circle',{x:world.x,y:world.y,w:2,h:2, stroke:state.currentColor, fill:state.fill, meta:{center:{x:world.x,y:world.y}}}); currPage().objects.push(o); state.drawing={id:o.id,center:world};
      }
      if(state.tool==='draw-ellipse'){
        const o=newObj('ellipse',{x:world.x,y:world.y,w:1,h:1, stroke:state.currentColor, fill:state.fill}); currPage().objects.push(o); state.drawing={id:o.id,start:world};
      }
      if(state.tool==='draw-star'){
        const pts=+($('#starPoints')?.value||5), rin=+($('#starInner')?.value||10), rout=+($('#starOuter')?.value||10);
        const star=buildStar(world.x,world.y,pts,rin,rout); currPage().objects.push(star);
        state.selection=new Set([star.id]); state.drawing={starId:star.id, center:{x:world.x,y:world.y}}; showStarControls(star);
      }
      if(state.tool==='draw-bucket'){
        const hit=hitTest(world,'draw');
        if(hit){ recolorObject(hit, state.currentColor); pushHistory(); }
        else { state.bucketBox={x:state.mouse.x,y:state.mouse.y,w:0,h:0}; }
      }
      if(state.tool==='draw-crop'){ state.marquee={x:state.mouse.x,y:state.mouse.y,w:0,h:0, crop:true}; }
      if(state.tool==='draw-text'){
        const hit=hitTest(world,'draw');
        if(hit?.type==='text'){ state.selection=new Set([hit.id]); beginTextEdit(hit); return; }
        const txt=($('#toolTextContent')?.value||'Text'); const size=+($('#toolTextSize')?.value||24); const fam=$('#toolTextFont')?.value||state.textFont;
        state.textFont=fam;
        const o=newObj('text',{x:world.x,y:world.y,w:1,h:size, stroke:state.currentColor, meta:{text:txt,size:size,family:fam, fill:state.currentColor, outline:false, outlineColor:'#000', outlineWidth:2}}); currPage().objects.push(o); state.selection=new Set([o.id]); beginTextEdit(o);
      }
      if(state.tool==='draw-hatch'){ state.drawing={hatchStart:world}; state.draw.hatchOrigin={x:world.x,y:world.y}; state.draw.hatchPreview={r:0}; }
    }
  });

  canvas.addEventListener('mousemove', e=>{
    const rect=canvasPanel.getBoundingClientRect();
    const sx=e.clientX-rect.left, sy=e.clientY-rect.top;
    state.mouse.x=sx; state.mouse.y=sy;
    let world=snapPoint(screenToWorld(state.mouse.x,state.mouse.y));
    if(!state.mouse.down) return;

    if(state.panning){
      const dx=(e.clientX-state.panStart.x)/state.camera.scale;
      const dy=(e.clientY-state.panStart.y)/state.camera.scale;
      state.camera.x = state.panStart.cx - dx;
      state.camera.y = state.panStart.cy - dy;
      return;
    }
    if(state.draw.hatchGuide){ state.draw.hatchGuide.b=world; return; }
    if(state.resizing){ applyResize(world); return; }

    if(state.tool.startsWith('panel-')){
      if(state.tool==='panel-select' && state.marquee){
        state.marquee.w=state.mouse.x-state.mouse.ox; state.marquee.h=state.mouse.y-state.mouse.oy; selectInMarquee_intersect(false,'panel');
      }
      if(state.tool==='panel-select' && state.draggingSel){
        const dx=(state.mouse.x - state.mouse.ox)/state.camera.scale;
        const dy=(state.mouse.y - state.mouse.oy)/state.camera.scale;
        for(const id of state.selection){
          const o=getById(id); if(!o) continue; const start=state.dragStartPositions.get(id);
          let nx=start.x+dx, ny=start.y+dy;
          if(state.snap.enabled){ nx=snapValue(nx); ny=snapValue(ny); }
          o.x=nx; o.y=ny;
        }
      }
      if(state.tool==='panel-rect' && state.drawing){
        const o=getById(state.drawing.id); if(!o) return;
        o.w = (world.x - state.drawing.start.x); o.h=(world.y - state.drawing.start.y);
      }
      if(state.tool==='panel-line' && state.drawing){
        const o=getById(state.drawing.id); if(!o) return;
        let dx=world.x - state.drawing.start.x, dy=world.y - state.drawing.start.y;
        if(e.shiftKey){ if(Math.abs(dx)>=Math.abs(dy)){ dy=0; } else { dx=0; } }
        o.points[1]={x: state.drawing.start.x + dx, y: state.drawing.start.y + dy};
        o.x=Math.min(o.points[0].x,o.points[1].x);
        o.y=Math.min(o.points[0].y,o.points[1].y);
        o.w=Math.abs(o.points[1].x-o.points[0].x);
        o.h=Math.abs(o.points[1].y-o.points[0].y);
      }
      if(state.tool==='panel-star' && state.drawing?.starId){
        const st=getById(state.drawing.starId); if(!st) return;
        const c=state.drawing.center;
        const rout=Math.hypot(world.x-c.x, world.y-c.y); const rin=rout*0.5;
        st.meta.outer=rout; st.meta.inner=rin; st.points = starPointsFromMeta(st.meta);
        const minx=Math.min(...st.points.map(p=>p.x)), miny=Math.min(...st.points.map(p=>p.y));
        const maxx=Math.max(...st.points.map(p=>p.x)), maxy=Math.max(...st.points.map(p=>p.y));
        st.x=minx; st.y=miny; st.w=maxx-minx; st.h=maxy-miny;
      }
    } else {
      if(state.tool==='draw-select' && state.marquee){
        state.marquee.w=state.mouse.x-state.mouse.ox; state.marquee.h=state.mouse.y-state.mouse.oy; selectInMarquee_intersect(false,'draw');
      }
      if(state.tool==='draw-select' && state.draggingSel){
        const dx=(state.mouse.x - state.mouse.ox)/state.camera.scale;
        const dy=(state.mouse.y - state.mouse.oy)/state.camera.scale;
        for(const id of state.selection){
          const o=getById(id); if(!o) continue; const start=state.dragStartPositions.get(id);
          let nx=start.x+dx, ny=start.y+dy;
          if(state.snap.enabled){ nx=snapValue(nx); ny=snapValue(ny); }
          o.x=nx; o.y=ny;
        }
      }
      if(state.tool==='draw-erase' && state.marquee && state.marquee.erase){ state.marquee.w=state.mouse.x-state.mouse.ox; state.marquee.h=state.mouse.y-state.mouse.oy; return; }
      if(state.tool==='draw-brush' && state.drawing){
        const o=getById(state.drawing.id); if(!o) return;
        const last=o.points[o.points.length-1]; if(dist(last,world)>0.25){ o.points.push(world); }
      }
      if(state.tool==='draw-line' && state.drawing){
        const o=getById(state.drawing.id); if(!o) return;
        let dx=world.x - state.drawing.start.x, dy=world.y - state.drawing.start.y;
        if(e.shiftKey){ if(Math.abs(dx)>=Math.abs(dy)){ dy=0; } else { dx=0; } }
        o.points[1]={x: state.drawing.start.x + dx, y: state.drawing.start.y + dy};
        o.x=Math.min(o.points[0].x,o.points[1].x);
        o.y=Math.min(o.points[0].y,o.points[1].y);
        o.w=Math.abs(o.points[1].x-o.points[0].x);
        o.h=Math.abs(o.points[1].y-o.points[0].y);
      }
      if(state.tool==='draw-rect' && state.drawing){
        const o=getById(state.drawing.id); if(!o) return;
        o.w=(world.x - state.drawing.start.x); o.h=(world.y - state.drawing.start.y);
      }
      if(state.tool==='draw-circle' && state.drawing){
        const o=getById(state.drawing.id); if(!o) return;
        const c=o.meta.center; const r=Math.hypot(world.x-c.x, world.y-c.y);
        o.x=c.x-r; o.y=c.y-r; o.w=r*2; o.h=r*2;
      }
      if(state.tool==='draw-ellipse' && state.drawing){
        const o=getById(state.drawing.id); if(!o) return;
        o.w=(world.x - state.drawing.start.x); o.h=(world.y - state.drawing.start.y);
      }
      if(state.tool==='draw-star' && state.drawing?.starId){
        const st=getById(state.drawing.starId); if(!st) return;
        const c=state.drawing.center;
        const rout=Math.hypot(world.x-c.x, world.y-c.y); const rin=rout*0.5;
        st.meta.outer=rout; st.meta.inner=rin; st.points=starPointsFromMeta(st.meta);
        const minx=Math.min(...st.points.map(p=>p.x)), miny=Math.min(...st.points.map(p=>p.y));
        const maxx=Math.max(...st.points.map(p=>p.x)), maxy=Math.max(...st.points.map(p=>p.y));
        st.x=minx; st.y=miny; st.w=maxx-minx; st.h=maxy-miny;
      }
      if(state.tool==='draw-bucket' && state.bucketBox){
        state.bucketBox.w=state.mouse.x-state.mouse.ox; state.bucketBox.h=state.mouse.y-state.mouse.oy;
      }
      if(state.tool==='draw-crop' && state.marquee){ state.marquee.w=state.mouse.x-state.mouse.ox; state.marquee.h=state.mouse.y-state.mouse.oy; }
      if(state.tool==='draw-hatch' && state.draw.hatchPreview && state.draw.hatchOrigin){
        const r = Math.hypot(world.x-state.draw.hatchOrigin.x, world.y-state.draw.hatchOrigin.y);
        state.draw.hatchPreview.r = r;
      }
    }
  });

  canvas.addEventListener('mouseup', e=>{
    let world=snapPoint(screenToWorld(state.mouse.x,state.mouse.y));

    if(state.draw.hatchGuide){
      const a=state.draw.hatchGuide.a, b=state.draw.hatchGuide.b;
      const angRad=Math.atan2(b.y-a.y, b.x-a.x);
      state.draw.hatchAngle = Math.round((angRad*180/Math.PI)*100)/100;
      const inp=$('#hatchAngle'); if(inp) inp.value=state.draw.hatchAngle;
      state.draw.hatchGuide=null;
    }

    if(state.tool.startsWith('panel-')){
      if(state.tool==='panel-rect' && state.drawing){
        const o=getById(state.drawing.id); if(o){ if(o.w<0){o.x+=o.w;o.w*=-1;} if(o.h<0){o.y+=o.h;o.h*=-1;} } state.drawing=null;
      }
      if(state.tool==='panel-line' && state.drawing){ state.drawing=null; }
      if(state.tool==='panel-star' && state.drawing?.starId){ state.drawing=null; }
    } else {
      if(state.tool==='draw-erase' && state.marquee?.erase){
        const xm=Math.min(state.marquee.x,state.marquee.x+state.marquee.w);
        const ym=Math.min(state.marquee.y,state.marquee.y+state.marquee.h);
        const wm=Math.abs(state.marquee.w), hm=Math.abs(state.marquee.h);
        const tl=screenToWorld(xm,ym), br=screenToWorld(xm+wm,ym+hm);
        const box={x:tl.x,y:tl.y,w:br.x-tl.x,h:br.y-tl.y};
        currPage().objects = currPage().objects.filter(o=> !(o.kind==='draw' && rectsIntersect(aabb(o), box)) );
        state.marquee=null;
      }
      if(state.tool==='draw-bucket' && state.bucketBox){
        const xm=Math.min(state.bucketBox.x,state.bucketBox.x+state.bucketBox.w);
        const ym=Math.min(state.bucketBox.y,state.bucketBox.y+state.bucketBox.h);
        const wm=Math.abs(state.bucketBox.w), hm=Math.abs(state.bucketBox.h);
        const tl=screenToWorld(xm,ym), br=screenToWorld(xm+wm,ym+hm);
        const box={x:tl.x,y:tl.y,w:br.x-tl.x,h:br.y-tl.y};
        for(const o of allObjects()){
          if(o.kind!=='draw') continue;
          if(rectsIntersect(aabb(o), box)) recolorObject(o, state.currentColor);
        }
        state.bucketBox=null; pushHistory();
      }
      if(state.tool==='draw-star' && state.drawing?.starId){ state.drawing=null; }
      if(state.tool==='draw-hatch' && state.draw.hatchOrigin && state.draw.hatchPreview){
        const o = state.draw.hatchOrigin; const r=state.draw.hatchPreview.r;
        const minx=o.x-r, maxx=o.x+r, miny=o.y-r, maxy=o.y+r;
        const ang=(+($('#hatchAngle')?.value||state.draw.hatchAngle))*Math.PI/180;
        const gap=+($('#hatchGap')?.value||state.draw.hatchGap);
        const diag=r*2*Math.SQRT2;
        for(let d=-diag; d<diag; d+=gap){
          const x1=o.x + d*Math.cos(ang), y1=o.y + d*Math.sin(ang);
          const x2=x1 + (maxx-minx)*Math.sin(ang+Math.PI/2), y2=y1 - (maxx-minx)*Math.cos(ang+Math.PI/2);
          const line=newObj('line',{x:Math.min(x1,x2), y:Math.min(y1,y2), points:[{x:x1,y:y1},{x:x2,y:y2}], stroke: state.currentColor, fill:'#00000000', strokeWidth:1, kind:'draw'});
          currPage().objects.push(line);
        }
        state.drawing=null; state.draw.hatchPreview=null; state.draw.hatchOrigin=null; pushHistory();
      }
      if(state.tool==='draw-crop' && state.marquee){
        const xm=Math.min(state.marquee.x,state.marquee.x+state.marquee.w);
        const ym=Math.min(state.marquee.y,state.marquee.y+state.marquee.h);
        const wm=Math.abs(state.marquee.w), hm=Math.abs(state.marquee.h);
        const tl=screenToWorld(xm,ym), br=screenToWorld(xm+wm,ym+hm);
        const box={x:tl.x,y:tl.y,w:br.x-tl.x,h:br.y-tl.y};

        if(!state.cropMask){ // first step
          state.cropMask=box; drawCropMask();
        } else { // second step: keep inside "box", delete (mask - box)
          const mask=state.cropMask;
          currPage().objects = currPage().objects.filter(o=>{
            if(o.kind!=='draw') return true;
            const inMask = rectsIntersect(aabb(o), mask);
            const inBox  = rectsIntersect(aabb(o), box);
            if(inMask && !inBox) return false; // delete the ring
            if(inBox && (o.type==='line' || o.type==='path') && o.points?.length){
              o.points = o.points.map(p=>({ x: clamp(p.x, box.x, box.x+box.w), y: clamp(p.y, box.y, box.y+box.h) }));
              const minx=Math.min(...o.points.map(p=>p.x)), miny=Math.min(...o.points.map(p=>p.y));
              const maxx=Math.max(...o.points.map(p=>p.x)), maxy=Math.max(...o.points.map(p=>p.y));
              o.x=minx; o.y=miny; o.w=maxx-minx; o.h=maxy-miny;
            }
            return true;
          });
          state.cropMask=null; drawCropMask();
        }
        state.marquee=null; pushHistory();
      }
      if(state.drawing && state.tool.startsWith('draw-')){
        const o=getById(state.drawing.id);
        if(state.tool==='draw-brush' && o){
          const pts=o.points||[];
          const minx=Math.min(...pts.map(p=>p.x)), miny=Math.min(...pts.map(p=>p.y));
          const maxx=Math.max(...pts.map(p=>p.x)), maxy=Math.max(...pts.map(p=>p.y));
          o.x=minx; o.y=miny; o.w=maxx-minx; o.h=maxy-miny;
          const first=pts[0], last=pts[pts.length-1];
          if(first && last && dist(first,last)<5) o.meta.closed=true;
        } else if(o){
          if(o.w<0){o.x+=o.w;o.w*=-1;}
          if(o.h<0){o.y+=o.h;o.h*=-1;}
        }
        state.drawing=null;
      }
    }

    const only = (type)=> state.selection.size===1 && getById([...state.selection][0])?.type===type;
    updateTextControls();
    $('#starControls').style.display = (only('star')) ? 'grid':'none';
    $('#imgControls').style.display = (only('image')) ? 'grid':'none';
    if(only('image')){
      const im=getById([...state.selection][0]);
      $('#imgOffL').value=im.meta.left||0;
      $('#imgOffR').value=im.meta.right||0;
      $('#imgOffT').value=im.meta.top||0;
      $('#imgOffB').value=im.meta.bottom||0;
    }

    endInteractions(true);
  });

  // click empty clears selection (when not dragging)
  canvas.addEventListener('click', e=>{
    if(state.mouse.button!==0) return;
    if(Math.hypot(state.mouse.x-state.mouse.ox, state.mouse.y-state.mouse.oy) < 2 && !state.resizing && !state.draggingSel){
      const world=snapPoint(screenToWorld(state.mouse.x,state.mouse.y));
      const hit=hitTest(world);
      if(!hit) state.selection.clear();
    }
  });

  /* ===== Text inline editing & controls ===== */
  function beginTextEdit(o){
    const s=worldToScreen(o.x,o.y);
    textEditor.style.left=s.x+'px'; textEditor.style.top=s.y+'px';
    textEditor.style.display='block';
    textEditor.style.minWidth=Math.max(40, o.w*state.camera.scale)+'px';
    textEditor.style.minHeight=Math.max(20, o.h*state.camera.scale)+'px';
    textEditor.textContent=o.meta.text||'';
    state.editingText={id:o.id};
    textEditor.focus();
    updateTextControls();
  }
  function commitTextEdit(){
    if(!state.editingText) return;
    const o=getById(state.editingText.id); if(o){ o.meta.text=textEditor.textContent; o.h=o.meta.size||24; }
    textEditor.style.display='none'; textEditor.textContent='';
    state.editingText=null;
    pushHistory();
  }
  textEditor.addEventListener('keydown', e=>{
    if(e.key==='Enter' && e.ctrlKey){ e.preventDefault(); commitTextEdit(); } // Enter alone inserts newline
  });
  textEditor.addEventListener('blur', commitTextEdit);

  function updateTextControls(){
    const onlyText = state.selection.size===1 && getById([...state.selection][0])?.type==='text';
    $('#textControls').style.display = onlyText ? 'flex':'none';
    if(onlyText){
      const t=getById([...state.selection][0]);
      buildFontSelect($('#textFont'));
      $('#textContent').value=t.meta.text||'Text';
      $('#textSize').value=t.meta.size||24;
      state.textFont=t.meta.family||state.textFont;
      $('#textFont').value=state.textFont;
      $('#textFill').value=t.meta.fill||t.stroke||'#e8eaf0';
      $('#textOutline').value=t.meta.outline?'on':'off';
      $('#textOutlineColor').value=t.meta.outlineColor||'#000000';
      $('#textOutlineWidth').value=t.meta.outlineWidth||2;
    }
  }

  // sidebar text controls bindings
  bind('#textContent','input', e=>{ if(state.selection.size!==1) return; const o=getById([...state.selection][0]); if(o?.type!=='text') return; o.meta.text=e.target.value; });
  bind('#textSize','input', e=>{ if(state.selection.size!==1) return; const o=getById([...state.selection][0]); if(o?.type!=='text') return; o.meta.size=+e.target.value; o.h=o.meta.size; });
  bind('#textFont','change', e=>{ state.textFont=e.target.value; if(state.selection.size!==1) return; const o=getById([...state.selection][0]); if(o?.type!=='text') return; o.meta.family=e.target.value; });
  bind('#textFill','input', e=>{ if(state.selection.size!==1) return; const o=getById([...state.selection][0]); if(o?.type!=='text') return; o.meta.fill=e.target.value; });
  bind('#textOutline','change', e=>{ if(state.selection.size!==1) return; const o=getById([...state.selection][0]); if(o?.type!=='text') return; o.meta.outline=(e.target.value==='on'); });
  bind('#textOutlineColor','input', e=>{ if(state.selection.size!==1) return; const o=getById([...state.selection][0]); if(o?.type!=='text') return; o.meta.outlineColor=e.target.value; });
  bind('#textOutlineWidth','input', e=>{ if(state.selection.size!==1) return; const o=getById([...state.selection][0]); if(o?.type!=='text') return; o.meta.outlineWidth=+e.target.value; });

  ['#imgOffL','#imgOffR','#imgOffT','#imgOffB'].forEach(sel=>{
    bind(sel,'input', ()=>{
      if(state.selection.size!==1) return;
      const o=getById([...state.selection][0]);
      if(o?.type!=='image') return;
      const img=o.meta.img; if(!img) return;
      o.meta.left = clamp(+$('#imgOffL').value||0, 0, img.width);
      o.meta.right = clamp(+$('#imgOffR').value||0, 0, img.width);
      o.meta.top = clamp(+$('#imgOffT').value||0, 0, img.height);
      o.meta.bottom = clamp(+$('#imgOffB').value||0, 0, img.height);
      o.w = img.width - o.meta.left - o.meta.right;
      o.h = img.height - o.meta.top - o.meta.bottom;
    });
  });

  // global shortcuts for undo/redo and deleting selection
  addEventListener('keydown', e=>{
    const tag=(e.target.tagName||'').toLowerCase();
    if(tag==='input' || tag==='textarea' || e.target.isContentEditable) return;
    const ctrl=e.ctrlKey||e.metaKey;
    if(ctrl && e.key.toLowerCase()==='z'){
      e.preventDefault();
      if(e.shiftKey) redo(); else undo();
    } else if(ctrl && e.key.toLowerCase()==='y'){
      e.preventDefault();
      redo();
    } else if(e.key==='Delete' || e.key==='Backspace'){
      if(state.selection.size){ e.preventDefault(); deleteSelection(); }
    }
  });

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

  /* ===== Bucket recolor ===== */
  function recolorObject(o, color){
    if(['rect','circle','ellipse','star','panel-rect','path'].includes(o.type) && o.meta?.closed){ o.fill=color; }
    else if(['rect','circle','ellipse','star','panel-rect'].includes(o.type)){ o.fill=color; }
    else if(['line','path'].includes(o.type)){ o.stroke=color; }
    else if(o.type==='text'){ o.meta.fill=color; }
  }

  /* ===== Export ===== */
  function makeExportHTML(project){
    const PCLONE = deepClone(project);
    // prune palettes in export (not used by reader)
    delete PCLONE[PROJ_BANKS_KEY];
    const JSON_DATA=JSON.stringify(PCLONE);
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
<div class="bar"><button class="btn" id="prev">Prev</button><span class="badge" id="info"></span><button class="btn" id="next">Next</button></div>
<script>
  const project=${JSON_DATA}; let i=0;
  const c=document.getElementById('c'), ctx=c.getContext('2d'), info=document.getElementById('info');
  function drawObj(o,s){
    ctx.save(); ctx.globalAlpha=o.opacity??1;
    ctx.translate((o.x+o.w/2)*s,(o.y+o.h/2)*s); ctx.rotate((o.r||0)*Math.PI/180); ctx.translate(-(o.w/2)*s, -(o.h/2)*s);
    ctx.lineWidth=Math.max(1,(o.strokeWidth||1)*s); ctx.strokeStyle=o.stroke||'#e8eaf0'; ctx.fillStyle=o.fill||'#00000000';
    if(o.type==='rect'||o.type==='panel-rect'){ ctx.beginPath(); ctx.rect(0,0,o.w*s,o.h*s); if((o.fill??'')!=='#00000000') ctx.fill(); ctx.stroke(); }
    else if(o.type==='circle' || o.type==='ellipse'){ ctx.beginPath(); ctx.ellipse((o.w*s)/2,(o.h*s)/2,Math.abs(o.w*s/2),Math.abs(o.h*s/2),0,0,Math.PI*2); if((o.fill??'')!=='#00000000') ctx.fill(); ctx.stroke(); }
    else if(o.type==='line' && o.points?.length>=2){ ctx.beginPath(); ctx.moveTo((o.points[0].x-o.x)*s,(o.points[0].y-o.y)*s); ctx.lineTo((o.points[1].x-o.x)*s,(o.points[1].y-o.y)*s); ctx.stroke(); }
    else if((o.type==='path'||o.type==='star') && o.points?.length>=2){ ctx.beginPath(); ctx.moveTo((o.points[0].x-o.x)*s,(o.points[0].y-o.y)*s); for(let k=1;k<o.points.length;k++){ const p=o.points[k]; ctx.lineTo((p.x-o.x)*s,(p.y-o.y)*s);} if(o.type==='star'||o.meta?.closed){ ctx.closePath(); if((o.fill??'')!=='#00000000') ctx.fill(); } ctx.stroke(); }
    else if(o.type==='image'){
      const img=new Image(); img.src=o.meta?.src||'';
      const draw=()=>{ ctx.save(); ctx.translate((o.x+o.w/2)*s,(o.y+o.h/2)*s); ctx.rotate((o.r||0)*Math.PI/180); ctx.translate(-(o.w/2)*s, -(o.h/2)*s); const sx=o.meta?.left||0, sy=o.meta?.top||0; const sw=img.width-(o.meta?.left||0)-(o.meta?.right||0); const sh=img.height-(o.meta?.top||0)-(o.meta?.bottom||0); ctx.drawImage(img,sx,sy,sw,sh,0,0,o.w*s,o.h*s); ctx.restore(); };
      if(img.complete) draw(); else img.onload=draw;
    }
    else if(o.type==='text'){ const fill=o.meta?.fill||o.stroke||'#eaeaf2'; const outline=o.meta?.outline; const outlineColor=o.meta?.outlineColor||'#000'; const outlineWidth=o.meta?.outlineWidth||2; ctx.font=\`\${o.meta?.size||24}px \${o.meta?.family||'sans-serif'}\`; ctx.textBaseline='top'; const lines=(o.meta?.text||'Text').split('\\n'); lines.forEach((line,i)=>{ if(outline){ ctx.lineWidth=Math.max(1,outlineWidth*s); ctx.strokeStyle=outlineColor; ctx.strokeText(line,0,i*(o.meta?.size||24)); } ctx.fillStyle=fill; ctx.fillText(line,0,i*(o.meta?.size||24)); }); }
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
  function refreshUIAfterProjectChange(opts={}){
    const {preserveCamera=false}=opts;
    const cam=preserveCamera? deepClone(state.camera):null;
    refreshTopMeta(); resizeCanvas();
    if(!preserveCamera) fitPage(true); else { state.camera.fit=false; drawPageFrame(); }
    buildPaletteBar(); buildPalBanksUI();
  }

  /* ===== Init ===== */
  function init(){
    applyTheme();
    // start project
    state.project=newProject('phik',1024,576);
    state.palBanks = ensurePalBanks(state.project[PROJ_BANKS_KEY]);
    buildToolButtons(); buildDynamicControls();
    refreshUIAfterProjectChange(); render();
    buildFontSelect($('#textFont'));
    updateMiniSwatches();
    $('#snapToggle').checked = state.snap.enabled;
    $('#snapSize').value = state.snap.size;
    $('#saveKbd').textContent=(state.settings.hotkeys||defaultsHK).save;
    pushHistory();
  }
  addEventListener('resize', resizeCanvas);
  init();
})();
</script>
</body>
</html>
