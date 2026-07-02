<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>ER Transfer Report Generator</title>
<style>
  :root{
    --navy:#0f2540;
    --navy-2:#16334f;
    --ink:#1a2433;
    --ink-soft:#5c6b7a;
    --line:#e2e6ea;
    --bg:#f4f6f8;
    --card:#ffffff;
    --accent:#0f2540;
    --accent-soft:#eef2f6;
    --danger:#b3261e;
  }
  *{box-sizing:border-box;}
  body{
    margin:0;
    font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",Helvetica,Arial,sans-serif;
    background:var(--bg);
    color:var(--ink);
  }
  .app{display:flex;min-height:100vh;}

  /* Sidebar */
  .sidebar{
    width:220px;
    background:var(--navy);
    color:#cfe0f0;
    display:flex;
    flex-direction:column;
    flex-shrink:0;
  }
  .sidebar-top{
    display:flex;
    justify-content:space-between;
    align-items:center;
    padding:16px;
    border-bottom:1px solid rgba(255,255,255,0.12);
  }
  .sidebar-top button{
    background:none;border:none;color:#cfe0f0;font-size:13px;cursor:pointer;
    display:flex;align-items:center;gap:4px;
  }
  .sidebar-list{flex:1;overflow-y:auto;padding:12px;}
  .sidebar-empty{font-size:12px;color:#8fa6bc;line-height:1.5;padding:8px;}
  .history-item{
    background:rgba(255,255,255,0.06);
    border-radius:8px;
    padding:10px 12px;
    margin-bottom:8px;
    cursor:pointer;
    font-size:12.5px;
    position:relative;
  }
  .history-item:hover{background:rgba(255,255,255,0.12);}
  .history-item .h-name{font-weight:600;color:#fff;display:block;margin-bottom:2px;}
  .history-item .h-date{color:#8fa6bc;font-size:11px;}
  .history-item .h-del{
    position:absolute;top:8px;right:8px;background:none;border:none;
    color:#8fa6bc;cursor:pointer;font-size:14px;
  }
  .history-item .h-del:hover{color:#fff;}

  /* Main */
  .main{flex:1;display:flex;flex-direction:column;min-width:0;}
  .topbar{
    display:flex;justify-content:space-between;align-items:center;
    padding:14px 24px;background:#fff;border-bottom:1px solid var(--line);
  }
  .topbar h1{font-size:16px;margin:0;}
  .topbar .sub{font-size:12px;color:var(--ink-soft);margin-top:2px;}
  .topbar .count{font-size:12px;color:var(--ink-soft);}

  .columns{
    flex:1;display:flex;gap:0;overflow:hidden;
  }
  .col-left{
    width:46%;min-width:380px;overflow-y:auto;padding:24px;border-right:1px solid var(--line);
  }
  .col-right{
    flex:1;overflow-y:auto;padding:24px;background:var(--bg);
  }

  h2.page-title{font-size:20px;margin:0 0 4px;}
  p.page-desc{font-size:13px;color:var(--ink-soft);margin:0 0 20px;}

  .card{
    background:var(--card);
    border:1px solid var(--line);
    border-radius:10px;
    padding:16px;
    margin-bottom:16px;
  }
  .card-label{
    font-size:11px;font-weight:700;letter-spacing:0.06em;color:var(--ink-soft);
    text-transform:uppercase;margin-bottom:12px;
  }
  .field{margin-bottom:12px;}
  .field:last-child{margin-bottom:0;}
  .field label{display:block;font-size:12px;color:var(--ink-soft);margin-bottom:4px;}
  .field input, .field textarea, .field select{
    width:100%;
    border:1px solid var(--line);
    border-radius:6px;
    padding:8px 10px;
    font-size:13px;
    font-family:inherit;
    color:var(--ink);
  }
  .field-row{display:flex;gap:12px;}
  .field-row .field{flex:1;}

  .tabs{display:flex;border:1px solid var(--line);border-radius:8px;overflow:hidden;margin-bottom:14px;}
  .tab{
    flex:1;text-align:center;padding:10px;font-size:13px;font-weight:600;
    cursor:pointer;background:#fff;color:var(--ink-soft);border:none;
  }
  .tab.active{background:var(--accent-soft);color:var(--navy);}

  .narrative-wrap{position:relative;}
  .load-sample{
    position:absolute;top:-28px;right:0;font-size:12px;color:var(--navy);
    background:none;border:none;cursor:pointer;display:flex;gap:4px;align-items:center;
  }
  textarea#narrative{min-height:160px;resize:vertical;}

  .structured-grid{display:none;}
  .structured-grid.active{display:block;}
  .narrative-pane{display:block;}
  .narrative-pane.hidden{display:none;}

  .generate-row{display:flex;justify-content:flex-end;margin-top:16px;}
  button.primary{
    background:var(--navy);color:#fff;border:none;border-radius:8px;
    padding:11px 20px;font-size:13.5px;font-weight:600;cursor:pointer;
    display:flex;align-items:center;gap:8px;
  }
  button.primary:hover{background:var(--navy-2);}
  button.primary:disabled{opacity:0.6;cursor:not-allowed;}

  .error-box{
    background:#fdecea;border:1px solid #f5c2c0;color:var(--danger);
    padding:10px 12px;border-radius:8px;font-size:12.5px;margin-bottom:14px;
  }

  /* Preview */
  .preview-toolbar{
    display:flex;justify-content:flex-end;gap:8px;margin-bottom:14px;
  }
  .preview-toolbar button{
    background:#fff;border:1px solid var(--line);border-radius:7px;
    padding:7px 12px;font-size:12.5px;color:var(--ink);cursor:pointer;
    display:flex;align-items:center;gap:6px;
  }
  .preview-toolbar button:hover{background:var(--accent-soft);}
  .preview-toolbar button:disabled{opacity:0.5;cursor:not-allowed;}

  .doc-sheet{
    background:#fff;border:1px solid var(--line);border-radius:10px;
    padding:40px 48px;min-height:500px;
    font-family:Georgia,'Times New Roman',serif;
  }
  .doc-sheet h3{font-size:19px;margin:0 0 4px;letter-spacing:0.02em;}
  .doc-sheet .addr{font-size:13px;color:#333;line-height:1.5;margin-bottom:14px;}
  .doc-sheet hr{border:none;border-top:2px solid var(--navy);margin:0 0 20px;}
  .doc-line{font-size:13.5px;margin-bottom:10px;line-height:1.6;}
  .doc-line b{font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",Helvetica,Arial,sans-serif;}
  .doc-section{margin-top:18px;}
  .doc-section b{display:block;margin-bottom:4px;font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",Helvetica,Arial,sans-serif;font-size:13.5px;}
  .doc-section p{font-size:13.5px;line-height:1.65;margin:0;color:#222;}
  .placeholder{color:#a7b1ba;}

  @media print{
    .sidebar, .col-left, .topbar, .preview-toolbar{display:none !important;}
    .columns{display:block;}
    .col-right{padding:0;background:#fff;}
    .doc-sheet{border:none;padding:0;}
  }
</style>
</head>
<body>
<div class="app">

  <div class="sidebar">
    <div class="sidebar-top">
      <button onclick="startNew()">＋ New</button>
      <span style="font-size:12px;color:#8fa6bc;">History</span>
    </div>
    <div class="sidebar-list" id="historyList">
      <div class="sidebar-empty">No saved reports yet. Generate one and it will appear here.</div>
    </div>
  </div>

  <div class="main">
    <div class="topbar">
      <div>
        <h1>ER Transfer Report Generator</h1>
        <div class="sub">Reportable Incident · Clinical documentation assistant</div>
      </div>
      <div class="count" id="savedCount">0 saved reports</div>
    </div>

    <div class="columns">
      <div class="col-left">
        <h2 class="page-title">ER Transfer Report Generator</h2>
        <p class="page-desc">Paste patient info or fill the form — a formatted incident report is generated in the style of your template.</p>

        <div class="card">
          <div class="card-label">Facility Header</div>
          <div class="field">
            <label>Facility Name</label>
            <input id="facName" value="SAFE HARBOR RECOVERY CENTER">
          </div>
          <div class="field-row">
            <div class="field">
              <label>Address Line 1</label>
              <input id="facAddr1" value="2700 London Boulevard">
            </div>
            <div class="field">
              <label>Address Line 2</label>
              <input id="facAddr2" value="Portsmouth, VA 23707">
            </div>
          </div>
        </div>

        <div class="tabs">
          <button class="tab active" id="tabNarrativeBtn" onclick="switchTab('narrative')">Paste Narrative</button>
          <button class="tab" id="tabStructuredBtn" onclick="switchTab('structured')">Structured Form</button>
        </div>

        <div id="narrativePane" class="narrative-pane">
          <div class="narrative-wrap">
            <button class="load-sample" onclick="loadSample()">⧉ Load sample</button>
            <div class="field">
              <label>Patient info / free-form notes</label>
              <textarea id="narrative" placeholder="Paste the patient's story here — symptoms, timeline, ED findings, treatments, UDS/BAC, notifications made, etc."></textarea>
            </div>
          </div>
        </div>

        <div id="structuredPane" class="structured-grid">
          <div class="card">
            <div class="field-row">
              <div class="field"><label>Client Name</label><input id="sWho" placeholder="Jane Doe"></div>
              <div class="field"><label>MRN</label><input id="sMrn" placeholder="000000-00"></div>
            </div>
            <div class="field-row">
              <div class="field"><label>Location</label><input id="sWhere" placeholder="Safe Harbor Recovery Center – London Location"></div>
              <div class="field"><label>Date/Time Left Facility</label><input id="sWhen" placeholder="09:33 PM 06/28/2026"></div>
            </div>
            <div class="field"><label>Reason for transfer</label><input id="sReason" placeholder="e.g. uncontrolled vomiting, abdominal pain"></div>
            <div class="field"><label>Transport method</label><input id="sTransport" placeholder="e.g. ambulance"></div>
            <div class="field"><label>Notifications / documentation made prior to transport</label><input id="sPreNotify" placeholder="e.g. med list & face sheet given to EMS, SafetyZone report completed"></div>
            <div class="field"><label>ED findings / treatment / diagnosis</label><textarea id="sEd" placeholder="Diagnosis, meds administered, etc." style="min-height:70px;"></textarea></div>
            <div class="field"><label>UDS / BAC results</label><input id="sTox" placeholder="e.g. UDS positive for fentanyl; BAC negative"></div>
            <div class="field"><label>Return time (leave blank if not yet returned)</label><input id="sReturn" placeholder="e.g. 2:00 AM"></div>
            <div class="field"><label>Post-return procedures</label><input id="sPostReturn" placeholder="e.g. wanded, vitals obtained, ED paperwork filed in HCS"></div>
          </div>
        </div>

        <div id="errorBox"></div>

        <div class="generate-row">
          <button class="primary" id="genBtn" onclick="generateReport()">✨ Generate Report</button>
        </div>
      </div>

      <div class="col-right">
        <div class="preview-toolbar">
          <button onclick="copyReport()" id="copyBtn" disabled>Copy</button>
          <button onclick="downloadDoc()" id="docBtn" disabled>.doc</button>
          <button onclick="window.print()" id="pdfBtn" disabled>PDF</button>
          <button onclick="window.print()" id="printBtn" disabled>Print</button>
        </div>
        <div class="doc-sheet" id="docSheet">
          <h3 id="pFacName">SAFE HARBOR RECOVERY CENTER</h3>
          <div class="addr"><span id="pAddr1">2700 London Boulevard</span><br><span id="pAddr2">Portsmouth, VA 23707</span></div>
          <hr>
          <div class="doc-line"><b>REPORTABLE INCIDENT:</b> ER Transfer/Returned</div>
          <div class="doc-line"><b>WHO:</b> <span id="pWho" class="placeholder">—</span> ; <b>MRN:</b> <span id="pMrn" class="placeholder">—</span></div>
          <div class="doc-line"><b>WHERE:</b> <span id="pWhere" class="placeholder">—</span></div>
          <div class="doc-line"><b>WHEN:</b> <span id="pWhen" class="placeholder">—</span></div>
          <div class="doc-section">
            <b>DESCRIPTION:</b>
            <p id="pDescription" class="placeholder">Paste patient info on the left and click Generate to populate this section.</p>
          </div>
          <div class="doc-section">
            <b>OUTCOME:</b>
            <p id="pOutcome" class="placeholder">—</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>

<script>
let currentReportId = null;
let lastGenerated = null;

function switchTab(tab){
  document.getElementById('narrativePane').classList.toggle('hidden', tab!=='narrative');
  document.getElementById('structuredPane').classList.toggle('active', tab==='structured');
  document.getElementById('tabNarrativeBtn').classList.toggle('active', tab==='narrative');
  document.getElementById('tabStructuredBtn').classList.toggle('active', tab==='structured');
}

function loadSample(){
  document.getElementById('narrative').value =
`Client Jane Doe (MRN 000000-00) was transferred from the London Location to the Emergency Department via ambulance at 09:33 PM on 06/28/2026 following complaints of uncontrolled vomiting and abdominal pain, per provider order. Medication list and face sheet were printed and given to EMS. All required notifications were made and a SafetyZone report was completed.

Client returned to the facility at approximately 2:00 AM. Discharge paperwork indicated a diagnosis of gastritis. Hospital records confirmed fentanyl was administered during treatment, consistent with a positive fentanyl result on the client's urine drug screen. No other substances were detected. BAC was negative on return.

Upon return, client was wanded, vitals were obtained, and ED discharge paperwork was scanned into HCS. No additional concerns were identified.`;
  switchTab('narrative');
}

function startNew(){
  currentReportId = null;
  document.getElementById('narrative').value = '';
  ['sWho','sMrn','sWhere','sWhen','sReason','sTransport','sPreNotify','sEd','sTox','sReturn','sPostReturn'].forEach(id=>{
    document.getElementById(id).value='';
  });
  renderPreview(null);
  setToolbarEnabled(false);
  document.getElementById('errorBox').innerHTML='';
}

function setToolbarEnabled(on){
  ['copyBtn','docBtn','pdfBtn','printBtn'].forEach(id=>{
    document.getElementById(id).disabled = !on;
  });
}

function gatherInput(){
  const structuredActive = document.getElementById('tabStructuredBtn').classList.contains('active');
  if(structuredActive){
    const vals = {
      who: document.getElementById('sWho').value.trim(),
      mrn: document.getElementById('sMrn').value.trim(),
      where: document.getElementById('sWhere').value.trim(),
      when: document.getElementById('sWhen').value.trim(),
      reason: document.getElementById('sReason').value.trim(),
      transport: document.getElementById('sTransport').value.trim(),
      preNotify: document.getElementById('sPreNotify').value.trim(),
      ed: document.getElementById('sEd').value.trim(),
      tox: document.getElementById('sTox').value.trim(),
      returnTime: document.getElementById('sReturn').value.trim(),
      postReturn: document.getElementById('sPostReturn').value.trim(),
    };
    const hasAny = Object.values(vals).some(v=>v.length>0);
    if(!hasAny) return null;
    return 'STRUCTURED FIELDS:\n' + Object.entries(vals).map(([k,v])=>`${k}: ${v||'(not provided)'}`).join('\n');
  } else {
    const text = document.getElementById('narrative').value.trim();
    if(!text) return null;
    return 'FREE-FORM NOTES:\n' + text;
  }
}

async function generateReport(){
  const errorBox = document.getElementById('errorBox');
  errorBox.innerHTML='';
  const input = gatherInput();
  if(!input){
    errorBox.innerHTML = '<div class="error-box">Enter patient info (narrative or structured fields) before generating.</div>';
    return;
  }
  const facName = document.getElementById('facName').value.trim() || 'SAFE HARBOR RECOVERY CENTER';
  const facAddr1 = document.getElementById('facAddr1').value.trim();
  const facAddr2 = document.getElementById('facAddr2').value.trim();

  const genBtn = document.getElementById('genBtn');
  genBtn.disabled = true;
  genBtn.textContent = 'Generating…';

  const systemPrompt = `You are a clinical documentation assistant for a Virginia-licensed behavioral health and substance use disorder treatment facility. You convert raw patient transfer notes into a formatted "Reportable Incident: ER Transfer/Returned" report.

Match this exact tone and structure (this is a real example of the target style, for style reference only):

"The client, [Name], was transferred to the Emergency Department via ambulance following complaints of [symptoms], per provider order. Prior to transport, the client's medication list and face sheet were printed and provided to EMS personnel. All required notifications were initiated, and a Safety Zone report was completed.

The client returned to the facility at approximately [time] following evaluation and treatment in the Emergency Department. Discharge paperwork indicated a diagnosis of [diagnosis]. Hospital documentation confirmed administration of [medication] during treatment, which corresponded with the positive [substance] result on the client's urine drug screen. No other substances were detected on the urine drug screen, and the client's blood alcohol content (BAC) was negative upon return.

Upon arrival at the facility, the client was wanded, vital signs were obtained, and Emergency Department discharge paperwork was scanned into and filed within HCS.

All appropriate notifications and documentation were completed."

Rules:
- Write in third person, past tense, professional clinical documentation style.
- Use only information given. Never invent symptoms, diagnoses, times, medications, or results that are not present in the input.
- If the client has not yet returned to the facility, adjust tense/scope accordingly (do not describe a return that hasn't happened) and note the transfer is pending in the outcome.
- If a detail is missing, omit it gracefully rather than fabricating or writing a placeholder like "[unknown]".
- description should be 2-4 short paragraphs covering: reason/circumstances of transfer, transport and pre-transport notifications, ED findings/treatment/diagnosis, tox screen results if given, return and post-return procedures.
- outcome should be 1-2 sentences summarizing final status.
- Return ONLY a valid JSON object with exactly these keys: who, mrn, where, when, description, outcome. No markdown fences, no commentary, no extra keys.`;

  const userPrompt = `Facility: ${facName}, ${facAddr1}, ${facAddr2}\n\n${input}`;

  try{
    const response = await fetch("/api/generate", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ systemPrompt, userPrompt })
    });
    if(!response.ok){
      const errBody = await response.text();
      throw new Error('Server error: ' + errBody);
    }
    const data = await response.json();
    const textBlock = (data.content || []).find(b=>b.type==='text');
    if(!textBlock) throw new Error('No response from model.');
    let clean = textBlock.text.replace(/```json|```/g,'').trim();
    const parsed = JSON.parse(clean);

    lastGenerated = {
      facName, facAddr1, facAddr2,
      who: parsed.who || '',
      mrn: parsed.mrn || '',
      where: parsed.where || '',
      when: parsed.when || '',
      description: parsed.description || '',
      outcome: parsed.outcome || '',
      savedAt: new Date().toISOString()
    };
    renderPreview(lastGenerated);
    setToolbarEnabled(true);
    await saveReport(lastGenerated);
  } catch(err){
    console.error(err);
    errorBox.innerHTML = '<div class="error-box">Could not generate the report. Check your input and try again.</div>';
  } finally {
    genBtn.disabled = false;
    genBtn.textContent = '✨ Generate Report';
  }
}

function renderPreview(r){
  const set = (id, val, isPlaceholder)=>{
    const el = document.getElementById(id);
    el.textContent = val;
    el.classList.toggle('placeholder', !!isPlaceholder);
  };
  document.getElementById('pFacName').textContent = document.getElementById('facName').value || 'SAFE HARBOR RECOVERY CENTER';
  document.getElementById('pAddr1').textContent = document.getElementById('facAddr1').value;
  document.getElementById('pAddr2').textContent = document.getElementById('facAddr2').value;

  if(!r){
    set('pWho','—',true); set('pMrn','—',true); set('pWhere','—',true); set('pWhen','—',true);
    set('pDescription','Paste patient info on the left and click Generate to populate this section.',true);
    set('pOutcome','—',true);
    return;
  }
  set('pWho', r.who || '—', !r.who);
  set('pMrn', r.mrn || '—', !r.mrn);
  set('pWhere', r.where || '—', !r.where);
  set('pWhen', r.when || '—', !r.when);
  set('pDescription', r.description || '—', !r.description);
  set('pOutcome', r.outcome || '—', !r.outcome);
}

function copyReport(){
  if(!lastGenerated) return;
  const r = lastGenerated;
  const text = `${r.facName}\n${r.facAddr1}\n${r.facAddr2}\n\nREPORTABLE INCIDENT: ER Transfer/Returned\n\nWHO: ${r.who} ; MRN: ${r.mrn}\nWHERE: ${r.where}\nWHEN: ${r.when}\n\nDESCRIPTION:\n${r.description}\n\nOUTCOME:\n${r.outcome}`;
  navigator.clipboard.writeText(text).then(()=>{
    const btn = document.getElementById('copyBtn');
    const old = btn.textContent;
    btn.textContent = 'Copied';
    setTimeout(()=>btn.textContent = old, 1500);
  });
}

function downloadDoc(){
  const sheet = document.getElementById('docSheet').innerHTML;
  const html = `<html xmlns:o='urn:schemas-microsoft-com:office:office' xmlns:w='urn:schemas-microsoft-com:office:word' xmlns='http://www.w3.org/TR/REC-html40'>
  <head><meta charset='utf-8'><title>ER Transfer Report</title></head>
  <body style="font-family:Georgia,'Times New Roman',serif;">${sheet}</body></html>`;
  const blob = new Blob(['\ufeff', html], { type: 'application/msword' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  const name = (lastGenerated && lastGenerated.who) ? lastGenerated.who.replace(/\s+/g,'_') : 'ER_Transfer_Report';
  a.href = url;
  a.download = `${name}_ER_Transfer_Report.doc`;
  document.body.appendChild(a);
  a.click();
  a.remove();
  URL.revokeObjectURL(url);
}

// ---- Storage / History ----
async function saveReport(r){
  try{
    const id = 'report_' + Date.now();
    currentReportId = id;
    const res = await window.storage.set('reports:' + id, JSON.stringify(r), false);
    if(!res) console.error('Save failed');
    await refreshHistory();
  } catch(e){
    console.error('Storage error saving report:', e);
  }
}

async function refreshHistory(){
  const listEl = document.getElementById('historyList');
  const countEl = document.getElementById('savedCount');
  try{
    const result = await window.storage.list('reports:', false);
    const keys = (result && result.keys) ? result.keys : [];
    countEl.textContent = `${keys.length} saved report${keys.length===1?'':'s'}`;
    if(keys.length===0){
      listEl.innerHTML = '<div class="sidebar-empty">No saved reports yet. Generate one and it will appear here.</div>';
      return;
    }
    const items = [];
    for(const key of keys){
      try{
        const got = await window.storage.get(key, false);
        if(got && got.value){
          const r = JSON.parse(got.value);
          items.push({key, r});
        }
      } catch(e){ /* skip unreadable key */ }
    }
    items.sort((a,b)=> new Date(b.r.savedAt) - new Date(a.r.savedAt));
    listEl.innerHTML = items.map(({key,r})=>{
      const dateStr = new Date(r.savedAt).toLocaleString();
      const name = r.who || 'Unnamed report';
      return `<div class="history-item" onclick="loadFromHistory('${key}')">
        <span class="h-name">${escapeHtml(name)}</span>
        <span class="h-date">${dateStr}</span>
        <button class="h-del" onclick="event.stopPropagation(); deleteReport('${key}')">✕</button>
      </div>`;
    }).join('');
  } catch(e){
    console.error('Storage error loading history:', e);
    listEl.innerHTML = '<div class="sidebar-empty">Could not load history.</div>';
  }
}

async function loadFromHistory(key){
  try{
    const got = await window.storage.get(key, false);
    if(!got || !got.value) return;
    const r = JSON.parse(got.value);
    lastGenerated = r;
    currentReportId = key;
    document.getElementById('facName').value = r.facName || 'SAFE HARBOR RECOVERY CENTER';
    document.getElementById('facAddr1').value = r.facAddr1 || '';
    document.getElementById('facAddr2').value = r.facAddr2 || '';
    renderPreview(r);
    setToolbarEnabled(true);
  } catch(e){
    console.error('Could not load report:', e);
  }
}

async function deleteReport(key){
  try{
    await window.storage.delete(key, false);
    if(currentReportId===key){ startNew(); }
    await refreshHistory();
  } catch(e){
    console.error('Could not delete report:', e);
  }
}

function escapeHtml(str){
  return str.replace(/[&<>"']/g, m=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[m]));
}

refreshHistory();
</script>
</body>
</html>
