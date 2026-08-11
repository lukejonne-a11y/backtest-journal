[backtest-journal-v4_4.html](https://github.com/user-attachments/files/30935009/backtest-journal-v4_4.html)
# backtest-journal<!doctype html>
<html lang="de">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Backtest Journal</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@500;600;700;800&family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500;600;700&display=swap');

  :root{
    --bg: #06070B;
    --panel: #0F1319;
    --panel-elev: #151B25;
    --panel-alt: #0B0E14;
    --border: #1D2430;
    --border-soft: #161C27;
    --text: #EDF1F8;
    --muted: #808CA0;
    --muted-dim: #454F62;
    --profit: #2EE6A6;
    --profit-soft: rgba(46,230,166,0.13);
    --loss: #FF5470;
    --loss-soft: rgba(255,84,112,0.13);
    --tp: #FFB84D;
    --tp-soft: rgba(255,184,77,0.15);
    --accent: #6C87FF;
    --accent2: #9B6CFF;
    --accent-soft: rgba(108,135,255,0.15);
    --radius: 16px;
    --radius-sm: 10px;
    --ease: cubic-bezier(.22,1,.36,1);
  }

  *{ box-sizing: border-box; }
  html, body{ margin:0; padding:0; }
  html{ scroll-behavior: smooth; }
  body{
    background:
      radial-gradient(ellipse 1000px 560px at 50% -12%, rgba(108,135,255,0.10), transparent 60%),
      radial-gradient(ellipse 700px 400px at 100% 110%, rgba(155,108,255,0.06), transparent 60%),
      repeating-linear-gradient(0deg, rgba(255,255,255,0.016) 0px, rgba(255,255,255,0.016) 1px, transparent 1px, transparent 34px),
      repeating-linear-gradient(90deg, rgba(255,255,255,0.016) 0px, rgba(255,255,255,0.016) 1px, transparent 1px, transparent 34px),
      var(--bg);
    color: var(--text);
    font-family: 'Inter', sans-serif;
    padding: 28px 18px 90px;
    min-height: 100vh;
    -webkit-font-smoothing: antialiased;
    overflow-x: hidden;
  }

  .wrap{ max-width: 1060px; margin: 0 auto; position: relative; }
  ::selection{ background: var(--accent-soft); color: var(--text); }

  #confettiCanvas{ position: fixed; inset: 0; pointer-events: none; z-index: 9999; }

  /* ---------- Save toast ---------- */
  .save-toast{
    position: fixed; bottom: 22px; right: 22px; background: var(--panel-elev);
    border: 1px solid var(--border); color: var(--profit); font-family:'JetBrains Mono', monospace;
    font-size: 12px; font-weight: 600; padding: 9px 14px; border-radius: 20px;
    display:flex; align-items:center; gap: 7px; box-shadow: 0 10px 30px -8px rgba(0,0,0,0.6);
    opacity: 0; transform: translateY(8px) scale(0.96); transition: opacity .25s var(--ease), transform .25s var(--ease);
    pointer-events: none; z-index: 500;
  }
  .save-toast.show{ opacity: 1; transform: translateY(0) scale(1); }
  .save-toast .sdot{ width: 6px; height: 6px; border-radius: 50%; background: var(--profit); box-shadow: 0 0 8px var(--profit); }
  .save-toast.error{ color: var(--loss); border-color: rgba(255,84,112,0.4); }
  .save-toast.error .sdot{ background: var(--loss); box-shadow: 0 0 8px var(--loss); animation: pulseDot 1s ease-in-out infinite; }
  .save-toast.retry{ color: var(--tp); border-color: rgba(255,184,77,0.4); }
  .save-toast.retry .sdot{ background: var(--tp); box-shadow: 0 0 8px var(--tp); animation: pulseDot 1s ease-in-out infinite; }

  /* ---------- Project tabs ---------- */
  .proj-bar-outer{ position: relative; margin-bottom: 18px; }
  .proj-bar{ display:flex; align-items:center; gap: 6px; overflow-x:auto; scrollbar-width:none; padding-bottom: 2px; }
  .proj-bar::-webkit-scrollbar{ display:none; }
  .proj-tab{
    position: relative; overflow: hidden; display:flex; align-items:center; gap: 7px;
    background: var(--panel); border: 1px solid var(--border); border-radius: 10px 10px 0 0;
    padding: 10px 14px; cursor: pointer; color: var(--muted); font-size: 13px; font-weight: 600;
    white-space: nowrap; border-bottom: 2px solid transparent; flex-shrink: 0;
    transition: color .15s ease, border-color .2s ease, background .15s ease, transform .12s var(--ease);
  }
  .proj-tab:hover{ color: var(--text); transform: translateY(-1px); }
  .proj-tab:active{ transform: translateY(0) scale(0.98); }
  .proj-tab.active{ color: var(--text); background: var(--panel-elev); border-color: var(--border); border-bottom: 2px solid var(--accent); }
  .proj-tab .folder-icon{ opacity: 0.55; font-size: 12px; transition: transform .2s var(--ease), color .2s ease; }
  .proj-tab.active .folder-icon{ opacity: 1; color: var(--accent); transform: rotate(90deg); }
  .proj-tab-actions{ display:none; align-items:center; gap: 4px; margin-left: 2px; }
  .proj-tab.active .proj-tab-actions{ display:flex; }
  .proj-icon-btn{
    background:none; border:none; color: var(--muted-dim); cursor:pointer; font-size: 12px;
    width: 18px; height: 18px; display:flex; align-items:center; justify-content:center; border-radius: 4px;
    transition: all .15s ease;
  }
  .proj-icon-btn:hover{ color: var(--text); background: rgba(255,255,255,0.07); }
  .proj-name-input{
    background: var(--panel-alt); border: 1px solid var(--accent); border-radius: 5px; color: var(--text);
    font-size: 13px; font-weight: 600; padding: 2px 6px; width: 120px; font-family:'Inter',sans-serif; outline:none;
  }
  .proj-add{
    background: none; border: 1px dashed var(--border); border-radius: 10px 10px 0 0; color: var(--muted-dim);
    padding: 10px 12px; cursor: pointer; font-size: 15px; flex-shrink:0; transition: all .15s var(--ease);
  }
  .proj-add:hover{ color: var(--accent); border-color: var(--accent); transform: translateY(-1px); }
  .proj-new-input{
    background: var(--panel-elev); border: 1px solid var(--accent); border-radius: 10px 10px 0 0; color: var(--text);
    padding: 8px 10px; font-size: 13px; width: 140px; outline: none; font-family:'Inter',sans-serif; flex-shrink:0;
  }

  /* ---------- Header ---------- */
  header{
    display:flex; flex-wrap:wrap; align-items:flex-end; justify-content:space-between;
    gap: 22px; margin-bottom: 26px; padding-bottom: 20px; border-bottom: 1px solid var(--border);
  }
  .title-block{ display:flex; align-items:center; gap: 13px; }
  .title-block .mark{
    width: 36px; height: 36px; border-radius: 10px; background: linear-gradient(155deg, var(--accent), var(--accent2));
    display:flex; align-items:center; justify-content:center; flex-shrink:0; position: relative;
    box-shadow: 0 4px 20px rgba(108,135,255,0.4);
  }
  .title-block .mark svg{ width:19px; height:19px; }
  .title-block .mark .live-dot{
    position:absolute; top:-2px; right:-2px; width:8px; height:8px; border-radius:50%;
    background: var(--profit); border: 2px solid var(--bg); animation: pulseDot 2.2s ease-in-out infinite;
  }
  @keyframes pulseDot{ 0%,100%{ opacity:1; } 50%{ opacity:0.35; } }
  .title-row{ display:flex; flex-direction: column; }
  .title-block h1{
    font-family:'Space Grotesk', sans-serif; font-size: 23px; font-weight: 800;
    margin: 0; letter-spacing: -0.015em; line-height: 1.1;
    background: linear-gradient(120deg, var(--text) 30%, var(--accent) 55%, var(--accent2) 70%, var(--text) 90%);
    background-size: 220% 100%;
    -webkit-background-clip: text; background-clip: text; color: transparent;
    animation: titleShimmer 7s ease-in-out infinite;
  }
  @keyframes titleShimmer{ 0%{ background-position: 0% 50%; } 50%{ background-position: 100% 50%; } 100%{ background-position: 0% 50%; } }
  .title-block p{ margin: 4px 0 0; color: var(--muted); font-size: 12.5px; }

  .overall-stats{ display:flex; gap: 26px; flex-wrap: wrap; }
  .stat{ text-align: right; min-width: 64px; }
  .stat .val{ font-family:'JetBrains Mono', monospace; font-size: 21px; font-weight: 700; letter-spacing: -0.01em; font-variant-numeric: tabular-nums; }
  .stat .lbl{ font-size: 10px; text-transform: uppercase; letter-spacing: 0.08em; color: var(--muted); margin-top: 3px; }
  .stat.profit .val{ color: var(--profit); }
  .stat.loss .val{ color: var(--loss); }

  /* ---------- Candle timeframe selector ---------- */
  .axis-wrap{ margin-bottom: 26px; }
  .axis-label{ font-size: 10.5px; text-transform: uppercase; letter-spacing: 0.1em; color: var(--muted-dim); margin-bottom: 10px; padding-left: 2px; }
  .ticks{ display:flex; gap: 6px; overflow-x:auto; scrollbar-width: none; padding: 4px 2px 4px; }
  .ticks::-webkit-scrollbar{ display:none; }
  .tick{
    position: relative; overflow: hidden;
    background: var(--panel); border: 1px solid var(--border); border-radius: var(--radius-sm);
    cursor: pointer; color: var(--muted); padding: 12px 16px 10px; flex: 1; min-width: 86px;
    display:flex; flex-direction:column; align-items:center; gap: 9px;
    transition: border-color .18s ease, background .18s ease, transform .15s var(--ease), box-shadow .18s ease;
  }
  .tick:hover{ border-color: #2A3344; transform: translateY(-2px); }
  .tick:active{ transform: translateY(0) scale(0.98); }
  .tick.active{ background: var(--panel-elev); border-color: var(--tf-color, var(--accent)); box-shadow: 0 0 0 1px var(--tf-color, var(--accent)), 0 8px 22px -4px var(--tf-color, var(--accent)); }

  .candle{ width: 15px; height: 42px; position: relative; will-change: auto; }
  .candle .wick{ position:absolute; left:50%; transform:translateX(-50%); top:2px; bottom:2px; width:1px; background: var(--muted-dim); }
  .candle .body{ position:absolute; left:50%; transform:translateX(-50%); width:11px; border-radius: 2.5px; transition: height .35s var(--ease), top .35s var(--ease), background .2s ease; }

  .tick .tf-label{ font-family:'Space Grotesk', sans-serif; font-size: 13px; font-weight: 600; color: var(--text); display:inline-flex; align-items:center; gap: 5px; }
  .tick .tf-meta{ font-family:'JetBrains Mono', monospace; font-size: 10px; color: var(--muted); white-space: nowrap; }
  .tick-bar{ width: 100%; height: 3px; background: var(--panel-alt); border-radius: 2px; overflow:hidden; margin-top: 1px; }
  .tick-bar-fill{ height: 100%; border-radius: 2px; transition: width .4s var(--ease), background .3s ease; }

  /* ---------- Panel ---------- */
  .panel{
    background: var(--panel); border: 1px solid var(--border); border-radius: var(--radius);
    overflow: hidden; box-shadow: 0 24px 60px -24px rgba(0,0,0,0.6);
    position: relative;
  }
  .panel::before{
    content:''; position:absolute; top:0; left:0; right:0; height:1px;
    background: linear-gradient(90deg, transparent, rgba(255,255,255,0.15), transparent);
  }
  .panel-head{
    display:flex; justify-content:space-between; align-items:center; flex-wrap: wrap; gap: 10px;
    padding: 16px 20px; border-bottom: 1px solid var(--border); background: var(--panel-alt);
  }
  .panel-head-left{ display:flex; flex-direction:column; gap: 6px; }
  .panel-head h2{
    font-family:'Space Grotesk', sans-serif; font-size: 16px; font-weight: 700; margin: 0;
    display:flex; align-items:center; gap: 9px; color: var(--text);
  }
  .panel-head h2 .badge{
    font-family:'JetBrains Mono', monospace; font-size: 10.5px; color: var(--muted);
    background: var(--panel); border: 1px solid var(--border-soft); padding: 3px 8px;
    border-radius: 20px; font-weight: 400;
  }
  .streak-badge{
    font-family:'JetBrains Mono', monospace; font-size: 11px; font-weight: 700; padding: 3px 9px;
    border-radius: 20px; display: inline-flex; align-items:center; gap: 4px; width: fit-content;
    animation: streakIn .3s var(--ease);
  }
  @keyframes streakIn{ from{ opacity:0; transform: scale(0.85); } to{ opacity:1; transform: scale(1); } }
  .streak-badge.hot{ background: var(--profit-soft); color: var(--profit); }
  .streak-badge.cold{ background: var(--loss-soft); color: var(--loss); }

  .mini-stats{ display:flex; gap: 18px; font-family:'JetBrains Mono', monospace; font-size: 12px; align-items: center; }
  .mini-stats span{ color: var(--muted); }
  .mini-stats b{ font-weight: 700; color: var(--text); font-variant-numeric: tabular-nums; }
  .mini-stats .mp b{ color: var(--profit); }
  .mini-stats .ml b{ color: var(--loss); }
  .winbar-wrap{ display:flex; align-items:center; gap: 8px; }
  .winbar{ width: 64px; height: 5px; background: var(--panel); border: 1px solid var(--border-soft); border-radius: 3px; overflow: hidden; }
  .winbar-fill{ height: 100%; border-radius: 3px; transition: width .5s var(--ease), background .3s ease; }

  /* Mode switch (Standard / LSOB) */
  .mode-switch-row{ display:flex; align-items:center; gap: 10px; flex-wrap: wrap; }
  .mode-switch{ display:flex; border: 1px solid var(--border); border-radius: 9px; overflow:hidden; background: var(--panel-alt); }
  .mode-btn{
    position: relative; overflow: hidden; background: none; border: none; color: var(--muted);
    font-family: 'Inter', sans-serif; font-size: 12px; font-weight: 700; padding: 8px 14px; cursor: pointer;
    transition: all .18s var(--ease); white-space: nowrap;
  }
  .mode-btn + .mode-btn{ border-left: 1px solid var(--border); }
  .mode-btn.active{ color: #06181B; background: linear-gradient(155deg, var(--accent2), var(--accent)); }
  .mode-btn:active{ transform: scale(0.96); }

  .crv-input-wrap{
    display:flex; align-items:center; gap: 6px; background: var(--panel-alt); border: 1px solid var(--border);
    border-radius: 9px; padding: 6px 10px;
  }
  .crv-label{ font-family:'JetBrains Mono', monospace; font-size: 11.5px; color: var(--muted); font-weight: 600; }
  .crv-input{
    width: 42px; background: none; border: none; color: var(--tp); font-family:'JetBrains Mono', monospace;
    font-size: 12.5px; font-weight: 700; outline: none; text-align: center;
  }

  .pct-toggle-btn{
    background: var(--panel-alt); border: 1px solid var(--border); color: var(--muted);
    font-family:'Inter', sans-serif; font-size: 12px; font-weight: 600; padding: 8px 12px;
    border-radius: 9px; cursor: pointer; transition: all .18s var(--ease); white-space: nowrap;
  }
  .pct-toggle-btn:hover{ color: var(--text); border-color: #2A3344; }
  .pct-toggle-btn.active{ background: var(--tp-soft); color: var(--tp); border-color: rgba(255,184,77,0.4); }
  .pct-toggle-btn:active{ transform: scale(0.96); }

  /* Timeframe / project accent dots */
  .tf-dot, .proj-dot{ width: 6px; height: 6px; border-radius: 50%; display: inline-block; flex-shrink: 0; }
  .proj-dot{ margin-right: 1px; box-shadow: 0 0 6px currentColor; }

  /* ---------- Trade cards ---------- */
  .cards{ display:flex; flex-direction:column; padding: 10px; gap: 10px; }
  .card{
    position: relative;
    background: var(--panel-elev); border: 1px solid var(--border-soft); border-left: 3px solid var(--border-soft);
    border-radius: var(--radius-sm);
    padding: 13px 14px 13px 13px; display:flex; flex-direction: column; gap: 10px;
    animation: cardIn .3s var(--ease);
    transition: border-color .15s ease, box-shadow .15s ease;
  }
  .card:hover{ border-color: #263047; }
  @keyframes cardIn{ from{ opacity:0; transform: translateY(8px) scale(0.99); } to{ opacity:1; transform: translateY(0) scale(1); } }

  .card-row{ display:flex; flex-wrap: wrap; gap: 8px; align-items: flex-end; }
  .field{ display:flex; flex-direction: column; gap: 4px; flex: 1 1 auto; }
  .field-label{ display:block; font-size: 9.5px; text-transform: uppercase; letter-spacing: 0.06em; color: var(--muted-dim); padding-left: 1px; }

  .field.w-asset{ flex-basis: 118px; min-width: 100px; }
  .field.w-seg2{ flex-basis: 96px; min-width: 88px; }
  .field.w-seg3{ flex-basis: 148px; min-width: 138px; }
  .field.w-num{ flex-basis: 82px; min-width: 72px; }
  .field.w-note{ flex: 2 1 220px; min-width: 180px; }

  .card-del{
    position: absolute; top: 10px; right: 10px;
  }

  input[type=text]{
    background: var(--panel-alt); border: 1px solid var(--border); color: var(--text);
    border-radius: 8px; padding: 8px 10px; font-family: 'Inter', sans-serif; font-size: 13px;
    width: 100%; outline: none; transition: border-color .15s ease, background .15s ease, box-shadow .15s ease;
  }
  input[type=text]:focus{ border-color: var(--accent); background: #0F1420; box-shadow: 0 0 0 3px var(--accent-soft); }
  input[type=text]::placeholder{ color: var(--muted-dim); }
  input.asset{ font-family:'JetBrains Mono', monospace; font-weight: 600; letter-spacing: 0.01em; }
  input.pnl, input.num{ font-family:'JetBrains Mono', monospace; font-weight: 700; text-align: center; padding-left: 4px; padding-right: 4px; }
  input.pnl.pos, input.num.pos{ color: var(--profit); }
  input.pnl.neg, input.num.neg{ color: var(--loss); }

  .r-readout{
    background: var(--panel-alt); border: 1px dashed var(--border); border-radius: 8px;
    padding: 8px 10px; font-family:'JetBrains Mono', monospace; font-weight: 700; font-size: 13px;
    text-align: center; color: var(--muted-dim);
  }
  .r-readout.pos{ color: var(--profit); border-color: rgba(46,230,166,0.35); background: var(--profit-soft); }
  .r-readout.neg{ color: var(--loss); border-color: rgba(255,84,112,0.35); background: var(--loss-soft); }

  /* segmented controls with sliding highlight */
  .seg{ position: relative; display:flex; border: 1px solid var(--border); border-radius: 9px; overflow: hidden; background: var(--panel-alt); }
  .seg-highlight{
    position:absolute; top:0; bottom:0; left:0; border-radius: 7px; margin: 2px;
    transition: transform .28s var(--ease), background .2s ease, width .2s ease;
    z-index: 0;
  }
  .seg-btn{
    position: relative; z-index: 1; flex:1; background: none; border: none; color: var(--muted);
    font-family:'Inter', sans-serif; font-size: 11.5px; font-weight: 600; padding: 8px 5px; cursor: pointer;
    transition: color .18s ease; overflow: hidden; white-space: nowrap;
  }
  .seg-btn:active{ transform: scale(0.96); }
  .seg-btn.active{ color: var(--text); }

  /* TP toggles */
  .tp-toggle{
    position: relative; overflow: hidden;
    width: 40px; height: 34px; border-radius: 9px; border: 1px solid var(--border);
    background: var(--panel-alt); color: var(--muted-dim); font-family:'JetBrains Mono', monospace;
    font-size: 10.5px; font-weight: 700; cursor: pointer; transition: all .18s var(--ease);
  }
  .tp-toggle:hover{ border-color: #3A2F1A; }
  .tp-toggle:active{ transform: scale(0.92); }
  .tp-toggle.active{ background: var(--tp-soft); color: var(--tp); border-color: rgba(255,184,77,0.45); box-shadow: 0 0 0 1px rgba(255,184,77,0.3) inset; }
  .tp-group{ display:flex; gap: 6px; }

  .del-btn{
    position: relative; overflow: hidden;
    background: none; border: 1px solid transparent; color: var(--muted-dim); cursor: pointer;
    font-size: 14px; width: 26px; height: 26px; border-radius: 7px; line-height: 1;
    transition: all .15s ease;
  }
  .del-btn:hover{ color: var(--loss); background: var(--loss-soft); border-color: rgba(255,84,112,0.3); }
  .del-btn:active{ transform: scale(0.88); }

  .add-row{
    display:flex; align-items:center; gap: 12px; padding: 14px 18px;
    border-top: 1px solid var(--border); background: var(--panel-alt);
  }
  .add-btn{
    position: relative; overflow: hidden;
    background: linear-gradient(155deg, var(--accent), #5A3FD6); color: #fff; border: none; border-radius: 10px;
    padding: 10px 20px; font-family:'Inter', sans-serif; font-weight: 700; font-size: 13px;
    cursor: pointer; white-space: nowrap; box-shadow: 0 8px 20px -6px rgba(108,135,255,0.55);
    transition: transform .15s var(--ease), box-shadow .15s var(--ease);
  }
  .add-btn:hover{ transform: translateY(-2px); box-shadow: 0 10px 26px -6px rgba(108,135,255,0.65); }
  .add-btn:active{ transform: translateY(0) scale(0.97); }
  .add-hint{ color: var(--muted); font-size: 12px; }

  .empty{ padding: 46px 18px; text-align:center; color: var(--muted-dim); font-size: 13px; display:flex; flex-direction:column; align-items:center; gap: 10px; }
  .empty .icon{ opacity: 0.35; font-size: 20px; }

  /* ---------- Compare / leaderboard panel ---------- */
  .compare-panel{ margin-top: 18px; }
  .compare-grid{ display:grid; grid-template-columns: repeat(3, 1fr); gap: 14px; padding: 16px; }
  .compare-grid-2{ grid-template-columns: repeat(2, 1fr); }
  @media (max-width: 760px){ .compare-grid{ grid-template-columns: 1fr; } }
  .compare-col-title{
    font-family:'Space Grotesk', sans-serif; font-size: 12.5px; font-weight: 700; color: var(--muted);
    text-transform: uppercase; letter-spacing: 0.05em; margin-bottom: 10px; padding-left: 2px;
  }
  .compare-row{
    display:flex; align-items:center; justify-content:space-between; gap: 8px;
    padding: 9px 11px; border-radius: 9px; background: var(--panel-alt); border: 1px solid var(--border-soft);
    margin-bottom: 7px; transition: border-color .2s ease, box-shadow .2s ease;
  }
  .compare-row.best{ border-color: var(--profit); box-shadow: 0 0 0 1px var(--profit) inset, 0 6px 16px -8px rgba(46,230,166,0.4); }
  .compare-row .crow-label{ font-family:'JetBrains Mono', monospace; font-size: 12px; font-weight: 700; color: var(--text); display:flex; align-items:center; gap: 6px; }
  .compare-row .crow-metrics{ display:flex; gap: 10px; font-family:'JetBrains Mono', monospace; font-size: 11px; color: var(--muted); white-space: nowrap; }
  .compare-row .crow-metrics b{ color: var(--text); }
  .compare-empty{ color: var(--muted-dim); font-size: 12px; padding: 8px 2px; }

  /* ---------- Equity / Auswertung panel ---------- */
  .equity-panel{ margin-top: 18px; }
  .equity-chart-wrap{ padding: 16px 16px 4px; }
  .equity-chart-wrap svg{ width: 100%; height: auto; display: block; }
  .equity-legend{ display:flex; gap: 16px; padding: 0 16px 12px; font-size: 11px; color: var(--muted); font-family:'JetBrains Mono', monospace; }
  .equity-legend span{ display:inline-flex; align-items:center; gap: 5px; }
  .equity-legend .dot{ width: 9px; height: 9px; border-radius: 2px; display:inline-block; }
  .equity-stats{ display:grid; grid-template-columns: repeat(4, 1fr); gap: 10px; padding: 4px 16px 16px; }
  @media (max-width: 700px){ .equity-stats{ grid-template-columns: repeat(2, 1fr); } }
  .equity-stat{
    background: var(--panel-alt); border: 1px solid var(--border-soft); border-radius: 10px;
    padding: 10px 12px;
  }
  .equity-stat .eq-val{ font-family:'JetBrains Mono', monospace; font-size: 16px; font-weight: 700; font-variant-numeric: tabular-nums; }
  .equity-stat .eq-lbl{ font-size: 9.5px; text-transform: uppercase; letter-spacing: 0.06em; color: var(--muted-dim); margin-top: 3px; }
  .equity-stat.profit .eq-val{ color: var(--profit); }
  .equity-stat.loss .eq-val{ color: var(--loss); }
  .equity-empty{ padding: 40px 18px; text-align:center; color: var(--muted-dim); font-size: 13px; }

  .coin-compare-panel{ margin-top: 18px; }
  .coin-compare-row{
    display:flex; align-items:center; justify-content:space-between; gap: 10px;
    padding: 10px 12px; border-radius: 10px; background: var(--panel-alt); border: 1px solid var(--border-soft);
    margin: 0 16px 8px;
  }
  .coin-compare-row:first-of-type{ margin-top: 4px; }
  .coin-compare-row.best{ border-color: var(--profit); box-shadow: 0 0 0 1px var(--profit) inset, 0 6px 16px -8px rgba(46,230,166,0.4); }
  .coin-compare-label{ display:flex; align-items:center; gap: 8px; font-family:'JetBrains Mono', monospace; font-size: 12.5px; font-weight: 700; color: var(--text); }
  .coin-compare-metrics{ display:flex; gap: 10px; font-family:'JetBrains Mono', monospace; font-size: 11px; color: var(--muted); white-space: nowrap; }
  .coin-compare-metrics b{ color: var(--text); }
  .coin-compare-empty{ padding: 30px 18px; text-align:center; color: var(--muted-dim); font-size: 13px; }

  .insight-box{
    margin: 14px 16px 0; padding: 12px 14px; border-radius: 12px;
    background: linear-gradient(120deg, rgba(108,135,255,0.10), rgba(155,108,255,0.08));
    border: 1px solid rgba(108,135,255,0.25);
  }
  .insight-label{
    font-size: 10.5px; text-transform: uppercase; letter-spacing: 0.08em; color: var(--muted);
    margin-bottom: 8px; font-weight: 600;
  }
  .insight-chips{ display:flex; flex-wrap: wrap; gap: 8px; }
  .insight-chip{
    display:inline-flex; align-items:center; gap: 6px; background: var(--panel-alt);
    border: 1px solid var(--border-soft); border-radius: 20px; padding: 5px 12px 5px 10px;
    font-family: 'Inter', sans-serif; font-size: 12.5px; color: var(--text);
  }
  .insight-chip b{ font-family:'JetBrains Mono', monospace; }
  .insight-chip .insight-metric{ color: var(--muted); font-family:'JetBrains Mono', monospace; font-size: 11px; }
  .insight-empty{
    margin: 14px 16px 0; padding: 10px 14px; border-radius: 12px; background: var(--panel-alt);
    border: 1px dashed var(--border-soft); color: var(--muted-dim); font-size: 12px;
  }

  footer{ text-align:center; margin-top: 26px; color: var(--muted-dim); font-size: 11.5px; }

  .backup-row{
    display:flex; align-items:center; justify-content:center; gap: 10px; flex-wrap: wrap;
    margin-top: 22px; padding-top: 18px; border-top: 1px dashed var(--border-soft);
  }
  .backup-btn{
    background: var(--panel); border: 1px solid var(--border); color: var(--text);
    font-family: 'Inter', sans-serif; font-size: 12.5px; font-weight: 600; padding: 8px 14px;
    border-radius: 8px; cursor: pointer; transition: all .15s var(--ease);
  }
  .backup-btn:hover{ border-color: var(--accent); color: var(--accent); transform: translateY(-1px); }
  .backup-btn:active{ transform: translateY(0) scale(0.97); }
  .backup-hint{ color: var(--muted-dim); font-size: 11px; width: 100%; text-align: center; margin-top: 2px; }

  .ripple{
    position:absolute; border-radius:50%; background: rgba(255,255,255,0.35);
    transform: scale(0); opacity:0.55; pointer-events:none; animation: rippleAnim .55s ease-out forwards;
  }
  @keyframes rippleAnim{ to{ transform: scale(2.8); opacity:0; } }

  @media (max-width: 480px){ .overall-stats{ gap: 16px; } header{ gap: 16px; } }

  @media (prefers-reduced-motion: reduce){
    *{ animation-duration: 0.001ms !important; animation-iteration-count: 1 !important; transition-duration: 0.001ms !important; }
  }
</style>
</head>
<body>
<canvas id="confettiCanvas"></canvas>
<div class="save-toast" id="saveToast"><span class="sdot"></span><span id="saveToastText">Gespeichert</span></div>

<div class="wrap">

  <div class="proj-bar-outer">
    <div class="proj-bar" id="projBar"></div>
  </div>

  <header>
    <div class="title-block">
      <div class="mark">
        <svg viewBox="0 0 24 24" fill="none"><path d="M4 18 L9 10 L13 14 L20 5" stroke="white" stroke-width="2.4" stroke-linecap="round" stroke-linejoin="round"/></svg>
        <div class="live-dot"></div>
      </div>
      <div class="title-row">
        <h1 id="projTitle">Backtest Journal</h1>
        <p>Manuelle Trade-Erfassung nach Timeframe</p>
      </div>
    </div>
    <div class="overall-stats" id="overallStats"></div>
  </header>

  <div class="axis-wrap">
    <div class="axis-label">Timeframe</div>
    <div class="ticks" id="ticks"></div>
  </div>

  <div class="panel">
    <div class="panel-head">
      <div class="panel-head-left">
        <h2 id="panelTitle">5 Min <span class="badge" id="panelBadge">0 Trades</span></h2>
        <div id="streakWrap"></div>
      </div>
      <div class="mode-switch-row" id="modeSwitch"></div>
      <div class="mini-stats" id="miniStats"></div>
    </div>
    <div class="cards" id="cardsWrap"></div>
    <div class="add-row">
      <button class="add-btn" id="addBtn">+ Neuer Trade</button>
      <span class="add-hint" id="addHint">Coin frei eintragen · Max-Plus in % oder R eintragen, wie weit's du magst</span>
    </div>
  </div>

  <div class="panel equity-panel" id="equityPanel"></div>

  <div class="panel compare-panel" id="comparePanel">
    <div class="panel-head">
      <h2>📊 Vergleich <span class="badge">aktuelles Projekt, alle Timeframes</span></h2>
    </div>
    <div id="insightBox"></div>
    <div class="compare-grid compare-grid-2">
      <div class="compare-col">
        <div class="compare-col-title">Wick-Verhalten</div>
        <div id="compareWick"></div>
      </div>
      <div class="compare-col">
        <div class="compare-col-title">Timeframe</div>
        <div id="compareTf"></div>
      </div>
    </div>
  </div>

  <div class="panel coin-compare-panel" id="coinComparePanel"></div>

  <div class="backup-row" id="backupRow">
    <button class="backup-btn" id="exportBtn">⬇ Backup herunterladen</button>
    <button class="backup-btn" id="importBtn">⬆ Backup laden</button>
    <input type="file" id="importFile" accept="application/json" style="display:none;">
    <span class="backup-hint">Empfehlung: regelmäßig als Datei sichern, falls der Server-Speicher gerade streikt.</span>
  </div>

  <footer>Daten werden automatisch gespeichert · nur für dich sichtbar</footer>
</div>

<script>
/* ================= Config ================= */
const TIMEFRAMES = [
  { key: '5m',  label: '5 Min'  },
  { key: '15m', label: '15 Min' },
  { key: '30m', label: '30 Min' },
  { key: '1h',  label: '1 Std'  },
  { key: '4h',  label: '4 Std'  },
  { key: '1d',  label: 'Tages'  },
];
const TF_COLORS = { '5m': '#2EE6A6', '15m': '#4FD1FF', '30m': '#6C87FF', '1h': '#9B6CFF', '4h': '#FF6CC9', '1d': '#FFB84D' };
const PROJECT_COLORS = ['#2EE6A6', '#6C87FF', '#FF6CC9', '#FFB84D', '#4FD1FF', '#9B6CFF'];
const STORAGE_KEY_V2 = 'backtest-journal-data-v2';
const STORAGE_KEY_V1 = 'backtest-journal-data-v1';

let store = { projects: {}, order: [], activeProjectId: null };
let activeTF = '5m';
let saveTimer = null;
let addingProject = false;

/* ================= Utilities ================= */
function emptyTimeframes(){ const d = {}; TIMEFRAMES.forEach(tf => d[tf.key] = []); return d; }
function uid(prefix){ return (prefix||'e') + Math.random().toString(36).slice(2, 10); }

function parsePnl(str){
  if(!str) return null;
  const cleaned = String(str).replace(',', '.').match(/-?\d+(\.\d+)?/);
  if(!cleaned) return null;
  let val = parseFloat(cleaned[0]);
  if(/^-/.test(String(str).trim())) val = -Math.abs(val);
  return val;
}

/* Berechnet das Ergebnis eines Trades in R automatisch, je nach Modus:
   - LSOB: fixes CRV (proj.crv, Standard 2) -> Win = +crv, Loss = -1, BE = 0
   - Standard: aus echten Preisen (Einstieg/SL/TP) hergeleitet, je nach Richtung */
function resolveR(entry, proj){
  if(!entry || !entry.result) return null;
  if(entry.result === 'BE') return 0;
  const mode = getMode(proj);
  if(mode === 'lsob'){
    const crv = (proj && typeof proj.crv === 'number' && proj.crv > 0) ? proj.crv : 2;
    return entry.result === 'Win' ? crv : -1;
  }
  const entryPx = parsePnl(entry.entryPx);
  const slPx = parsePnl(entry.slPx);
  const tpPx = parsePnl(entry.tpPx);
  if(entryPx === null || slPx === null) return null;
  const risk = Math.abs(entryPx - slPx);
  if(risk <= 0) return null;
  let exitPx;
  if(entry.result === 'Win'){
    if(tpPx === null) return null;
    exitPx = tpPx;
  } else if(entry.result === 'Loss'){
    exitPx = slPx;
  } else return null;
  const sign = entry.direction === 'Short' ? -1 : 1;
  return sign * (exitPx - entryPx) / risk;
}

function escapeHtml(str){
  if(str === undefined || str === null) return '';
  return String(str).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;');
}

/* ================= Data load / save ================= */
async function loadData(){
  try{
    const res = await window.storage.get(STORAGE_KEY_V2, false);
    if(res && res.value){
      const parsed = JSON.parse(res.value);
      if(parsed && parsed.projects && parsed.order && parsed.order.length){
        store = parsed;
        if(!store.activeProjectId || !store.projects[store.activeProjectId]) store.activeProjectId = store.order[0];
        return;
      }
    }
  }catch(e){ /* not found */ }

  let migratedTimeframes = null;
  try{
    const oldRes = await window.storage.get(STORAGE_KEY_V1, false);
    if(oldRes && oldRes.value){
      const oldData = JSON.parse(oldRes.value);
      if(oldData && typeof oldData === 'object') migratedTimeframes = Object.assign(emptyTimeframes(), oldData);
    }
  }catch(e){ /* not found */ }

  const pid = uid('p');
  store = {
    projects: { [pid]: { name: 'Projekt 1', timeframes: migratedTimeframes || emptyTimeframes() } },
    order: [pid],
    activeProjectId: pid
  };
  scheduleSave();
}

let saveRetryTimer = null;
let saveRetryDelay = 2000;
let saveGeneration = 0; // bumps on every new change, invalidates stale retry loops

function scheduleSave(){
  clearTimeout(saveTimer);
  saveTimer = setTimeout(() => { saveGeneration++; saveData(saveGeneration); }, 350);
}

async function saveData(generation, isRetry){
  const gen = generation !== undefined ? generation : ++saveGeneration;
  const payload = JSON.stringify(store); // snapshot at call time
  try{
    await window.storage.set(STORAGE_KEY_V2, payload, false);
    // Only clear retry state / show success if nothing newer has been queued meanwhile
    if(gen === saveGeneration){
      clearTimeout(saveRetryTimer);
      saveRetryDelay = 2000;
      showSaveToast('ok', 'Gespeichert');
    }
  }catch(e){
    console.error('Speichern fehlgeschlagen', e);
    if(gen === saveGeneration){
      showSaveToast('retry', 'Speichern fehlgeschlagen — erneuter Versuch…');
      clearTimeout(saveRetryTimer);
      saveRetryTimer = setTimeout(() => {
        saveData(saveGeneration, true);
      }, saveRetryDelay);
      saveRetryDelay = Math.min(saveRetryDelay * 1.7, 20000);
    }
  }
}

// Retry immediately once the tab/window regains focus or comes back online
window.addEventListener('online', () => { clearTimeout(saveRetryTimer); saveData(saveGeneration, true); });
document.addEventListener('visibilitychange', () => {
  if(document.visibilityState === 'visible' && saveRetryTimer){
    clearTimeout(saveRetryTimer);
    saveData(saveGeneration, true);
  }
});

let toastTimer = null;
function showSaveToast(state, text){
  const t = document.getElementById('saveToast');
  document.getElementById('saveToastText').textContent = text;
  t.classList.remove('error', 'retry');
  if(state === 'retry') t.classList.add('retry');
  t.classList.add('show');
  clearTimeout(toastTimer);
  if(state === 'ok'){
    toastTimer = setTimeout(() => t.classList.remove('show'), 1400);
  }
  // retry state stays visible until resolved (no auto-hide), so the user knows data isn't lost yet
}

function currentProject(){ return store.projects[store.activeProjectId]; }
function getMode(proj){ return proj && proj.mode === 'standard' ? 'standard' : 'lsob'; }
function setMode(proj, mode){ proj.mode = mode; }

function computeStats(entries, proj){
  let wins = 0, losses = 0, be = 0, pnlSum = 0, pnlCount = 0;
  entries.forEach(e => {
    if(e.result === 'Win') wins++;
    else if(e.result === 'Loss') losses++;
    else if(e.result === 'BE') be++;
    const v = resolveR(e, proj);
    if(v !== null){ pnlSum += v; pnlCount++; }
  });
  const total = entries.length;
  const decided = wins + losses;
  const winrate = decided > 0 ? Math.round((wins / decided) * 100) : null;
  return { total, wins, losses, be, winrate, pnlSum, pnlCount };
}

function computeStreak(entries){
  if(!entries.length) return { type: null, count: 0 };
  let count = 0, type = null;
  for(let i = entries.length - 1; i >= 0; i--){
    const r = entries[i].result;
    if(r !== 'Win' && r !== 'Loss') break;
    if(type === null){ type = r; count = 1; }
    else if(r === type){ count++; }
    else break;
  }
  return { type, count };
}

/* ================= Ripple ================= */
function attachRipple(el){
  el.addEventListener('pointerdown', (e) => {
    const rect = el.getBoundingClientRect();
    const size = Math.max(rect.width, rect.height) * 1.1;
    const span = document.createElement('span');
    span.className = 'ripple';
    span.style.width = size + 'px';
    span.style.height = size + 'px';
    span.style.left = (e.clientX - rect.left - size/2) + 'px';
    span.style.top = (e.clientY - rect.top - size/2) + 'px';
    if(getComputedStyle(el).position === 'static') el.style.position = 'relative';
    el.appendChild(span);
    setTimeout(() => span.remove(), 560);
  });
}

/* ================= Number tween ================= */
const tweenState = new WeakMap();
function tweenNumber(el, target, suffix){
  suffix = suffix || '';
  const prev = tweenState.get(el);
  const start = prev !== undefined ? prev : target;
  if(start === target){ el.textContent = target + suffix; tweenState.set(el, target); return; }
  const duration = 380;
  const t0 = performance.now();
  function frame(now){
    const p = Math.min(1, (now - t0) / duration);
    const eased = 1 - Math.pow(1 - p, 3);
    const val = Math.round(start + (target - start) * eased);
    el.textContent = val + suffix;
    if(p < 1) requestAnimationFrame(frame);
    else tweenState.set(el, target);
  }
  requestAnimationFrame(frame);
}

/* ================= Confetti ================= */
const confettiCanvas = document.getElementById('confettiCanvas');
const cctx = confettiCanvas.getContext('2d');
let confettiParticles = [];
let confettiRunning = false;
let dpr = Math.min(window.devicePixelRatio || 1, 2);

function resizeCanvas(){
  confettiCanvas.width = window.innerWidth * dpr;
  confettiCanvas.height = window.innerHeight * dpr;
  confettiCanvas.style.width = window.innerWidth + 'px';
  confettiCanvas.style.height = window.innerHeight + 'px';
  cctx.setTransform(dpr, 0, 0, dpr, 0, 0);
}
resizeCanvas();
let resizeRaf = null;
window.addEventListener('resize', () => {
  if(resizeRaf) return;
  resizeRaf = requestAnimationFrame(() => { resizeCanvas(); resizeRaf = null; });
});

const CONFETTI_COLORS = ['#2EE6A6', '#6C87FF', '#FFB84D', '#9B6CFF'];
function spawnConfetti(x, y){
  const reduced = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
  if(reduced) return;
  for(let i = 0; i < 16; i++){
    const angle = (Math.PI * 2 * i) / 16 + Math.random() * 0.4;
    const speed = 2.4 + Math.random() * 2.6;
    confettiParticles.push({
      x, y,
      vx: Math.cos(angle) * speed,
      vy: Math.sin(angle) * speed - 2.2,
      size: 3 + Math.random() * 3,
      color: CONFETTI_COLORS[Math.floor(Math.random() * CONFETTI_COLORS.length)],
      rot: Math.random() * Math.PI,
      vrot: (Math.random() - 0.5) * 0.3,
      life: 1
    });
  }
  if(!confettiRunning){ confettiRunning = true; requestAnimationFrame(confettiLoop); }
}
function confettiLoop(){
  cctx.clearRect(0, 0, window.innerWidth, window.innerHeight);
  confettiParticles.forEach(p => {
    p.vy += 0.12;
    p.x += p.vx;
    p.y += p.vy;
    p.rot += p.vrot;
    p.life -= 0.018;
    cctx.save();
    cctx.globalAlpha = Math.max(0, p.life);
    cctx.translate(p.x, p.y);
    cctx.rotate(p.rot);
    cctx.fillStyle = p.color;
    cctx.fillRect(-p.size/2, -p.size/2, p.size, p.size);
    cctx.restore();
  });
  confettiParticles = confettiParticles.filter(p => p.life > 0);
  if(confettiParticles.length > 0){
    requestAnimationFrame(confettiLoop);
  } else {
    confettiRunning = false;
    cctx.clearRect(0, 0, window.innerWidth, window.innerHeight);
  }
}

/* ================= Project bar ================= */
function renderProjBar(){
  const bar = document.getElementById('projBar');
  bar.innerHTML = '';

  store.order.forEach(pid => {
    const proj = store.projects[pid];
    if(!proj) return;
    const tab = document.createElement('div');
    tab.className = 'proj-tab' + (pid === store.activeProjectId ? ' active' : '');

    const dot = document.createElement('span');
    dot.className = 'proj-dot';
    const dotColor = PROJECT_COLORS[store.order.indexOf(pid) % PROJECT_COLORS.length];
    dot.style.background = dotColor;
    dot.style.color = dotColor;

    const folderIcon = document.createElement('span');
    folderIcon.className = 'folder-icon';
    folderIcon.textContent = '▸';
    const nameSpan = document.createElement('span');
    nameSpan.textContent = proj.name;

    tab.appendChild(dot);
    tab.appendChild(folderIcon);
    tab.appendChild(nameSpan);

    tab.addEventListener('click', (e) => {
      if(e.target.closest('.proj-tab-actions')) return;
      if(pid === store.activeProjectId) return;
      store.activeProjectId = pid;
      renderAll();
      scheduleSave();
    });
    attachRipple(tab);

    const actions = document.createElement('div');
    actions.className = 'proj-tab-actions';

    const renameBtn = document.createElement('button');
    renameBtn.className = 'proj-icon-btn';
    renameBtn.title = 'Umbenennen';
    renameBtn.textContent = '✎';
    renameBtn.addEventListener('click', (e) => { e.stopPropagation(); startRename(tab, folderIcon, pid); });

    const delBtn = document.createElement('button');
    delBtn.className = 'proj-icon-btn';
    delBtn.title = 'Projekt löschen';
    delBtn.textContent = '🗑';
    delBtn.addEventListener('click', (e) => {
      e.stopPropagation();
      if(store.order.length <= 1){ alert('Das letzte verbleibende Projekt kann nicht gelöscht werden.'); return; }
      if(confirm(`Projekt „${proj.name}“ inklusive aller Trades wirklich löschen?`)){
        delete store.projects[pid];
        store.order = store.order.filter(id => id !== pid);
        if(store.activeProjectId === pid) store.activeProjectId = store.order[0];
        renderAll();
        scheduleSave();
      }
    });

    actions.appendChild(renameBtn);
    actions.appendChild(delBtn);
    tab.appendChild(actions);
    bar.appendChild(tab);
  });

  const addBtn = document.createElement('button');
  addBtn.className = 'proj-add';
  addBtn.textContent = '+';
  addBtn.title = 'Neues Projekt';
  addBtn.addEventListener('click', () => {
    if(addingProject) return;
    addingProject = true;
    const input = document.createElement('input');
    input.className = 'proj-new-input';
    input.placeholder = 'Projektname…';
    bar.insertBefore(input, addBtn);
    input.focus();
    function finish(){
      const name = input.value.trim();
      addingProject = false;
      if(name){
        const pid = uid('p');
        store.projects[pid] = { name, mode: 'standard', timeframes: emptyTimeframes() };
        store.order.push(pid);
        store.activeProjectId = pid;
        renderAll();
        scheduleSave();
      } else renderProjBar();
    }
    input.addEventListener('keydown', (e) => {
      if(e.key === 'Enter') finish();
      if(e.key === 'Escape'){ addingProject = false; renderProjBar(); }
    });
    input.addEventListener('blur', finish);
  });
  attachRipple(addBtn);
  bar.appendChild(addBtn);
}

function startRename(tab, folderIcon, pid){
  const proj = store.projects[pid];
  const input = document.createElement('input');
  input.className = 'proj-name-input';
  input.value = proj.name;
  tab.innerHTML = '';
  tab.appendChild(folderIcon);
  tab.appendChild(input);
  input.focus();
  input.select();
  function finish(){
    const val = input.value.trim();
    if(val) proj.name = val;
    renderAll();
    scheduleSave();
  }
  input.addEventListener('keydown', (e) => { if(e.key === 'Enter') finish(); if(e.key === 'Escape') renderAll(); });
  input.addEventListener('blur', finish);
  input.addEventListener('click', (e) => e.stopPropagation());
}

/* ================= Overall + ticks ================= */
function renderOverall(){
  const proj = currentProject();
  let all = [];
  TIMEFRAMES.forEach(tf => all = all.concat(proj.timeframes[tf.key] || []));
  const s = computeStats(all, proj);
  document.getElementById('projTitle').textContent = proj.name;

  const wrap = document.getElementById('overallStats');
  if(!wrap.dataset.built){
    wrap.innerHTML = `
      <div class="stat"><div class="val" id="ovTotal">0</div><div class="lbl">Trades</div></div>
      <div class="stat"><div class="val" id="ovWinrate">–</div><div class="lbl">Winrate</div></div>
      <div class="stat profit"><div class="val" id="ovWins">0</div><div class="lbl">Wins</div></div>
      <div class="stat loss"><div class="val" id="ovLosses">0</div><div class="lbl">Losses</div></div>
    `;
    wrap.dataset.built = '1';
  }
  tweenNumber(document.getElementById('ovTotal'), s.total);
  tweenNumber(document.getElementById('ovWins'), s.wins);
  tweenNumber(document.getElementById('ovLosses'), s.losses);
  const wrEl = document.getElementById('ovWinrate');
  if(s.winrate === null) wrEl.textContent = '–';
  else tweenNumber(wrEl, s.winrate, '%');

  renderCompare();
  renderGlobalCoinCompare();
}

/* ================= Compare / leaderboard ================= */

function computeGroupStats(items, keyFn, proj){
  const buckets = {};
  items.forEach(e => {
    const k = keyFn(e);
    if(!k) return;
    if(!buckets[k]) buckets[k] = [];
    buckets[k].push(e);
  });
  const rows = Object.entries(buckets).map(([key, list]) => {
    const s = computeStats(list, proj);
    const avgR = s.pnlCount > 0 ? s.pnlSum / s.pnlCount : null;
    return { key, count: list.length, winrate: s.winrate, avgR };
  });
  rows.sort((a, b) => {
    const av = a.avgR !== null ? a.avgR : -999;
    const bv = b.avgR !== null ? b.avgR : -999;
    return bv - av;
  });
  return rows;
}

function renderCompareList(targetId, rows, labelMap){
  const el = document.getElementById(targetId);
  if(!rows.length){
    el.innerHTML = `<div class="compare-empty">Noch keine bewerteten Trades in dieser Kategorie.</div>`;
    return;
  }
  el.innerHTML = rows.map((r, i) => {
    const label = (labelMap && labelMap[r.key]) || r.key;
    const isBest = i === 0 && r.avgR !== null;
    const parts = [];
    if(r.avgR !== null) parts.push(`Ø <b>${r.avgR >= 0 ? '+' : ''}${r.avgR.toFixed(2)}R</b>`);
    if(r.winrate !== null) parts.push(`<b>${r.winrate}%</b> WR`);
    parts.push(`<b>${r.count}</b>x`);
    return `<div class="compare-row ${isBest ? 'best' : ''}">
      <span class="crow-label">${isBest ? '🏆' : ''} ${escapeHtml(label)}</span>
      <span class="crow-metrics">${parts.join(' · ')}</span>
    </div>`;
  }).join('');
}

function renderGlobalCoinCompare(){
  const panel = document.getElementById('coinComparePanel');
  const rows = store.order.map((pid, idx) => {
    const proj = store.projects[pid];
    if(!proj) return null;
    const rVals = [];
    TIMEFRAMES.forEach(tf => (proj.timeframes[tf.key] || []).forEach(e => {
      const v = resolveR(e, proj);
      if(v !== null) rVals.push(v);
    }));
    const wins = rVals.filter(v => v > 0).length;
    const losses = rVals.filter(v => v < 0).length;
    const winrate = (wins + losses) ? Math.round((wins / (wins + losses)) * 100) : null;
    const avgR = rVals.length ? rVals.reduce((a, b) => a + b, 0) / rVals.length : null;
    const sumR = rVals.length ? rVals.reduce((a, b) => a + b, 0) : null;
    return { pid, name: proj.name, color: PROJECT_COLORS[idx % PROJECT_COLORS.length], count: rVals.length, winrate, avgR, sumR };
  }).filter(Boolean);

  rows.sort((a, b) => {
    const av = a.avgR !== null ? a.avgR : -999;
    const bv = b.avgR !== null ? b.avgR : -999;
    return bv - av;
  });

  const withData = rows.filter(r => r.count > 0);

  panel.innerHTML = `
    <div class="panel-head"><h2>🪙 Coins im Vergleich <span class="badge">alle Projekte</span></h2></div>
    ${withData.length === 0
      ? `<div class="coin-compare-empty">Noch keine bewerteten Trades — sobald du in deinen Projekten R-Ergebnisse einträgst, siehst du hier, welcher Coin am besten läuft.</div>`
      : withData.map((r, i) => {
          const isBest = i === 0 && r.count >= INSIGHT_MIN_COUNT;
          const metrics = [];
          if(r.avgR !== null) metrics.push(`Ø <b>${r.avgR >= 0 ? '+' : ''}${r.avgR.toFixed(2)}R</b>`);
          if(r.sumR !== null) metrics.push(`Σ <b>${r.sumR >= 0 ? '+' : ''}${r.sumR.toFixed(2)}R</b>`);
          if(r.winrate !== null) metrics.push(`<b>${r.winrate}%</b> WR`);
          metrics.push(`<b>${r.count}</b>x`);
          return `<div class="coin-compare-row ${isBest ? 'best' : ''}">
            <span class="coin-compare-label"><span class="proj-dot" style="background:${r.color};color:${r.color};"></span>${isBest ? '🏆 ' : ''}${escapeHtml(r.name)}</span>
            <span class="coin-compare-metrics">${metrics.join(' · ')}</span>
          </div>`;
        }).join('')
    }
  `;
}

function renderCompare(){
  const proj = currentProject();
  const panel = document.getElementById('comparePanel');
  panel.style.display = '';

  const all = [];
  TIMEFRAMES.forEach(tf => (proj.timeframes[tf.key] || []).forEach(e => all.push(Object.assign({ _tfLabel: tf.label }, e))));

  const dirRows = computeGroupStats(all, e => e.direction, proj);
  const tfRows = computeGroupStats(all, e => e._tfLabel, proj);

  panel.innerHTML = `
    <div class="panel-head">
      <h2>📊 Vergleich <span class="badge">${escapeHtml(proj.name)}, alle Timeframes</span></h2>
    </div>
    <div id="insightBox"></div>
    <div class="compare-grid compare-grid-2">
      <div class="compare-col">
        <div class="compare-col-title">Richtung</div>
        <div id="compareDir"></div>
      </div>
      <div class="compare-col">
        <div class="compare-col-title">Timeframe</div>
        <div id="compareTf"></div>
      </div>
    </div>
  `;

  renderCompareList('compareDir', dirRows, null);
  renderCompareList('compareTf', tfRows, null);
  renderInsight(dirRows, tfRows);
}

/* Minimum sample size before a category is confident enough to headline the insight sentence */
const INSIGHT_MIN_COUNT = 2;

function bestQualified(rows){
  if(!rows.length) return null;
  const top = rows[0];
  if(top.count < INSIGHT_MIN_COUNT) return null;
  if(top.avgR === null) return null;
  return top;
}

function fmtMetric(row){
  const parts = [];
  if(row.avgR !== null) parts.push(`Ø ${row.avgR >= 0 ? '+' : ''}${row.avgR.toFixed(2)}R`);
  if(row.winrate !== null) parts.push(`${row.winrate}% WR`);
  parts.push(`${row.count}x`);
  return parts.join(' · ');
}

function renderInsight(dirRows, tfRows){
  const box = document.getElementById('insightBox');
  const bestDir = bestQualified(dirRows);
  const bestTf = bestQualified(tfRows);

  if(!bestDir && !bestTf){
    box.innerHTML = `<div class="insight-empty">💡 Noch zu wenig Daten für eine Einschätzung — trag ein paar Trades mehr ein (mind. ${INSIGHT_MIN_COUNT} pro Kategorie).</div>`;
    return;
  }

  const chips = [];
  if(bestTf) chips.push(`<span class="insight-chip">🕒 <b>${escapeHtml(bestTf.key)}</b> <span class="insight-metric">${fmtMetric(bestTf)}</span></span>`);
  if(bestDir) chips.push(`<span class="insight-chip">↕️ <b>${escapeHtml(bestDir.key)}</b> <span class="insight-metric">${fmtMetric(bestDir)}</span></span>`);

  box.innerHTML = `
    <div class="insight-box">
      <div class="insight-label">Bisher am besten gelaufen</div>
      <div class="insight-chips">${chips.join('')}</div>
    </div>
  `;
}

function candleStyle(entries, maxCount, proj){
  const s = computeStats(entries, proj);
  let color = 'var(--muted-dim)';
  if(s.winrate !== null) color = s.winrate >= 50 ? 'var(--profit)' : 'var(--loss)';
  const containerH = 38, minH = 8;
  const heightPct = s.total > 0 ? Math.max(0.22, s.total / maxCount) : 0.14;
  const h = Math.round(minH + heightPct * (containerH - minH));
  const top = Math.round((containerH - h) / 2 + 2);
  return { color, h, top, winrate: s.winrate };
}

function renderTicks(){
  const proj = currentProject();
  const el = document.getElementById('ticks');
  el.innerHTML = '';
  const maxCount = Math.max(1, ...TIMEFRAMES.map(tf => (proj.timeframes[tf.key] || []).length));
  TIMEFRAMES.forEach(tf => {
    const entries = proj.timeframes[tf.key] || [];
    const s = computeStats(entries, proj);
    const cs = candleStyle(entries, maxCount, proj);
    const btn = document.createElement('button');
    btn.className = 'tick' + (tf.key === activeTF ? ' active' : '');
    btn.style.setProperty('--tf-color', TF_COLORS[tf.key]);
    const barColor = s.winrate === null ? 'var(--muted-dim)' : (s.winrate >= 50 ? 'var(--profit)' : 'var(--loss)');
    btn.innerHTML = `
      <div class="candle">
        <div class="wick"></div>
        <div class="body" style="height:${cs.h}px; top:${cs.top}px; background:${cs.color};"></div>
      </div>
      <span class="tf-label"><span class="tf-dot" style="background:${TF_COLORS[tf.key]};"></span>${tf.label}</span>
      <span class="tf-meta">${s.total} · ${s.winrate !== null ? s.winrate + '%' : '–'}</span>
      <div class="tick-bar"><div class="tick-bar-fill" style="width:${s.winrate !== null ? s.winrate : 0}%; background:${barColor};"></div></div>
    `;
    btn.addEventListener('click', () => { if(activeTF === tf.key) return; activeTF = tf.key; renderPanel(); renderTicks(); });
    attachRipple(btn);
    el.appendChild(btn);
  });
}

/* ================= Mini stats + streak ================= */
function updateMiniStats(){
  const proj = currentProject();
  const entries = proj.timeframes[activeTF] || [];
  const s = computeStats(entries, proj);
  const winrateColor = s.winrate === null ? 'var(--muted-dim)' : (s.winrate >= 50 ? 'var(--profit)' : 'var(--loss)');
  document.getElementById('miniStats').innerHTML = `
    <span class="mp"><b>${s.wins}</b> Win</span>
    <span class="ml"><b>${s.losses}</b> Loss</span>
    <span><b>${s.be}</b> BE</span>
    <div class="winbar-wrap"><span>Winrate</span><div class="winbar"><div class="winbar-fill" style="width:${s.winrate !== null ? s.winrate : 0}%; background:${winrateColor};"></div></div><b>${s.winrate !== null ? s.winrate + '%' : '–'}</b></div>
    ${s.pnlCount > 0 ? `<span>Σ R <b style="color:${s.pnlSum >= 0 ? 'var(--profit)' : 'var(--loss)'}">${s.pnlSum >= 0 ? '+' : ''}${s.pnlSum.toFixed(2)}R</b></span>` : ''}
  `;
  document.getElementById('panelBadge').textContent = `${s.total} Trade${s.total === 1 ? '' : 's'}`;

  const streak = computeStreak(entries);
  const streakWrap = document.getElementById('streakWrap');
  if(streak.type && streak.count >= 2){
    const hot = streak.type === 'Win';
    streakWrap.innerHTML = `<span class="streak-badge ${hot ? 'hot' : 'cold'}">${hot ? '🔥' : '❄️'} ${streak.count}er ${hot ? 'Win' : 'Loss'}-Serie</span>`;
  } else {
    streakWrap.innerHTML = '';
  }
}

function pnlClass(str){ const v = parsePnl(str); if(v === null) return ''; return v >= 0 ? 'pos' : 'neg'; }

/* ================= Equity curve / Auswertung ================= */
function buildEquitySVG(rValues){
  const cum = [];
  let running = 0;
  rValues.forEach(v => { running += v; cum.push(running); });
  const hwm = [];
  let peak = 0;
  cum.forEach(v => { peak = Math.max(peak, v); hwm.push(peak); });

  const series = [0, ...cum];
  const hwmSeries = [0, ...hwm];
  const n = series.length;

  const W = 640, H = 190, padL = 4, padR = 4, padT = 12, padB = 12;
  const innerW = W - padL - padR, innerH = H - padT - padB;
  const allVals = series.concat(hwmSeries, [0]);
  const minV = Math.min(...allVals);
  const maxV = Math.max(...allVals);
  const range = (maxV - minV) || 1;

  const x = i => padL + (n > 1 ? (i / (n - 1)) * innerW : innerW / 2);
  const y = v => padT + innerH - ((v - minV) / range) * innerH;
  const zeroY = y(0);

  const eqPts = series.map((v, i) => `${x(i).toFixed(1)},${y(v).toFixed(1)}`);
  const hwmPts = hwmSeries.map((v, i) => `${x(i).toFixed(1)},${y(v).toFixed(1)}`);

  const areaPoly = `${x(0).toFixed(1)},${zeroY.toFixed(1)} ${eqPts.join(' ')} ${x(n - 1).toFixed(1)},${zeroY.toFixed(1)}`;
  const ribbonPoly = `${hwmPts.join(' ')} ${eqPts.slice().reverse().join(' ')}`;

  return `
    <svg viewBox="0 0 ${W} ${H}" preserveAspectRatio="none">
      <line x1="${padL}" y1="${zeroY.toFixed(1)}" x2="${W - padR}" y2="${zeroY.toFixed(1)}" style="stroke:var(--border);stroke-width:1;stroke-dasharray:2,3;" />
      <polygon points="${areaPoly}" style="fill:var(--accent);fill-opacity:0.14;" />
      <polygon points="${ribbonPoly}" style="fill:var(--loss);fill-opacity:0.16;" />
      <polyline points="${hwmPts.join(' ')}" style="fill:none;stroke:var(--muted);stroke-width:1.3;stroke-dasharray:3,3;" />
      <polyline points="${eqPts.join(' ')}" style="fill:none;stroke:var(--accent);stroke-width:2.2;stroke-linejoin:round;stroke-linecap:round;" />
    </svg>
  `;
}

function computeEquityStats(rValues){
  const n = rValues.length;
  const wins = rValues.filter(v => v > 0);
  const losses = rValues.filter(v => v < 0);
  const cumFinal = rValues.reduce((a, b) => a + b, 0);

  let running = 0, peak = 0, maxDD = 0, peakVal = 0;
  rValues.forEach(v => {
    running += v;
    peak = Math.max(peak, running);
    peakVal = Math.max(peakVal, running);
    maxDD = Math.min(maxDD, running - peak);
  });

  const avgWin = wins.length ? wins.reduce((a, b) => a + b, 0) / wins.length : null;
  const avgLoss = losses.length ? losses.reduce((a, b) => a + b, 0) / losses.length : null;
  const profitFactor = losses.length
    ? (wins.reduce((a, b) => a + b, 0) / Math.abs(losses.reduce((a, b) => a + b, 0)))
    : (wins.length ? Infinity : null);
  const rr = (avgWin !== null && avgLoss !== null && avgLoss !== 0) ? (avgWin / Math.abs(avgLoss)) : null;

  let maxWinStreak = 0, maxLossStreak = 0, curWin = 0, curLoss = 0;
  rValues.forEach(v => {
    if(v > 0){ curWin++; curLoss = 0; maxWinStreak = Math.max(maxWinStreak, curWin); }
    else if(v < 0){ curLoss++; curWin = 0; maxLossStreak = Math.max(maxLossStreak, curLoss); }
    else { curWin = 0; curLoss = 0; }
  });

  return {
    n, winCount: wins.length, lossCount: losses.length,
    winrate: (wins.length + losses.length) ? Math.round((wins.length / (wins.length + losses.length)) * 100) : null,
    cumFinal, peakVal, maxDD, avgWin, avgLoss, profitFactor, rr, maxWinStreak, maxLossStreak
  };
}

function renderEquity(){
  const proj = currentProject();
  const entries = proj.timeframes[activeTF] || [];
  const rValues = entries.map(e => resolveR(e, proj)).filter(v => v !== null);
  const panel = document.getElementById('equityPanel');
  const tf = TIMEFRAMES.find(t => t.key === activeTF);

  if(rValues.length < 2){
    panel.innerHTML = `
      <div class="panel-head"><h2>📈 Auswertung <span class="badge">${tf.label}</span></h2></div>
      <div class="equity-empty">Trag bei mind. 2 Trades ein Ergebnis in R ein, dann erscheint hier deine Equity-Kurve.</div>
    `;
    return;
  }

  const stats = computeEquityStats(rValues);
  const fmt = v => v === null ? '–' : (v >= 0 ? '+' : '') + v.toFixed(2) + 'R';

  const mode = getMode(proj);
  const crv = (proj.crv && proj.crv > 0) ? proj.crv : 2;
  let mfeBlock = '';
  if(mode === 'lsob'){
    const mfeVals = entries.map(e => parsePnl(e.maxR)).filter(v => v !== null);
    const maeVals = entries.map(e => parsePnl(e.maxMinusR)).filter(v => v !== null);
    const pastVals = entries.map(e => {
      const m = parsePnl(e.maxR);
      return m !== null ? Math.max(0, m - crv) : null;
    }).filter(v => v !== null);
    const avgMFE = mfeVals.length ? mfeVals.reduce((a,b)=>a+b,0)/mfeVals.length : null;
    const avgMAE = maeVals.length ? maeVals.reduce((a,b)=>a+b,0)/maeVals.length : null;
    const avgPast = pastVals.length ? pastVals.reduce((a,b)=>a+b,0)/pastVals.length : null;
    if(avgMFE !== null || avgMAE !== null){
      mfeBlock = `
        <div class="equity-stats" style="padding-top:0;">
          <div class="equity-stat profit"><div class="eq-val">${avgMFE === null ? '–' : '+' + avgMFE.toFixed(2) + 'R'}</div><div class="eq-lbl">Ø MFE (max. erreichtes CRV)</div></div>
          <div class="equity-stat loss"><div class="eq-val">${avgMAE === null ? '–' : avgMAE.toFixed(2) + 'R'}</div><div class="eq-lbl">Ø MAE</div></div>
          <div class="equity-stat"><div class="eq-val">${avgPast === null ? '–' : '+' + avgPast.toFixed(2) + 'R'}</div><div class="eq-lbl">Ø Lauf über ${crv}R hinaus</div></div>
          <div class="equity-stat"><div class="eq-val">1 : ${crv}</div><div class="eq-lbl">Fixes CRV dieses Projekts</div></div>
        </div>
      `;
    }
  }

  panel.innerHTML = `
    <div class="panel-head"><h2>📈 Auswertung <span class="badge">${tf.label} · ${stats.n} bewertete Trades</span></h2></div>
    <div class="equity-chart-wrap">${buildEquitySVG(rValues)}</div>
    <div class="equity-legend">
      <span><span class="dot" style="background:var(--accent);"></span>Kumuliertes R</span>
      <span><span class="dot" style="background:var(--muted); opacity:0.7;"></span>High-Water-Mark</span>
      <span><span class="dot" style="background:var(--loss); opacity:0.6;"></span>Drawdown-Zone</span>
    </div>
    <div class="equity-stats">
      <div class="equity-stat ${stats.cumFinal >= 0 ? 'profit' : 'loss'}"><div class="eq-val">${fmt(stats.cumFinal)}</div><div class="eq-lbl">Kumuliert</div></div>
      <div class="equity-stat profit"><div class="eq-val">${fmt(stats.peakVal)}</div><div class="eq-lbl">Höchster Stand</div></div>
      <div class="equity-stat loss"><div class="eq-val">${fmt(stats.maxDD)}</div><div class="eq-lbl">Max. Drawdown</div></div>
      <div class="equity-stat"><div class="eq-val">${stats.winrate !== null ? stats.winrate + '%' : '–'}</div><div class="eq-lbl">Winrate</div></div>
      <div class="equity-stat"><div class="eq-val">${stats.profitFactor === null ? '–' : (stats.profitFactor === Infinity ? '∞' : stats.profitFactor.toFixed(2))}</div><div class="eq-lbl">Profit Factor</div></div>
      <div class="equity-stat profit"><div class="eq-val">${fmt(stats.avgWin)}</div><div class="eq-lbl">Ø Win</div></div>
      <div class="equity-stat loss"><div class="eq-val">${fmt(stats.avgLoss)}</div><div class="eq-lbl">Ø Loss</div></div>
      <div class="equity-stat"><div class="eq-val">${stats.rr === null ? '–' : '1 : ' + stats.rr.toFixed(2)}</div><div class="eq-lbl">Ø Risk:Reward</div></div>
      <div class="equity-stat"><div class="eq-val">${stats.n}</div><div class="eq-lbl">Bewertete Trades</div></div>
      <div class="equity-stat profit"><div class="eq-val">${stats.maxWinStreak}</div><div class="eq-lbl">Längste Win-Serie</div></div>
      <div class="equity-stat loss"><div class="eq-val">${stats.maxLossStreak}</div><div class="eq-lbl">Längste Loss-Serie</div></div>
      <div class="equity-stat"><div class="eq-val">${stats.winCount}/${stats.lossCount}</div><div class="eq-lbl">Wins / Losses</div></div>
    </div>
    ${mfeBlock}
  `;
}

function renderModeSwitch(proj){
  const mode = getMode(proj);
  const wrap = document.getElementById('modeSwitch');
  wrap.innerHTML = '';

  const segWrap = document.createElement('div');
  segWrap.className = 'mode-switch';
  [['standard','Standard'], ['lsob','LSOB']].forEach(([val, label]) => {
    const btn = document.createElement('button');
    btn.className = 'mode-btn' + (mode === val ? ' active' : '');
    btn.textContent = label;
    attachRipple(btn);
    btn.addEventListener('click', () => {
      if(getMode(proj) === val) return;
      setMode(proj, val);
      renderAll();
      scheduleSave();
    });
    segWrap.appendChild(btn);
  });
  wrap.appendChild(segWrap);

  if(mode === 'lsob'){
    const crvWrap = document.createElement('div');
    crvWrap.className = 'crv-input-wrap';
    crvWrap.title = 'Fixes CRV für diesen Backtest (Win = +CRV, Loss = -1)';
    const crvLabel = document.createElement('span');
    crvLabel.className = 'crv-label';
    crvLabel.textContent = 'CRV 1:';
    const crvInput = document.createElement('input');
    crvInput.type = 'text';
    crvInput.className = 'crv-input';
    crvInput.value = (proj.crv && proj.crv > 0) ? proj.crv : 2;
    crvInput.addEventListener('input', () => {
      const v = parseFloat(String(crvInput.value).replace(',', '.'));
      proj.crv = (isFinite(v) && v > 0) ? v : 2;
      renderTicks(); renderOverall(); updateMiniStats(); renderEquity(); scheduleSave();
    });
    crvWrap.appendChild(crvLabel);
    crvWrap.appendChild(crvInput);
    wrap.appendChild(crvWrap);
  }

  const pctBtn = document.createElement('button');
  pctBtn.className = 'pct-toggle-btn' + (store.showPctMovement ? ' active' : '');
  pctBtn.textContent = '% Bewegung';
  pctBtn.title = 'Zusätzliche, optionale %-Kursbewegung pro Trade ein-/ausblenden';
  attachRipple(pctBtn);
  pctBtn.addEventListener('click', () => {
    store.showPctMovement = !store.showPctMovement;
    renderAll();
    scheduleSave();
  });
  wrap.appendChild(pctBtn);
}

/* ================= Segmented control builder ================= */
function buildSeg(field, options, currentValue, onChange){
  // options: [{value,label,activeClass,color}]
  const seg = document.createElement('div');
  seg.className = 'seg';
  seg.setAttribute('data-field', field);

  const highlight = document.createElement('div');
  highlight.className = 'seg-highlight';
  seg.appendChild(highlight);

  const btns = options.map(opt => {
    const btn = document.createElement('button');
    btn.className = 'seg-btn' + (opt.value === currentValue ? ' active' : '');
    btn.textContent = opt.label;
    btn.setAttribute('data-value', opt.value);
    if(opt.title) btn.title = opt.title;
    attachRipple(btn);
    btn.addEventListener('click', () => {
      positionHighlight(opt);
      btns.forEach(b => b.classList.remove('active'));
      btn.classList.add('active');
      onChange(opt.value);
    });
    seg.appendChild(btn);
    return btn;
  });

  function positionHighlight(opt){
    const n = options.length;
    const idx = options.findIndex(o => o.value === opt.value);
    highlight.style.width = `calc(${100/n}% - 4px)`;
    highlight.style.transform = `translateX(${idx * 100}%)`;
    highlight.style.background = opt.color;
  }
  const initialOpt = options.find(o => o.value === currentValue) || options[0];
  requestAnimationFrame(() => positionHighlight(initialOpt));

  return seg;
}

/* ================= Panel / cards ================= */
function renderPanel(){
  const proj = currentProject();
  const tf = TIMEFRAMES.find(t => t.key === activeTF);
  const entries = proj.timeframes[activeTF] || [];
  const mode = getMode(proj);

  document.getElementById('panelTitle').innerHTML =
    `${tf.label} <span class="badge" id="panelBadge">${entries.length} Trade${entries.length === 1 ? '' : 's'}</span>`;

  renderModeSwitch(proj);
  renderEquity();
  updateMiniStats();

  const hint = document.getElementById('addHint');
  hint.textContent = mode === 'lsob'
    ? `Ergebnis (R) wird automatisch aus dem fixen CRV berechnet · trag zusätzlich MFE/MAE ein`
    : 'Einstieg, SL, TP eintragen · Ergebnis (R) wird automatisch daraus berechnet';

  const wrap = document.getElementById('cardsWrap');
  wrap.innerHTML = '';

  if(entries.length === 0){
    wrap.innerHTML = `<div class="empty"><span class="icon">◇</span>Noch keine Trades für ${tf.label} — leg unten den ersten an.</div>`;
    return;
  }

  entries.forEach(entry => {
    const card = document.createElement('div');
    card.className = 'card';
    card.style.borderLeftColor = TF_COLORS[activeTF];

    const rowTop = document.createElement('div');
    rowTop.className = 'card-row';

    const rowBottom = document.createElement('div');
    rowBottom.className = 'card-row';

    function field(labelText, widthClass){
      const w = document.createElement('div');
      w.className = 'field ' + widthClass;
      const lbl = document.createElement('div');
      lbl.className = 'field-label';
      lbl.textContent = labelText;
      w.appendChild(lbl);
      return w;
    }

    // --- Computed-R readout (live, auto-berechnet je nach Modus) ---
    const rBadgeWrap = field('→ Ergebnis (R)', 'w-num');
    const rBadge = document.createElement('div');
    rBadge.className = 'r-readout';
    rBadgeWrap.appendChild(rBadge);
    function refreshRBadge(){
      const v = resolveR(entry, proj);
      if(v === null){ rBadge.textContent = '–'; rBadge.className = 'r-readout'; }
      else{ rBadge.textContent = (v >= 0 ? '+' : '') + v.toFixed(2) + 'R'; rBadge.className = 'r-readout ' + (v >= 0 ? 'pos' : 'neg'); }
    }

    // --- Row 1: Klassifizierung ---
    const dirWrap = field('Richtung', 'w-seg2');
    const dirSeg = buildSeg('direction', [
      { value: 'Long', label: 'Long', color: 'var(--accent-soft)' },
      { value: 'Short', label: 'Short', color: 'var(--accent-soft)' }
    ], entry.direction, (val) => { entry.direction = val; refreshRBadge(); renderTicks(); renderOverall(); updateMiniStats(); renderEquity(); scheduleSave(); });
    dirWrap.appendChild(dirSeg);

    const resWrap = field('Ergebnis', 'w-seg3');
    const resSeg = buildSeg('result', [
      { value: 'Win', label: 'Win', color: 'var(--profit-soft)' },
      { value: 'Loss', label: 'Loss', color: 'var(--loss-soft)' },
      { value: 'BE', label: 'BE', color: 'rgba(124,135,156,0.18)' }
    ], entry.result, (val) => {
      entry.result = val;
      refreshRBadge(); renderTicks(); renderOverall(); updateMiniStats(); renderEquity(); scheduleSave();
      if(val === 'Win'){
        const rect = resSeg.getBoundingClientRect();
        spawnConfetti(rect.left + rect.width/2, rect.top + rect.height/2);
      }
    });
    resWrap.appendChild(resSeg);

    rowTop.appendChild(dirWrap);
    rowTop.appendChild(resWrap);
    rowTop.appendChild(rBadgeWrap);

    // --- Row 2: Performance (unterscheidet sich je nach Modus) ---
    if(mode === 'standard'){
      const mk = (labelText, field_, placeholder) => {
        const w = field(labelText, 'w-num');
        const input = document.createElement('input');
        input.type = 'text'; input.className = 'num'; input.placeholder = placeholder; input.value = entry[field_] || '';
        input.addEventListener('input', () => {
          entry[field_] = input.value;
          refreshRBadge(); renderTicks(); renderOverall(); updateMiniStats(); renderEquity(); scheduleSave();
        });
        w.appendChild(input);
        return w;
      };
      rowBottom.appendChild(mk('Einstieg', 'entryPx', 'z. B. 61234.5'));
      rowBottom.appendChild(mk('Stop-Loss', 'slPx', 'z. B. 60800'));
      rowBottom.appendChild(mk('Take-Profit', 'tpPx', 'z. B. 62100'));
    } else {
      const mfeWrap = field('MFE (R)', 'w-num');
      const mfeInput = document.createElement('input');
      mfeInput.type = 'text'; mfeInput.className = 'num ' + pnlClass(entry.maxR); mfeInput.placeholder = '+2.8R'; mfeInput.value = entry.maxR || '';
      mfeInput.addEventListener('input', () => {
        entry.maxR = mfeInput.value;
        mfeInput.className = 'num ' + pnlClass(entry.maxR);
        renderCompare(); renderEquity(); scheduleSave();
      });
      mfeWrap.appendChild(mfeInput);

      const maeWrap = field('MAE (R)', 'w-num');
      const maeInput = document.createElement('input');
      maeInput.type = 'text'; maeInput.className = 'num ' + pnlClass(entry.maxMinusR); maeInput.placeholder = '-0.6R'; maeInput.value = entry.maxMinusR || '';
      maeInput.addEventListener('input', () => {
        entry.maxMinusR = maeInput.value;
        maeInput.className = 'num ' + pnlClass(entry.maxMinusR);
        renderCompare(); renderEquity(); scheduleSave();
      });
      maeWrap.appendChild(maeInput);

      rowBottom.appendChild(mfeWrap);
      rowBottom.appendChild(maeWrap);
    }

    // --- Optionaler %-Bewegungs-Block (unabhängig vom Modus, standardmäßig aus) ---
    if(store.showPctMovement){
      const pctPlusWrap = field('Max. Plus %', 'w-num');
      const pctPlusInput = document.createElement('input');
      pctPlusInput.type = 'text'; pctPlusInput.className = 'num ' + pnlClass(entry.maxPct); pctPlusInput.placeholder = '+4.8%'; pctPlusInput.value = entry.maxPct || '';
      pctPlusInput.addEventListener('input', () => {
        entry.maxPct = pctPlusInput.value;
        pctPlusInput.className = 'num ' + pnlClass(entry.maxPct);
        scheduleSave();
      });
      pctPlusWrap.appendChild(pctPlusInput);

      const pctMinusWrap = field('Max. Minus %', 'w-num');
      const pctMinusInput = document.createElement('input');
      pctMinusInput.type = 'text'; pctMinusInput.className = 'num ' + pnlClass(entry.minPct); pctMinusInput.placeholder = '-1.2%'; pctMinusInput.value = entry.minPct || '';
      pctMinusInput.addEventListener('input', () => {
        entry.minPct = pctMinusInput.value;
        pctMinusInput.className = 'num ' + pnlClass(entry.minPct);
        scheduleSave();
      });
      pctMinusWrap.appendChild(pctMinusInput);

      rowBottom.appendChild(pctPlusWrap);
      rowBottom.appendChild(pctMinusWrap);
    }

    const noteWrap = field('Notiz', 'w-note');
    const noteInput = document.createElement('input');
    noteInput.type = 'text'; noteInput.placeholder = 'Struktur, Session, Beobachtung…'; noteInput.value = entry.note || '';
    noteInput.addEventListener('input', () => { entry.note = noteInput.value; scheduleSave(); });
    noteWrap.appendChild(noteInput);
    rowBottom.appendChild(noteWrap);

    // delete (top-right corner)
    const delBtn = document.createElement('button');
    delBtn.className = 'del-btn card-del'; delBtn.title = 'Trade löschen'; delBtn.textContent = '✕';
    attachRipple(delBtn);
    delBtn.addEventListener('click', () => {
      proj.timeframes[activeTF] = proj.timeframes[activeTF].filter(e => e.id !== entry.id);
      renderAll();
      scheduleSave();
    });

    card.appendChild(delBtn);
    card.appendChild(rowTop);
    card.appendChild(rowBottom);
    refreshRBadge();

    wrap.appendChild(card);
  });
}

function addEntry(){
  const proj = currentProject();
  const entry = {
    id: uid('t'), direction: 'Long', result: 'Win',
    entryPx: '', slPx: '', tpPx: '',
    maxR: '', maxMinusR: '',
    maxPct: '', minPct: '',
    note: ''
  };
  if(!proj.timeframes[activeTF]) proj.timeframes[activeTF] = [];
  proj.timeframes[activeTF].push(entry);
  renderAll();
  scheduleSave();
  setTimeout(() => {
    const inputs = document.querySelectorAll('.num');
    if(inputs.length) inputs[inputs.length - 1].focus();
  }, 30);
}

function renderAll(){
  renderProjBar();
  renderTicks();
  renderPanel();
  renderOverall();
}

const addBtnEl = document.getElementById('addBtn');
attachRipple(addBtnEl);
addBtnEl.addEventListener('click', addEntry);

/* ================= Backup: Export / Import ================= */
document.getElementById('exportBtn').addEventListener('click', () => {
  try{
    const payload = JSON.stringify(store, null, 2);
    const blob = new Blob([payload], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    const stamp = new Date().toISOString().slice(0,19).replace(/[:T]/g,'-');
    a.href = url;
    a.download = `backtest-journal-backup-${stamp}.json`;
    document.body.appendChild(a);
    a.click();
    a.remove();
    setTimeout(() => URL.revokeObjectURL(url), 2000);
  }catch(e){
    alert('Backup konnte nicht erstellt werden: ' + e.message);
  }
});

document.getElementById('importBtn').addEventListener('click', () => {
  document.getElementById('importFile').click();
});

document.getElementById('importFile').addEventListener('change', (e) => {
  const file = e.target.files[0];
  if(!file) return;
  const reader = new FileReader();
  reader.onload = () => {
    try{
      const parsed = JSON.parse(reader.result);
      if(!parsed || !parsed.projects || !parsed.order || !Array.isArray(parsed.order) || !parsed.order.length){
        alert('Diese Datei sieht nicht wie ein gültiges Backup aus.');
        return;
      }
      if(!confirm('Backup laden? Der aktuell angezeigte Stand wird dadurch komplett ersetzt.')) return;
      store = parsed;
      if(!store.activeProjectId || !store.projects[store.activeProjectId]) store.activeProjectId = store.order[0];
      if(!TIMEFRAMES.find(tf => tf.key === activeTF)) activeTF = TIMEFRAMES[0].key;
      renderAll();
      scheduleSave();
      alert('Backup erfolgreich geladen.');
    }catch(err){
      alert('Datei konnte nicht gelesen werden: ' + err.message);
    }finally{
      e.target.value = '';
    }
  };
  reader.readAsText(file);
});

/* Warn before leaving the page if a save attempt is still pending/failed,
   so unsaved changes aren't silently lost when the storage backend is unreliable. */
window.addEventListener('beforeunload', (e) => {
  if(saveRetryTimer){
    e.preventDefault();
    e.returnValue = '';
    return '';
  }
});

(async function init(){
  await loadData();
  renderAll();
})();
</script>
</body>
</html>
