<!doctype html>
<html lang="id">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Checklist Sains Islam dan Lingkungan</title>
<style>
  body{font-family:system-ui,-apple-system,Segoe UI,Roboto; background:#f4f7fb; color:#0f172a; padding:24px; display:flex; flex-direction:column; align-items:center}
  .wrap{max-width:820px;width:100%}
  h1{margin:0 0 8px;font-size:20px;color:#0b4a6f}
  p.lead{margin:0 0 18px;color:#526475}
  .card{background:#fff;padding:14px;border-radius:12px;box-shadow:0 6px 18px rgba(3,10,30,0.06)}
  ul{list-style:none;padding:0;margin:0}
  li{display:flex;align-items:center;padding:10px;border-radius:8px;margin-bottom:8px;border:1px solid rgba(10,20,30,0.04)}
  li label{flex:1;cursor:pointer}
  input[type="checkbox"]{width:18px;height:18px;margin-right:12px;accent-color:#0b4a6f}
  .actions{display:flex;gap:8px;margin-top:12px;flex-wrap:wrap}
  button{padding:8px 12px;border-radius:8px;border:none;cursor:pointer;background:#0b4a6f;color:#fff}
  button.ghost{background:transparent;color:#0b4a6f;border:1px solid rgba(11,74,111,0.12)}
  .muted{color:#6b7b86;font-size:13px;margin-top:10px}
  input[type=file]{display:none}
  .top-row{display:flex;justify-content:space-between;align-items:center;margin-bottom:10px}
  @media(max-width:600px){li{padding:8px;font-size:14px}}
</style>
</head>
<body>
  <div class="wrap">
    <div class="top-row">
      <div>
        <h1>Checklist — Sains Islam dan Lingkungan</h1>
        <p class="lead">Progress disimpan otomatis di browser masing-masing. Bisa ekspor/import untuk pindah perangkat.</p>
      </div>
    </div>

    <div class="card" id="card">
      <ul id="list"></ul>

      <div class="actions">
        <button id="saveBtn">💾 Simpan Manual</button>
        <button id="exportBtn" class="ghost">⬇️ Ekspor (.json)</button>
        <button id="importBtn" class="ghost">⬆️ Impor (.json)</button>
        <button id="clearBtn" class="ghost">🗑️ Kosongkan</button>
        <input type="file" id="fileIn" accept=".json" />
      </div>

      <div class="muted">Local save key: <code id="keyLabel"></code></div>
    </div>
  </div>

<script>
/* DATA TOPIK */
const TOPICS = [
 'Asal Usul Sains Dalam Islam',
 'Integrasi Islam Dan Sains I',
 'Integrasi Islam Dan Sains II',
 'Alam Semesta',
 'Penciptaan Manusia',
 'Evolusi Dalam Perspektif Islam Dan Sains',
 'Hubungan Tuhan, Manusia, Dan Alam Dalam Islam',
 'Hidrologi Dalam Perspektif Islam Dan Sains',
 'Penciptaan Tumbuhan Dalam Perspektif Islam Dan Sains',
 'Penciptaan Gunung Menurut Islam Dan Sains',
 'Kelestarian Lingkungan Dalam Perspektif Islam Dan Sains',
 'Ilmu Dan Teknologi Modern Dalam Islam Dan Sains',
 'Hancurnya Alam Semesta Menurut Islam Dan Sains'
];

/* KEY untuk localStorage (mudah diganti) */
const STORAGE_KEY = 'pai_sains_s1_v1';
document.getElementById('keyLabel').textContent = STORAGE_KEY;

/* Build list */
const listEl = document.getElementById('list');
TOPICS.forEach((t, i) => {
  const li = document.createElement('li');
  const cb = document.createElement('input');
  cb.type = 'checkbox';
  cb.id = 't_' + i;
  const label = document.createElement('label');
  label.htmlFor = cb.id;
  label.textContent = (i+1) + '. ' + t;
  li.appendChild(cb);
  li.appendChild(label);
  listEl.appendChild(li);
});

/* Load saved state */
function load(){
  try{
    const raw = localStorage.getItem(STORAGE_KEY);
    if(!raw) return;
    const arr = JSON.parse(raw);
    arr.forEach((val, idx) => {
      const el = document.getElementById('t_' + idx);
      if(el) el.checked = !!val;
    });
  }catch(e){ console.warn('load err', e); }
}
load();

/* Auto-save on change */
listEl.addEventListener('change', () => {
  save();
});

/* Save function */
function save(){
  const data = Array.from(document.querySelectorAll('#list input[type="checkbox"]')).map(cb => cb.checked);
  try{ localStorage.setItem(STORAGE_KEY, JSON.stringify(data)); }catch(e){ console.warn('save err', e); }
}

/* Buttons */
document.getElementById('saveBtn').addEventListener('click', () => {
  save();
  alert('Progress disimpan di browser ini.');
});

/* Export JSON */
document.getElementById('exportBtn').addEventListener('click', () => {
  const data = {
    meta: { subject: 'Sains Islam dan Lingkungan', key: STORAGE_KEY, exportedAt: new Date().toISOString() },
    items: Array.from(document.querySelectorAll('#list input[type="checkbox"]')).map((cb, i) => ({
      index: i+1, topic: document.querySelector('#list label[for="'+cb.id+'"]').textContent, checked: !!cb.checked
    }))
  };
  const blob = new Blob([JSON.stringify(data, null, 2)], {type:'application/json'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = 'sains_checklist_export.json';
  document.body.appendChild(a);
  a.click();
  a.remove();
  URL.revokeObjectURL(url);
});

/* Import JSON */
const fileIn = document.getElementById('fileIn');
document.getElementById('importBtn').addEventListener('click', () => fileIn.click());
fileIn.addEventListener('change', (e) => {
  const f = e.target.files[0];
  if(!f) return;
  const reader = new FileReader();
  reader.onload = ev => {
    try{
      const parsed = JSON.parse(ev.target.result);
      if(parsed && parsed.items && Array.isArray(parsed.items)){
        parsed.items.forEach((it, idx) => {
          const el = document.getElementById('t_' + idx);
          if(el) el.checked = !!it.checked;
        });
        save();
        alert('Progress diimpor dan disimpan di browser ini.');
      }else alert('File tidak valid.');
    }catch(err){ alert('Gagal membaca file. Pastikan format JSON sesuai.'); }
  };
  reader.readAsText(f);
});

/* Clear */
document.getElementById('clearBtn').addEventListener('click', () => {
  if(!confirm('Kosongkan semua progress di browser ini?')) return;
  document.querySelectorAll('#list input[type="checkbox"]').forEach(cb => cb.checked = false);
  localStorage.removeItem(STORAGE_KEY);
  alert('Dikosongkan.');
});
</script>
</body>
</html>
