<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>GNURS 303 — Live Quiz</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=IBM+Plex+Mono:wght@400;600&family=DM+Sans:wght@300;400;500;600&display=swap');

  :root {
    --navy: #0a1628;
    --mid: #132240;
    --panel: #1a2d50;
    --gold: #c8922a;
    --gold-light: #e8b84b;
    --text: #e8edf5;
    --muted: #7a90b0;
    --green: #2ecc85;
    --red: #e85555;
    --blue-acc: #4a9eff;
    --border: rgba(200,146,42,0.25);
    --answered: rgba(74,158,255,0.12);
    --radius: 8px;
  }

  * { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    font-family: 'DM Sans', sans-serif;
    background: var(--navy);
    color: var(--text);
    min-height: 100vh;
  }

  /* ── SPLASH ── */
  #splash {
    display: flex; flex-direction: column; align-items: center; justify-content: center;
    min-height: 100vh; padding: 40px 20px; text-align: center;
    background: radial-gradient(ellipse at 60% 30%, rgba(200,146,42,0.12) 0%, transparent 70%),
                radial-gradient(ellipse at 20% 80%, rgba(74,158,255,0.08) 0%, transparent 60%),
                var(--navy);
  }
  .splash-badge {
    font-family: 'IBM Plex Mono', monospace;
    font-size: 11px; letter-spacing: 3px; text-transform: uppercase;
    color: var(--gold); border: 1px solid var(--gold); padding: 5px 14px;
    border-radius: 2px; margin-bottom: 28px;
  }
  #splash h1 { font-size: clamp(26px,5vw,46px); font-weight: 600; line-height: 1.2; margin-bottom: 10px; }
  #splash h1 span { color: var(--gold-light); }
  #splash p { color: var(--muted); font-size: 15px; margin-bottom: 32px; max-width: 480px; }
  .info-grid {
    display: flex; gap: 24px; flex-wrap: wrap; justify-content: center; margin-bottom: 40px;
  }
  .info-box {
    background: var(--panel); border: 1px solid var(--border); border-radius: var(--radius);
    padding: 16px 24px; min-width: 120px;
  }
  .info-box .num { font-family: 'IBM Plex Mono', monospace; font-size: 28px; font-weight: 600; color: var(--gold-light); }
  .info-box .lbl { font-size: 11px; color: var(--muted); text-transform: uppercase; letter-spacing: 1px; margin-top: 2px; }
  .topics { text-align: left; background: var(--panel); border: 1px solid var(--border); border-radius: var(--radius); padding: 20px 28px; margin-bottom: 36px; max-width: 560px; width: 100%; }
  .topics h3 { font-size: 12px; text-transform: uppercase; letter-spacing: 2px; color: var(--gold); margin-bottom: 12px; font-family: 'IBM Plex Mono', monospace; }
  .topics ul { list-style: none; }
  .topics li { padding: 5px 0; font-size: 14px; color: var(--muted); }
  .topics li::before { content: '▸ '; color: var(--gold-light); }
  #startBtn {
    background: var(--gold); color: var(--navy); border: none; border-radius: var(--radius);
    padding: 16px 52px; font-size: 16px; font-weight: 600; cursor: pointer;
    font-family: 'DM Sans', sans-serif; letter-spacing: 0.5px;
    transition: background 0.2s, transform 0.1s;
  }
  #startBtn:hover { background: var(--gold-light); transform: translateY(-1px); }

  /* ── QUIZ SHELL ── */
  #quizShell { display: none; min-height: 100vh; }

  /* TOP BAR */
  #topBar {
    position: sticky; top: 0; z-index: 100;
    background: var(--mid); border-bottom: 1px solid var(--border);
    padding: 0 24px; height: 56px;
    display: flex; align-items: center; justify-content: space-between; gap: 16px;
  }
  .tb-left { font-size: 13px; color: var(--muted); font-family: 'IBM Plex Mono', monospace; }
  .tb-left strong { color: var(--text); }
  #timerBox {
    display: flex; align-items: center; gap: 8px;
    background: var(--panel); border: 1px solid var(--border); border-radius: 6px; padding: 6px 14px;
  }
  #timerBox .t-icon { font-size: 14px; }
  #timerDisplay { font-family: 'IBM Plex Mono', monospace; font-size: 18px; font-weight: 600; color: var(--gold-light); min-width: 58px; }
  #timerDisplay.warn { color: #ff9933; }
  #timerDisplay.danger { color: var(--red); animation: blink 1s infinite; }
  @keyframes blink { 0%,100%{opacity:1} 50%{opacity:0.4} }
  .tb-right { display: flex; gap: 10px; align-items: center; }
  #progressCount { font-size: 12px; color: var(--muted); font-family: 'IBM Plex Mono', monospace; }

  /* PROGRESS BAR */
  #progressBar {
    height: 3px; background: var(--panel);
    transition: none;
  }
  #progressFill { height: 100%; background: var(--gold); width: 0%; transition: width 0.4s; }

  /* LAYOUT */
  #quizBody { display: flex; height: calc(100vh - 59px); overflow: hidden; }

  /* NAVIGATOR */
  #navPanel {
    width: 220px; min-width: 220px; background: var(--mid); border-right: 1px solid var(--border);
    overflow-y: auto; padding: 16px;
    display: flex; flex-direction: column; gap: 0;
  }
  .nav-section { margin-bottom: 18px; }
  .nav-section-title { font-size: 10px; text-transform: uppercase; letter-spacing: 2px; color: var(--gold); font-family: 'IBM Plex Mono', monospace; margin-bottom: 8px; }
  .nav-grid { display: grid; grid-template-columns: repeat(5, 1fr); gap: 4px; }
  .nav-btn {
    aspect-ratio: 1; border-radius: 4px; border: 1px solid var(--border);
    background: var(--panel); color: var(--muted); font-size: 11px;
    font-family: 'IBM Plex Mono', monospace; cursor: pointer;
    transition: all 0.15s; display: flex; align-items: center; justify-content: center;
  }
  .nav-btn:hover { border-color: var(--gold); color: var(--gold-light); }
  .nav-btn.answered { background: rgba(74,158,255,0.2); border-color: var(--blue-acc); color: var(--blue-acc); }
  .nav-btn.current { background: var(--gold); border-color: var(--gold); color: var(--navy); font-weight: 600; }
  .nav-legend { margin-top: auto; padding-top: 12px; border-top: 1px solid var(--border); }
  .legend-item { display: flex; align-items: center; gap: 6px; font-size: 11px; color: var(--muted); margin-bottom: 6px; }
  .leg-dot { width: 10px; height: 10px; border-radius: 2px; flex-shrink: 0; }
  .leg-dot.unanswered { background: var(--panel); border: 1px solid var(--border); }
  .leg-dot.answered { background: var(--blue-acc); }
  .leg-dot.current { background: var(--gold); }

  /* MAIN CONTENT */
  #mainContent { flex: 1; overflow-y: auto; padding: 32px 40px; }

  .question-card {
    background: var(--panel); border: 1px solid var(--border); border-radius: 10px;
    padding: 28px 32px; margin-bottom: 16px; max-width: 820px;
  }
  .q-header { display: flex; align-items: flex-start; gap: 14px; margin-bottom: 20px; }
  .q-num {
    font-family: 'IBM Plex Mono', monospace; font-size: 11px; color: var(--gold);
    background: rgba(200,146,42,0.12); border: 1px solid var(--border);
    border-radius: 4px; padding: 3px 8px; white-space: nowrap; flex-shrink: 0; margin-top: 2px;
  }
  .q-text { font-size: 16px; line-height: 1.6; font-weight: 400; }

  .options { display: flex; flex-direction: column; gap: 10px; }
  .option-label {
    display: flex; align-items: flex-start; gap: 12px;
    background: rgba(255,255,255,0.03); border: 1px solid rgba(255,255,255,0.08);
    border-radius: var(--radius); padding: 13px 16px; cursor: pointer;
    transition: all 0.15s; position: relative;
  }
  .option-label:hover { border-color: rgba(200,146,42,0.4); background: rgba(200,146,42,0.05); }
  .option-label input[type="radio"] { display: none; }
  .option-label.selected { border-color: var(--blue-acc); background: rgba(74,158,255,0.1); }
  .option-key {
    font-family: 'IBM Plex Mono', monospace; font-size: 12px; font-weight: 600;
    color: var(--muted); min-width: 20px; flex-shrink: 0; padding-top: 1px;
  }
  .option-label.selected .option-key { color: var(--blue-acc); }
  .option-text { font-size: 14px; line-height: 1.55; }

  /* NAV CONTROLS */
  .q-controls { display: flex; gap: 12px; margin-top: 20px; max-width: 820px; }
  .btn-nav {
    background: var(--panel); border: 1px solid var(--border); color: var(--text);
    border-radius: var(--radius); padding: 10px 22px; font-size: 14px; cursor: pointer;
    font-family: 'DM Sans', sans-serif; transition: all 0.15s;
  }
  .btn-nav:hover { border-color: var(--gold); color: var(--gold-light); }
  .btn-nav:disabled { opacity: 0.3; cursor: not-allowed; }
  .btn-submit-final {
    margin-left: auto; background: var(--gold); color: var(--navy); border: none;
    border-radius: var(--radius); padding: 10px 28px; font-size: 14px; font-weight: 600;
    cursor: pointer; font-family: 'DM Sans', sans-serif; transition: background 0.15s;
  }
  .btn-submit-final:hover { background: var(--gold-light); }

  /* ── RESULTS ── */
  #results {
    display: none; min-height: 100vh; padding: 48px 24px;
    background: radial-gradient(ellipse at 50% 0%, rgba(200,146,42,0.1) 0%, transparent 60%), var(--navy);
  }
  .results-inner { max-width: 860px; margin: 0 auto; }
  .results-title { font-size: clamp(24px,4vw,38px); font-weight: 600; margin-bottom: 6px; }
  .results-title span { color: var(--gold-light); }
  .results-sub { color: var(--muted); font-size: 14px; margin-bottom: 36px; }
  .score-hero {
    background: var(--panel); border: 1px solid var(--border); border-radius: 12px;
    padding: 36px; text-align: center; margin-bottom: 28px;
  }
  .score-pct { font-family: 'IBM Plex Mono', monospace; font-size: 72px; font-weight: 600; line-height: 1; }
  .score-pct.pass { color: var(--green); }
  .score-pct.fail { color: var(--red); }
  .score-grade { font-size: 20px; color: var(--muted); margin-top: 8px; }
  .score-detail { display: flex; gap: 24px; justify-content: center; flex-wrap: wrap; margin-top: 20px; }
  .sd-item { text-align: center; }
  .sd-item .n { font-family: 'IBM Plex Mono', monospace; font-size: 26px; font-weight: 600; }
  .sd-item .l { font-size: 12px; color: var(--muted); text-transform: uppercase; letter-spacing: 1px; }
  .n.correct { color: var(--green); }
  .n.wrong { color: var(--red); }
  .n.skipped { color: var(--muted); }

  .section-breakdown { margin-bottom: 28px; }
  .section-breakdown h3 { font-size: 13px; text-transform: uppercase; letter-spacing: 2px; color: var(--gold); font-family: 'IBM Plex Mono', monospace; margin-bottom: 14px; }
  .section-row { display: flex; align-items: center; gap: 14px; margin-bottom: 10px; }
  .sr-name { font-size: 13px; color: var(--muted); width: 260px; flex-shrink: 0; }
  .sr-bar { flex: 1; height: 8px; background: var(--mid); border-radius: 4px; overflow: hidden; }
  .sr-fill { height: 100%; border-radius: 4px; transition: width 0.8s ease; }
  .sr-score { font-family: 'IBM Plex Mono', monospace; font-size: 12px; color: var(--text); width: 60px; text-align: right; }

  .review-list { display: flex; flex-direction: column; gap: 14px; }
  .review-item { background: var(--panel); border: 1px solid var(--border); border-radius: var(--radius); padding: 18px 22px; }
  .ri-top { display: flex; align-items: center; gap: 10px; margin-bottom: 8px; }
  .ri-num { font-family: 'IBM Plex Mono', monospace; font-size: 11px; color: var(--muted); }
  .ri-badge { font-size: 11px; font-weight: 600; padding: 2px 8px; border-radius: 4px; }
  .ri-badge.correct { background: rgba(46,204,133,0.15); color: var(--green); }
  .ri-badge.wrong { background: rgba(232,85,85,0.15); color: var(--red); }
  .ri-badge.skipped { background: rgba(122,144,176,0.15); color: var(--muted); }
  .ri-q { font-size: 14px; line-height: 1.5; margin-bottom: 10px; }
  .ri-answers { display: flex; flex-direction: column; gap: 5px; }
  .ri-ans { font-size: 13px; padding: 6px 10px; border-radius: 5px; }
  .ri-ans.your-wrong { background: rgba(232,85,85,0.1); color: var(--red); }
  .ri-ans.correct-ans { background: rgba(46,204,133,0.1); color: var(--green); }
  .ri-ans.your-correct { background: rgba(46,204,133,0.1); color: var(--green); }
  .ri-ans.neutral { background: rgba(255,255,255,0.03); color: var(--muted); }

  .results-actions { display: flex; gap: 12px; margin-top: 32px; flex-wrap: wrap; }
  .btn-retry { background: var(--gold); color: var(--navy); border: none; border-radius: var(--radius); padding: 12px 32px; font-size: 15px; font-weight: 600; cursor: pointer; font-family: 'DM Sans', sans-serif; }
  .btn-retry:hover { background: var(--gold-light); }
  .btn-review-toggle { background: var(--panel); border: 1px solid var(--border); color: var(--text); border-radius: var(--radius); padding: 12px 28px; font-size: 15px; cursor: pointer; font-family: 'DM Sans', sans-serif; }

  /* MODAL */
  .modal-overlay {
    display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.7);
    z-index: 999; align-items: center; justify-content: center;
  }
  .modal-overlay.open { display: flex; }
  .modal-box { background: var(--mid); border: 1px solid var(--border); border-radius: 12px; padding: 36px; max-width: 420px; width: 90%; text-align: center; }
  .modal-box h2 { font-size: 22px; margin-bottom: 10px; }
  .modal-box p { color: var(--muted); font-size: 14px; margin-bottom: 28px; line-height: 1.6; }
  .modal-btns { display: flex; gap: 12px; justify-content: center; }
  .modal-cancel { background: var(--panel); border: 1px solid var(--border); color: var(--text); border-radius: var(--radius); padding: 10px 24px; font-size: 14px; cursor: pointer; font-family: 'DM Sans', sans-serif; }
  .modal-confirm { background: var(--red); border: none; color: #fff; border-radius: var(--radius); padding: 10px 24px; font-size: 14px; font-weight: 600; cursor: pointer; font-family: 'DM Sans', sans-serif; }

  @media(max-width:700px){
    #navPanel{display:none}
    #mainContent{padding:20px 16px}
    .question-card{padding:20px 16px}
  }
</style>
</head>
<body>

<!-- SPLASH -->
<div id="splash">
  <div class="splash-badge">University of Ghana · School of Nursing & Midwifery</div>
  <h1>GNURS 303<br><span>Eye, Ear, Throat & Dental Nursing</span></h1>
  <p>Diseases & Abnormalities of the Nose · Tonsillitis · Foreign Bodies & Growths</p>
  <div class="info-grid">
    <div class="info-box"><div class="num">120</div><div class="lbl">Questions</div></div>
    <div class="info-box"><div class="num">60</div><div class="lbl">Minutes</div></div>
    <div class="info-box"><div class="num">50%</div><div class="lbl">Pass Mark</div></div>
    <div class="info-box"><div class="num">MCQ</div><div class="lbl">Format</div></div>
  </div>
  <div class="topics">
    <h3>Topics Covered</h3>
    <ul>
      <li>Nasal Polyps — Definition, Types, Aetiology, Clinical Features, Management</li>
      <li>Trauma to the Nose — Causes, Signs, Investigations, Surgical Care</li>
      <li>Foreign Bodies in the Nose — Presentation, Diagnosis, Removal</li>
      <li>Tonsillitis — Types, Pathophysiology, Treatment, Tonsillectomy</li>
      <li>Peritonsillar Abscess — Clinical Features, Complications, Nursing Care</li>
      <li>Foreign Body Aspiration (Throat) — Risk Factors, Signs, Management</li>
    </ul>
  </div>
  <button id="startBtn" onclick="startQuiz()">Begin Assessment</button>
</div>

<!-- QUIZ SHELL -->
<div id="quizShell">
  <div id="topBar">
    <div class="tb-left">GNURS 303 &nbsp;·&nbsp; <strong id="qCounter">Q1 of 120</strong></div>
    <div id="timerBox">
      <span class="t-icon">⏱</span>
      <div id="timerDisplay">60:00</div>
    </div>
    <div class="tb-right">
      <div id="progressCount">0 / 120 answered</div>
    </div>
  </div>
  <div id="progressBar"><div id="progressFill"></div></div>
  <div id="quizBody">
    <div id="navPanel">
      <div id="navSections"></div>
      <div class="nav-legend">
        <div class="legend-item"><div class="leg-dot unanswered"></div> Not answered</div>
        <div class="legend-item"><div class="leg-dot answered"></div> Answered</div>
        <div class="legend-item"><div class="leg-dot current"></div> Current</div>
      </div>
    </div>
    <div id="mainContent">
      <div id="questionArea"></div>
      <div class="q-controls">
        <button class="btn-nav" id="btnPrev" onclick="navigate(-1)">◀ Previous</button>
        <button class="btn-nav" id="btnNext" onclick="navigate(1)">Next ▶</button>
        <button class="btn-submit-final" onclick="openSubmitModal()">Submit Quiz</button>
      </div>
    </div>
  </div>
</div>

<!-- RESULTS -->
<div id="results"></div>

<!-- SUBMIT MODAL -->
<div class="modal-overlay" id="submitModal">
  <div class="modal-box">
    <h2>Submit Quiz?</h2>
    <p id="modalMsg">You still have unanswered questions. Are you sure you want to submit?</p>
    <div class="modal-btns">
      <button class="modal-cancel" onclick="closeModal()">Go Back</button>
      <button class="modal-confirm" onclick="submitQuiz()">Yes, Submit</button>
    </div>
  </div>
</div>

<script>
const QUESTIONS = [
  // ── SECTION 1: NASAL POLYPS (Q1–30) ──
  {s:"Nasal Polyps",q:"Which of the following best describes a nasal polyp?",o:["A cancerous tumour of the nasal mucosa","A noncancerous teardrop-shaped growth of inflamed nasal tissue","A hard, painful swelling attached firmly to the septum","A blood clot that forms in the nasal cavity after trauma"],a:1},
  {s:"Nasal Polyps",q:"Nasal polyps are classified into two main types. Which pairing is correct?",o:["Antrochoanal and Ethmoidal","Sphenoidal and Frontal","Cribriform and Maxillary","Adenoid and Palatine"],a:0},
  {s:"Nasal Polyps",q:"Antrochoanal polyps characteristically arise from the:",o:["Ethmoid sinus","Sphenoid sinus","Maxillary sinus","Frontal sinus"],a:2},
  {s:"Nasal Polyps",q:"Compared with ethmoidal polyps, antrochoanal polyps tend to be:",o:["Multiple and bilateral","Single and unilateral","Multiple and unilateral","Single and bilateral"],a:1},
  {s:"Nasal Polyps",q:"Which of the following diseases is NOT commonly associated with nasal polyps?",o:["Allergic rhinitis","Asthma","Cystic fibrosis","Otosclerosis"],a:3},
  {s:"Nasal Polyps",q:"The visual appearance of mature multiple nasal polyps is often compared to:",o:["Cauliflower heads","Seedless, peeled grapes","Bunch of cherries","Cotton balls"],a:1},
  {s:"Nasal Polyps",q:"A key characteristic that distinguishes nasal polyps on physical examination is that they are:",o:["Fixed and very tender","Freely movable and non-tender","Hard and non-movable","Attached to the turbinate and painful"],a:1},
  {s:"Nasal Polyps",q:"Anosmia as a clinical feature of nasal polyps refers to:",o:["Loss of taste","Loss of hearing","Loss of smell","Loss of vision"],a:2},
  {s:"Nasal Polyps",q:"Which diagnostic investigation would best confirm whether a nasal polyp is malignant?",o:["Nasal endoscopy","X-ray of sinuses","Biopsy of the polyp","CT scan"],a:2},
  {s:"Nasal Polyps",q:"On CT scan or MRI, nasal polyps characteristically appear as:",o:["Bright white calcified spots","Cloudy (opaque) areas","Dark cystic voids","Air-filled spaces"],a:1},
  {s:"Nasal Polyps",q:"Which nasal steroid spray action is described in the management of nasal polyps?",o:["It is given indefinitely to prevent recurrence","It is continued only until symptoms subside","It replaces the need for surgery in all cases","It is only given post-operatively"],a:1},
  {s:"Nasal Polyps",q:"Ephedrine used in managing nasal polyps acts as a:",o:["Antibiotic","Corticosteroid","Nasal decongestant","Antihistamine"],a:2},
  {s:"Nasal Polyps",q:"Clemastine fumarate, mentioned in nasal polyp management, is classified as a/an:",o:["Antibiotic","Antihistamine","Decongestant","Antifungal"],a:1},
  {s:"Nasal Polyps",q:"Which surgical procedure is appropriate for small or isolated nasal polyps and can be performed under local anaesthesia?",o:["Rhinoplasty","Endoscopic sinus surgery","Polypectomy","Septoplasty"],a:2},
  {s:"Nasal Polyps",q:"During a polypectomy, after the polyp is incised with a snare-like instrument, the next immediate steps include:",o:["Wound suturing and discharge","Cauterization of bleeding and intranasal packing","Nasal splinting and IV antibiotics","CT scan and biopsy"],a:1},
  {s:"Nasal Polyps",q:"After endoscopic sinus surgery for nasal polyps, which medication is often prescribed to reduce recurrence?",o:["Oral antibiotics","Corticosteroid nasal spray","Systemic antifungals","Nasal decongestant drops"],a:1},
  {s:"Nasal Polyps",q:"Which pre-operative care instruction is specific to patients undergoing nasal polyp surgery?",o:["Teach abdominal breathing exercises","Teach mouth breathing exercises","Advise deep nasal blowing","Restrict all oral fluids"],a:1},
  {s:"Nasal Polyps",q:"What is the recommended positioning post-operatively following nasal polyp surgery?",o:["Supine flat","Trendelenburg","Semi-Fowler's to high Fowler's","Prone"],a:2},
  {s:"Nasal Polyps",q:"Why are sinus rinses with warm saline continued post-operatively after nasal polyp surgery?",o:["To administer antibiotics topically","To discourage regrowth of polyps","To reduce post-operative fever","To maintain anaesthetic effect"],a:1},
  {s:"Nasal Polyps",q:"A post-operative patient keeps swallowing frequently after nasal polyp surgery. This should alert the nurse to:",o:["Normal post-anaesthetic response","Possible nausea from anaesthesia","Possible nasal bleeding","Excess nasal discharge"],a:2},
  {s:"Nasal Polyps",q:"In anterior nosebleeds following nasal surgery, the slide states the area may be treated with silver nitrate and gelfoam, or by:",o:["Laser ablation","Electrocautery","Nasal tamponade only","Systemic haemostatic agents"],a:1},
  {s:"Nasal Polyps",q:"For how long are nasal packs typically maintained after polypectomy before removal?",o:["6–12 hours","12–24 hours","24–48 hours","48–72 hours"],a:2},
  {s:"Nasal Polyps",q:"After nasal pack removal, the bridge of the nose is compressed against the septum for how many minutes?",o:["1–2 minutes","3–5 minutes","5–10 minutes","10–15 minutes"],a:2},
  {s:"Nasal Polyps",q:"Kartagener's syndrome, associated with nasal polyps, is characterized by the triad of situs inversus, chronic sinusitis, and:",o:["Asthma","Bronchiectasis","Cystic fibrosis","Rhinitis"],a:1},
  {s:"Nasal Polyps",q:"Nasal mastocytosis is characterized by abnormal accumulation of which cells?",o:["Eosinophils","Mast cells","Neutrophils","Plasma cells"],a:1},
  {s:"Nasal Polyps",q:"Churg-Strauss syndrome (EGPA), associated with nasal polyps, primarily involves:",o:["Joint inflammation","Blood vessel inflammation","Nerve demyelination","Renal tubule inflammation"],a:1},
  {s:"Nasal Polyps",q:"Which of the following is a preventive measure for nasal polyps?",o:["Use nasal decongestants daily","Minimize exposure to allergens","Increase intake of spicy foods","Avoid all nasal sprays permanently"],a:1},
  {s:"Nasal Polyps",q:"Post-nasal drip, loss of taste, and sinusitis are clinical manifestations of:",o:["Nasal trauma","Nasal polyps","Septal haematoma","Peritonsillar abscess"],a:1},
  {s:"Nasal Polyps",q:"In culture and sensitivity of nasal swab for polyp management, when is this investigation primarily performed?",o:["After surgery","Before surgery","Only if cancer is suspected","When CT scan is inconclusive"],a:1},
  {s:"Nasal Polyps",q:"Which topical vasoconstrictors may be prescribed in nasal polyp post-operative management? (Select the group mentioned in the slides)",o:["Adrenaline, cocaine 0.5%, phenylephrine","Atropine, homatropine, scopolamine","Lignocaine, prilocaine, bupivacaine","Oxymetazoline, xylometazoline, naphazoline"],a:0},

  // ── SECTION 2: NASAL TRAUMA (Q31–55) ──
  {s:"Nasal Trauma",q:"Which of the following best defines trauma to the nose?",o:["Inflammation of nasal mucosa due to allergy","Any injury to the nose or related structure that may lead to bleeding, deformity, or impaired smell","A benign growth arising from inflamed nasal tissue","Chronic infection of the nasal sinuses"],a:1},
  {s:"Nasal Trauma",q:"Epistaxis is a clinical term for:",o:["Fracture of the nasal septum","Bleeding from the nose","Inflammation of the nasal cavity","A foreign body in the nose"],a:1},
  {s:"Nasal Trauma",q:"Crepitus heard or felt on palpation of the nose indicates:",o:["Presence of a polyp","Nasal fracture","Nasal abscess","Allergic swelling"],a:1},
  {s:"Nasal Trauma",q:"Which type of nasal fracture does the slide describe as most difficult to treat?",o:["Frontal bone fractures","Nasal tip fractures","Fractures involving the nasal septum","Fractures of the turbinates"],a:2},
  {s:"Nasal Trauma",q:"Rhinoplasty is defined as:",o:["Surgical removal of polyps","Endoscopic cleaning of sinuses","Plastic surgery to repair or change the shape of the nose","Surgical correction of a deviated septum"],a:2},
  {s:"Nasal Trauma",q:"Leakage of cerebrospinal fluid through the nostrils following nasal trauma is known as:",o:["Rhinorrhoea","CSF rhinorrhoea","Post-nasal drip","Epistaxis"],a:1},
  {s:"Nasal Trauma",q:"Ecchymosis of the tissues around the eye following nasal trauma indicates:",o:["Orbital cellulitis","Bruising or discoloration from bleeding beneath the skin","Conjunctival haemorrhage","Retinal detachment"],a:1},
  {s:"Nasal Trauma",q:"A septal haematoma following nasal trauma is a collection of:",o:["Pus between the mucosa and cartilage","Blood between the mucosa and cartilage of the septum","Fluid in the maxillary sinus","Cerebrospinal fluid in the septum"],a:1},
  {s:"Nasal Trauma",q:"Which investigation for nasal trauma may be ordered to check for lower respiratory tract involvement?",o:["Nasal endoscopy","Culture and sensitivity","Pulmonary function test","Audiometry"],a:2},
  {s:"Nasal Trauma",q:"In the intentional causes of nasal trauma, which of the following is listed?",o:["Sporting injuries","Nasal piercing complications","Criminal assault","Transportation accidents"],a:2},
  {s:"Nasal Trauma",q:"Moffett's method is referenced in nasal trauma management as a form of:",o:["Post-operative positioning","Local preparation of the nasal area before surgery","Suturing technique for nasal lacerations","Technique for removing septal haematoma"],a:1},
  {s:"Nasal Trauma",q:"Pre-operatively for nasal trauma, the nurse should inform the patient about nasal packs or splints and give an estimate of:",o:["The cost of surgery","How long the packs will remain in place","Likely dietary restrictions for 6 weeks","The exact degree of cosmetic change"],a:1},
  {s:"Nasal Trauma",q:"The slide states that after nasal surgery for trauma, full resolution of swelling may take up to:",o:["2 weeks","4 weeks","6 weeks","3 months"],a:2},
  {s:"Nasal Trauma",q:"Post-operatively for nasal trauma, helping the patient into a position to aid drainage and reduce venous congestion serves to:",o:["Increase blood pressure","Promote patient comfort and reduce swelling","Speed up absorption of nasal packs","Prevent post-operative nausea"],a:1},
  {s:"Nasal Trauma",q:"Which of the following is a chemical cause of nasal trauma?",o:["Animal bite","Foreign objects picked by children","Breathing or sniffing of irritating substances","Being struck with a baseball bat"],a:2},
  {s:"Nasal Trauma",q:"A patient with nasal packs post-operatively may develop hypoxia and hypercarbia due to:",o:["Increased oxygen demand from fever","Hypoventilation resulting from airway obstruction by the packs","Side effects of corticosteroids","Excessive blood loss"],a:1},
  {s:"Nasal Trauma",q:"Rhinitis as a clinical sign of nasal trauma refers to:",o:["Nasal fracture with visible displacement","Inflammation of the nasal mucous membranes","A collection of blood under the skin","A bone spur on the nasal septum"],a:1},
  {s:"Nasal Trauma",q:"Which diagnostic imaging modality is NOT listed under investigations for nasal trauma in the slides?",o:["CT scan","MRI scan","X-ray","PET scan"],a:3},
  {s:"Nasal Trauma",q:"In the post-operative care for nasal trauma surgery, a prosthesis may be inserted. The nurse should:",o:["Remove it daily for cleaning without instruction","Teach the patient how to care for the inserted prosthesis","Leave prosthesis care entirely to the surgeon","Wrap the prosthesis in sterile gauze without explanation"],a:1},
  {s:"Nasal Trauma",q:"What is the conclusion drawn in the slides about the role of the nose?",o:["The nose is primarily important for aesthetic appearance","The nose is a vital organ for gas exchange; foreign body or trauma can have life-threatening consequences","Nasal trauma is rarely serious and heals spontaneously","Nasal polyps are more dangerous than nasal trauma"],a:1},
  {s:"Nasal Trauma",q:"Which of the following is listed as a physical cause of nasal trauma from sports?",o:["Allergic exposure during exercise","Being hit in the face by a baseball or hockey ball at high speed","Inhalation of dry air during outdoor sports","Pressure changes during swimming"],a:1},
  {s:"Nasal Trauma",q:"Reducing venous congestion and crust formation post-nasal surgery is best achieved through:",o:["Maintaining the flat supine position","Applying measures to keep the nasal airway open and reduce congestion","Withholding all oral fluids","Applying warm compresses to the nasal bridge"],a:1},
  {s:"Nasal Trauma",q:"Which concern should be raised pre-operatively about the consequences of nasal packs on function?",o:["They will permanently reduce nasal cavity size","They can cause mouth-breathing and temporary loss of smell","They will disfigure the nasal bridge permanently","They increase risk of orbital cellulitis"],a:1},
  {s:"Nasal Trauma",q:"Assessment and prevention of facial oedema is listed specifically as a post-operative nursing care goal in:",o:["Nasal polyp surgery only","Nasal trauma surgery","Tonsillectomy","Foreign body removal from the nose"],a:1},
  {s:"Nasal Trauma",q:"A close or open reduction of a nasal fracture can be performed under:",o:["Topical anaesthesia only","Local or general anaesthesia","Sedation only, not full anaesthesia","IV analgesia without anaesthesia"],a:1},

  // ── SECTION 3: FOREIGN BODIES IN THE NOSE (Q56–75) ──
  {s:"Foreign Body — Nose",q:"What is the primary age group most commonly affected by foreign bodies in the nose?",o:["Elderly adults aged 65 and above","Toddlers and children aged 1–8 years","Teenagers aged 13–17 years","Young adults aged 18–25 years"],a:1},
  {s:"Foreign Body — Nose",q:"Children develop the ability to pick up objects at approximately what age, making nasal foreign bodies more likely?",o:["3 months","6 months","9 months","12 months"],a:2},
  {s:"Foreign Body — Nose",q:"In the early stages of a nasal foreign body, the nasal discharge tends to be:",o:["Thick and yellow","Blood-stained","Clear","Green and purulent"],a:2},
  {s:"Foreign Body — Nose",q:"Which symptom indicates the foreign body may have migrated posteriorly into the throat?",o:["Nasal congestion only","Prolonged foul odour","Choking, wheezing, and inability to talk","Post-nasal drip"],a:2},
  {s:"Foreign Body — Nose",q:"Impetigo of the skin under the nose in a foreign body case results from:",o:["Secondary bacterial infection spreading from sinuses","Continuous rubbing of the irritated skin under the nostrils","Exposure to chemical foreign bodies","Medication side effects"],a:1},
  {s:"Foreign Body — Nose",q:"Haematemesis as a sign of a nasal foreign body most likely results from:",o:["Direct nasal bleeding only","Swallowed blood from a nasal bleed","Gastrointestinal co-morbidity","Trauma to the throat during removal"],a:1},
  {s:"Foreign Body — Nose",q:"A stimulant like pepper may be used in foreign body management to:",o:["Loosen the object chemically","Stimulate sneezing to expel the foreign body","Anaesthetize the nasal mucosa","Dissolve organic foreign bodies"],a:1},
  {s:"Foreign Body — Nose",q:"If a foreign body in the nose is metallic, which specialised instrument may assist in its removal?",o:["Vacuum suction cup","Long magnetized instrument","Balloon catheter","Heated wire loop"],a:1},
  {s:"Foreign Body — Nose",q:"The balloon catheter technique for nasal foreign body removal works by:",o:["Grasping the object with forceps under direct vision","Passing the catheter past the object, inflating the balloon, then pulling back","Injecting saline to float the object out","Advancing the object into the stomach for digestion"],a:1},
  {s:"Foreign Body — Nose",q:"When examining for a nasal foreign body that is deeply placed, which investigation is used alongside CT scan and X-ray?",o:["MRI","Ultrasonography","Fibreoptic camera","Fluoroscopy"],a:2},
  {s:"Foreign Body — Nose",q:"Thick, yellow, and foul-smelling nasal discharge in a child is a late-stage indicator of:",o:["Nasal polyp","Nasal trauma","Nasal foreign body with sinusitis","Allergic rhinitis"],a:2},
  {s:"Foreign Body — Nose",q:"Which of the following is NOT a listed clinical manifestation of a foreign body in the nose?",o:["Prolonged foul odour","Nasal bleeding","Anosmia (complete loss of smell)","Difficulty breathing through the nose"],a:2},
  {s:"Foreign Body — Nose",q:"A patient is sedated before nasal foreign body removal. This is most appropriate for:",o:["All paediatric patients","Cooperative adult patients","Anxious patients who cannot tolerate awake procedures","Patients with a metallic foreign body"],a:2},
  {s:"Foreign Body — Nose",q:"Antibiotics are given in nasal foreign body management when:",o:["The object has been removed successfully","Infection is anticipated or already present","The patient is allergic to the foreign material","The object is organic"],a:1},
  {s:"Foreign Body — Nose",q:"Long tweezers or hook/loop-tipped instruments are used to remove a nasal foreign body by:",o:["Dissolving it","Gently suctioning or grasping it","Magnetizing it","Pushing it into the throat"],a:1},
  {s:"Foreign Body — Nose",q:"Nasal foreign bodies in adults are described in the slides as occurring most commonly in association with:",o:["Occupational exposure to chemicals","Car accidents where an object is pushed into the nose","Drug abuse","Dental procedures"],a:1},
  {s:"Foreign Body — Nose",q:"The sensation of 'something in the nose' as a symptom of a nasal foreign body is caused by:",o:["Mucosal swelling from allergy","Mechanical stimulation of nasal mucosal nerve endings","Post-nasal drip","Rhinitis medicamentosa"],a:1},
  {s:"Foreign Body — Nose",q:"Most nasal foreign bodies can be seen with:",o:["MRI alone","Good lighting and a few instruments","General anaesthesia and endoscopy","X-ray and CT scan combined"],a:1},
  {s:"Foreign Body — Nose",q:"Management of bleeding that occurs during nasal foreign body removal is described as:",o:["Apply topical antifibrinolytics","Manage the bleeding if it occurs","Pack with ribbon gauze immediately","Transfer to theatre for haemostasis"],a:1},
  {s:"Foreign Body — Nose",q:"Why is it important to prevent a nasal foreign body from being pushed further into the nose during removal?",o:["It causes permanent septal deviation","It may dislodge into the throat causing airway obstruction","It damages the olfactory nerve permanently","It triggers systemic allergic reaction"],a:1},

  // ── SECTION 4: TONSILLITIS (Q76–100) ──
  {s:"Tonsillitis",q:"Tonsillitis is defined as inflammation and infection of the tonsils, which consist of pairs of lymph tissue in the:",o:["Oral and hypopharyngeal passages","Nasal and oropharyngeal passages","Laryngeal and tracheal passages","Adenoid and tonsillar rings only"],a:1},
  {s:"Tonsillitis",q:"Which type of tonsillitis is characterized by several different acute episodes within a year?",o:["Chronic tonsillitis","Acute tonsillitis","Recurrent tonsillitis","Membranous tonsillitis"],a:2},
  {s:"Tonsillitis",q:"Acute tonsillitis symptoms typically last:",o:["Less than 24 hours","3–4 days but may last up to 2 weeks","3–6 weeks","More than 2 months"],a:1},
  {s:"Tonsillitis",q:"Chronic tonsillitis is characterised by an ongoing sore throat and:",o:["High fever","Foul-smelling breath","Bilateral ear pain","Stridor"],a:1},
  {s:"Tonsillitis",q:"Which of the following viruses is listed as a cause of tonsillitis?",o:["Herpes simplex virus","Adenovirus","Rhinovirus","Coronavirus"],a:1},
  {s:"Tonsillitis",q:"Which bacterial genus is specifically listed as a cause of tonsillitis in the slides?",o:["Staphylococcus","Streptococcus","Pseudomonas","Haemophilus"],a:1},
  {s:"Tonsillitis",q:"The pathophysiology of tonsillitis begins with:",o:["Direct fungal invasion of the tonsils","Bacterial or viral pharyngitis spreading to infect the tonsils","Autoimmune destruction of tonsillar tissue","Haematogenous spread from the middle ear"],a:1},
  {s:"Tonsillitis",q:"As tonsillitis progresses, inflammation and oedema of tonsillar tissue makes the child:",o:["Breathe only through the nose","Force the child to breathe through the mouth","Develop bilateral hearing loss","Develop complete loss of voice"],a:1},
  {s:"Tonsillitis",q:"Advanced tonsillitis infection can result in cellulitis or abscess formation, which may require:",o:["Antiviral therapy","IV corticosteroids","Drainage","Radiation therapy"],a:2},
  {s:"Tonsillitis",q:"Which of the following is NOT listed among the clinical manifestations of tonsillitis?",o:["Fever and chills","Enlarged and reddened tonsils","Epistaxis","Difficulty swallowing"],a:2},
  {s:"Tonsillitis",q:"Malaise as a clinical feature of tonsillitis means:",o:["Productive cough with sputum","A general feeling of discomfort, illness, or lack of wellbeing","Specific joint pain","Peripheral oedema"],a:1},
  {s:"Tonsillitis",q:"Which diagnostic investigation is MOST targeted at identifying the causative organism in tonsillitis?",o:["WBC count alone","Nasal endoscopy","Pharyngeal swabs","CT scan of the neck"],a:2},
  {s:"Tonsillitis",q:"An elevated WBC count in tonsillitis suggests:",o:["Viral aetiology only","Presence of infection or inflammation","Autoimmune cause","Malignancy"],a:1},
  {s:"Tonsillitis",q:"The medical management of bacterial tonsillitis includes adequate hydration, rest, antipyretics, analgesics, and:",o:["Antifungals","Corticosteroids","Antivirals","Antibiotics"],a:3},
  {s:"Tonsillitis",q:"Tonsillectomy is the surgical:",o:["Removal of adenoids","Removal of the tonsils","Drainage of a peritonsillar abscess","Incision and drainage of quinsy"],a:1},
  {s:"Tonsillitis",q:"Adenoidectomy specifically refers to:",o:["Surgical removal of the tonsils","Surgical removal of the adenoids","Drainage of tonsillar crypts","Laser ablation of tonsillar tissue"],a:1},
  {s:"Tonsillitis",q:"Which condition caused by tonsillitis can block the pharynx and cause sleep apnoea?",o:["Acute otitis media","Peritonsillar abscess (quinsy)","Acute rhinitis","Chronic sinusitis"],a:1},
  {s:"Tonsillitis",q:"Hearing loss due to chronic otitis media is listed as an indication for:",o:["Nasal polypectomy","Rhinoplasty","Tonsillectomy/adenoidectomy","Sinus surgery"],a:2},
  {s:"Tonsillitis",q:"Pre-operative care for tonsillectomy includes preparation that is listed as physical but:",o:["Includes extensive skin preparation","Includes no skin preparation","Focuses only on psychological aspects","Excludes all spiritual needs"],a:1},
  {s:"Tonsillitis",q:"In which age groups is tonsillitis stated to be most prevalent?",o:["Neonates and infants","Children and adolescents","Middle-aged adults","Elderly patients"],a:1},
  {s:"Tonsillitis",q:"Which tonsillitis complication involving the ear is listed in the slides?",o:["Mastoiditis","Labyrinthitis","Acute otitis media","Tympanosclerosis"],a:2},
  {s:"Tonsillitis",q:"Acute rhinitis and acute sinusitis are listed as complications of:",o:["Foreign body in the nose","Nasal polyps","Tonsillitis","Nasal trauma"],a:2},
  {s:"Tonsillitis",q:"Worsening of which systemic conditions is listed as an indication for tonsillectomy?",o:["Hypertension and diabetes","Asthma and rheumatic fever","Chronic renal failure and lupus","COPD and bronchitis"],a:1},
  {s:"Tonsillitis",q:"Corine bacterium diphtheria (Corynebacterium diphtheriae) as a cause of tonsillitis is associated with which serious disease?",o:["Scarlet fever","Whooping cough","Diphtheria","Typhoid"],a:2},
  {s:"Tonsillitis",q:"Which surgical preparation note is unique to tonsillectomy compared to most other surgeries?",o:["Extensive shaving of the area","No skin preparation is required","Special antiseptic scrubbing of the oral cavity","Insertion of nasal packs pre-operatively"],a:1},

  // ── SECTION 5: PERITONSILLAR ABSCESS & POST-OP TONSIL CARE (Q101–112) ──
  {s:"Peritonsillar Abscess",q:"Peritonsillar abscess is also known as:",o:["Retropharyngeal abscess","Ludwig's angina","Quinsy","Parapharyngeal abscess"],a:2},
  {s:"Peritonsillar Abscess",q:"Which clinical feature is specific to peritonsillar abscess and NOT typically seen in simple tonsillitis?",o:["Fever and sore throat","Difficulty swallowing","Palpable lymph glands at the angle of the mandible","Enlarged tonsils"],a:2},
  {s:"Peritonsillar Abscess",q:"Halitosis in peritonsillar abscess refers to:",o:["Nasal discharge","Foul-smelling breath","Earache","Discharge from the ears"],a:1},
  {s:"Peritonsillar Abscess",q:"Discharge from the ears as a feature of peritonsillar abscess suggests extension of infection involving the:",o:["External ear canal","Eustachian tube and middle ear","Cochlea","Mastoid directly"],a:1},
  {s:"Peritonsillar Abscess",q:"In post-operative tonsillectomy care, placing the patient in a prone position with head turned to one side is intended to:",o:["Prevent aspiration of blood and secretions","Improve venous return to the heart","Reduce post-operative pain","Aid removal of nasal packs"],a:0},
  {s:"Peritonsillar Abscess",q:"When a post-tonsillectomy patient is conscious and the swallowing reflex has returned, the nurse should:",o:["Keep the patient flat","Prop the patient up","Sit the patient upright at 90 degrees immediately","Apply ice packs to the throat"],a:1},
  {s:"Peritonsillar Abscess",q:"What type of bleeding is considered most critical to report immediately after tonsillectomy?",o:["Oozing of pink saliva","Haemoptysis that is severe and haematemesis that becomes dark or bright red","Minor blood-streaked secretions","Expected small amount of bleeding in first 2 hours"],a:1},
  {s:"Peritonsillar Abscess",q:"Ice cubes are served for the post-tonsillectomy patient to suck after:",o:["The patient first regains consciousness","Bleeding has subsided","The first post-operative meal is given","Analgesics have been administered"],a:1},
  {s:"Peritonsillar Abscess",q:"Which food type is avoided after tonsillectomy until healing is complete?",o:["Soft yoghurt and non-milk cold creams","Spicy, hot, rough, and hard foods","Room-temperature porridge","Blended fruit smoothies"],a:1},
  {s:"Peritonsillar Abscess",q:"Copious fluid intake is encouraged after tonsillectomy primarily to:",o:["Dilute any post-operative medications","Maintain hydration and aid healing","Replace blood lost during surgery","Cool the surgical site"],a:1},
  {s:"Peritonsillar Abscess",q:"A patient's education on discharge after tonsillectomy should include observation for fever, pain, infection, and:",o:["Return to full diet immediately","Bleeding","Unrestricted physical activity","Resuming work the next day"],a:1},
  {s:"Peritonsillar Abscess",q:"Oral hygiene maintenance with saline after tonsillectomy helps to:",o:["Anaesthetize the surgical site","Remove sutures","Keep the area clean and reduce infection risk","Neutralize gastric acid"],a:2},

  // ── SECTION 6: FOREIGN BODY ASPIRATION / THROAT (Q113–120) ──
  {s:"Foreign Body — Throat",q:"Which age group is described as most susceptible to airway foreign body aspiration?",o:["Neonates 0–3 months","Children 9–30 months","School-age children 6–12 years","Elderly adults over 65"],a:1},
  {s:"Foreign Body — Throat",q:"'Penetration syndrome' in foreign body aspiration is defined as:",o:["Progressive stridor over several days","Sudden onset of choking and intractable cough with or without vomiting","Gradual onset of wheeze and fever","Bilateral wheezing on auscultation"],a:1},
  {s:"Foreign Body — Throat",q:"Foreign bodies lodging in the larynx and trachea can cause complete obstruction, potentially leading to:",o:["Bronchiectasis only","Sudden death","Chronic cough only","Pneumonia after weeks"],a:1},
  {s:"Foreign Body — Throat",q:"Inspiratory stridor with respiratory distress and supraclavicular/intercostal indrawing indicates the foreign body is likely located in the:",o:["Distal bronchus","Oesophagus","Larynx or subglottic area","Right main bronchus"],a:2},
  {s:"Foreign Body — Throat",q:"Which gender is stated in the slides to be more susceptible to foreign body aspiration?",o:["Female","Male","Both equally","Only determined by age, not gender"],a:1},
  {s:"Foreign Body — Throat",q:"Absence of molars before age 4 years predisposes to aspiration because children:",o:["Have weaker immune systems","Cannot chew certain foods adequately","Have narrower tracheas","Produce less saliva for lubrication"],a:1},
  {s:"Foreign Body — Throat",q:"Which of the following is an advanced complication of a retained foreign body in the airway?",o:["Simple nasal polyp formation","Bronchiectasis, lung abscess, haemoptysis","Upper respiratory tract infection only","Adenoid hypertrophy"],a:1},
  {s:"Foreign Body — Throat",q:"On chest X-ray, which finding can suggest air trapping caused by a bronchial foreign body?",o:["Bilateral opacification","Unilateral hyperinflation","Complete white-out of affected lobe","Normal chest X-ray in all cases"],a:1},
];

// State
let currentQ = 0;
let answers = new Array(120).fill(null);
let timerInterval = null;
let timeLeft = 60 * 60;
let quizSubmitted = false;
let reviewVisible = false;

function startQuiz() {
  document.getElementById('splash').style.display = 'none';
  document.getElementById('quizShell').style.display = 'block';
  buildNavPanel();
  renderQuestion(0);
  startTimer();
}

function buildNavPanel() {
  const sections = {};
  QUESTIONS.forEach((q,i) => {
    if (!sections[q.s]) sections[q.s] = [];
    sections[q.s].push(i);
  });
  const container = document.getElementById('navSections');
  container.innerHTML = '';
  Object.entries(sections).forEach(([sec, indices]) => {
    const div = document.createElement('div');
    div.className = 'nav-section';
    div.innerHTML = `<div class="nav-section-title">${sec}</div><div class="nav-grid" id="navGrid_${indices[0]}"></div>`;
    container.appendChild(div);
    const grid = div.querySelector('.nav-grid');
    indices.forEach(i => {
      const btn = document.createElement('button');
      btn.className = 'nav-btn';
      btn.id = `nav_${i}`;
      btn.textContent = i+1;
      btn.onclick = () => renderQuestion(i);
      grid.appendChild(btn);
    });
  });
}

function renderQuestion(index) {
  currentQ = index;
  const q = QUESTIONS[index];
  updateNavHighlight();
  document.getElementById('qCounter').textContent = `Q${index+1} of 120`;

  const area = document.getElementById('questionArea');
  area.innerHTML = `
    <div class="question-card">
      <div class="q-header">
        <div class="q-num">Q${index+1}</div>
        <div class="q-text">${q.q}</div>
      </div>
      <div class="options">
        ${q.o.map((opt,oi)=>`
          <label class="option-label ${answers[index]===oi?'selected':''}" onclick="selectAnswer(${index},${oi},this)">
            <input type="radio" name="q${index}" ${answers[index]===oi?'checked':''}>
            <span class="option-key">${String.fromCharCode(65+oi)}.</span>
            <span class="option-text">${opt}</span>
          </label>`).join('')}
      </div>
    </div>`;

  document.getElementById('btnPrev').disabled = index === 0;
  document.getElementById('btnNext').disabled = index === QUESTIONS.length-1;
  updateProgress();
  // scroll to top
  document.getElementById('mainContent').scrollTop = 0;
}

function selectAnswer(qIdx, optIdx, labelEl) {
  answers[qIdx] = optIdx;
  document.querySelectorAll('.option-label').forEach(l=>l.classList.remove('selected'));
  labelEl.classList.add('selected');
  updateNavHighlight();
  updateProgress();
}

function updateNavHighlight() {
  QUESTIONS.forEach((_,i) => {
    const btn = document.getElementById(`nav_${i}`);
    if (!btn) return;
    btn.className = 'nav-btn';
    if (answers[i] !== null) btn.classList.add('answered');
    if (i === currentQ) btn.classList.add('current');
  });
}

function updateProgress() {
  const answered = answers.filter(a=>a!==null).length;
  document.getElementById('progressCount').textContent = `${answered} / 120 answered`;
  document.getElementById('progressFill').style.width = `${(answered/120)*100}%`;
}

function navigate(dir) {
  const next = currentQ + dir;
  if (next >= 0 && next < QUESTIONS.length) renderQuestion(next);
}

function startTimer() {
  timerInterval = setInterval(() => {
    timeLeft--;
    const display = document.getElementById('timerDisplay');
    const m = Math.floor(timeLeft/60);
    const s = timeLeft%60;
    display.textContent = `${String(m).padStart(2,'0')}:${String(s).padStart(2,'0')}`;
    display.className = '';
    if (timeLeft <= 300) display.classList.add('warn');
    if (timeLeft <= 60) display.classList.add('danger');
    if (timeLeft <= 0) {
      clearInterval(timerInterval);
      submitQuiz();
    }
  }, 1000);
}

function openSubmitModal() {
  const answered = answers.filter(a=>a!==null).length;
  const unanswered = 120 - answered;
  const msg = unanswered > 0
    ? `You have ${unanswered} unanswered question(s). Unanswered questions will be marked wrong. Are you sure?`
    : `All 120 questions answered. Ready to submit?`;
  document.getElementById('modalMsg').textContent = msg;
  document.getElementById('submitModal').classList.add('open');
}
function closeModal() { document.getElementById('submitModal').classList.remove('open'); }

function submitQuiz() {
  if (quizSubmitted) return;
  quizSubmitted = true;
  clearInterval(timerInterval);
  closeModal();
  document.getElementById('quizShell').style.display = 'none';
  showResults();
}

function showResults() {
  let correct=0, wrong=0, skipped=0;
  QUESTIONS.forEach((q,i) => {
    if (answers[i] === null) skipped++;
    else if (answers[i] === q.a) correct++;
    else wrong++;
  });
  const pct = Math.round((correct/120)*100);
  const pass = pct >= 50;

  // Section breakdown
  const sections = {};
  QUESTIONS.forEach((q,i) => {
    if (!sections[q.s]) sections[q.s] = {total:0,correct:0};
    sections[q.s].total++;
    if (answers[i] === q.a) sections[q.s].correct++;
  });

  const secHTML = Object.entries(sections).map(([name,data]) => {
    const spct = Math.round((data.correct/data.total)*100);
    const col = spct>=70?'var(--green)':spct>=50?'var(--gold)':'var(--red)';
    return `<div class="section-row">
      <div class="sr-name">${name}</div>
      <div class="sr-bar"><div class="sr-fill" style="width:${spct}%;background:${col}"></div></div>
      <div class="sr-score">${data.correct}/${data.total}</div>
    </div>`;
  }).join('');

  // Review items
  const reviewHTML = QUESTIONS.map((q,i) => {
    const status = answers[i]===null?'skipped':answers[i]===q.a?'correct':'wrong';
    const optHTML = q.o.map((opt,oi) => {
      let cls = 'neutral';
      if (oi === q.a) cls = 'correct-ans';
      if (answers[i] === oi && oi !== q.a) cls = 'your-wrong';
      if (answers[i] === oi && oi === q.a) cls = 'your-correct';
      const prefix = oi===q.a?'✓ Correct: ':answers[i]===oi&&oi!==q.a?'✗ Your answer: ':'';
      return `<div class="ri-ans ${cls}">${prefix}${String.fromCharCode(65+oi)}. ${opt}</div>`;
    }).join('');
    return `<div class="review-item">
      <div class="ri-top">
        <span class="ri-num">Q${i+1} · ${q.s}</span>
        <span class="ri-badge ${status}">${status.toUpperCase()}</span>
      </div>
      <div class="ri-q">${q.q}</div>
      <div class="ri-answers">${optHTML}</div>
    </div>`;
  }).join('');

  const resultsEl = document.getElementById('results');
  resultsEl.innerHTML = `
    <div class="results-inner">
      <div class="results-title">Quiz <span>${pass?'Passed ✓':'Failed ✗'}</span></div>
      <div class="results-sub">GNURS 303 — Eye, Ear, Throat & Dental Nursing Assessment</div>
      <div class="score-hero">
        <div class="score-pct ${pass?'pass':'fail'}">${pct}%</div>
        <div class="score-grade">${pass?'PASS — Well done!':'FAIL — Keep studying'} &nbsp;·&nbsp; ${correct} of 120 correct</div>
        <div class="score-detail">
          <div class="sd-item"><div class="n correct">${correct}</div><div class="l">Correct</div></div>
          <div class="sd-item"><div class="n wrong">${wrong}</div><div class="l">Wrong</div></div>
          <div class="sd-item"><div class="n skipped">${skipped}</div><div class="l">Skipped</div></div>
        </div>
      </div>
      <div class="section-breakdown">
        <h3>Performance by Topic</h3>
        ${secHTML}
      </div>
      <div class="results-actions">
        <button class="btn-retry" onclick="location.reload()">Retake Quiz</button>
        <button class="btn-review-toggle" onclick="toggleReview()">📋 View Full Review</button>
      </div>
      <div id="reviewSection" style="display:none;margin-top:28px">
        <h3 style="font-size:13px;text-transform:uppercase;letter-spacing:2px;color:var(--gold);font-family:'IBM Plex Mono',monospace;margin-bottom:14px">Question Review</h3>
        <div class="review-list">${reviewHTML}</div>
      </div>
    </div>`;
  resultsEl.style.display = 'block';
  // animate section bars
  setTimeout(()=>{
    document.querySelectorAll('.sr-fill').forEach(el=>{
      const w = el.style.width;
      el.style.width='0';
      setTimeout(()=>el.style.width=w,100);
    });
  },200);
}

function toggleReview() {
  const section = document.getElementById('reviewSection');
  reviewVisible = !reviewVisible;
  section.style.display = reviewVisible ? 'block' : 'none';
  document.querySelector('.btn-review-toggle').textContent = reviewVisible ? '▲ Hide Review' : '📋 View Full Review';
}
</script>
</body>
</html>
