<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>AREXMADO 온라인 IDE (VS Code 스타일 · 단일 HTML)</title>
  <!-- Tailwind (CDN) -->
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    :root{--bg:#0b1020;--panel:#0f172a;--ink:#e5e7eb;--muted:#94a3b8;--border:#334155;--acc:#7aa2f7}
    html,body,#app{height:100%}
    body{margin:0;background:var(--bg);color:var(--ink);font-family:ui-sans-serif,system-ui,Segoe UI,Roboto,Apple SD Gothic Neo,Pretendard,Malgun Gothic,Arial}
    .btn{border:1px solid var(--border);border-radius:.75rem;padding:.4rem .8rem;background:#111827}
    .btn:hover{background:#1f2937}
    .tab{border-top-left-radius:.5rem;border-top-right-radius:.5rem}
    .file-item{border-radius:.5rem}
    .console{font-family:ui-monospace,SFMono-Regular,Menlo,Monaco,Consolas,monospace}
    .badge{display:inline-flex;align-items:center;justify-content:center;width:20px;height:20px;font-size:10px;font-weight:700;border-radius:4px;margin-right:.3rem}
  </style>
  <!-- Monaco Editor (CDN) -->
  <script>var requireMonaco={paths:{vs:"https://cdn.jsdelivr.net/npm/monaco-editor@0.45.0/min/vs"}};</script>
  <script src="https://cdn.jsdelivr.net/npm/monaco-editor@0.45.0/min/vs/loader.js"></script>
</head>
<body>
<div id="app" class="flex flex-col h-full">
  <!-- Header -->
  <div class="flex items-center gap-2 px-3 py-2 border-b" style="border-color:var(--border);background:rgba(0,0,0,.25);backdrop-filter:blur(6px)">
    <div class="font-bold text-lg">AREXMADO IDE</div>
    <div class="text-xs" style="color:var(--muted)">VS Code 스타일 · 단일 HTML</div>
    <div class="flex-1"></div>
    <button id="runBtn" class="btn">▶ 실행</button>
    <button id="addFileBtn" class="btn">＋ 새 파일</button>
    <label class="ml-3 text-sm">글꼴</label>
    <input id="fontSize" type="number" min="10" max="22" value="14" class="w-16 px-2 py-1 rounded-md bg-transparent" style="border:1px solid var(--border)">
  </div>

  <!-- Main -->
  <div class="flex flex-1 min-h-0">
    <!-- Sidebar -->
    <aside class="w-64 p-2 border-r overflow-auto" style="border-color:var(--border)">
      <div class="text-xs uppercase tracking-wide mb-2" style="color:var(--muted)">파일</div>
      <ul id="fileList" class="space-y-1"></ul>
    </aside>

    <!-- Editor + Tabs -->
    <main class="flex-1 min-w-0 flex flex-col">
      <div id="tabbar" class="flex items-center gap-2 px-2 py-1 border-b overflow-auto" style="border-color:var(--border)"></div>
      <div id="editor" style="height:100%"></div>
    </main>

    <!-- Preview / Console -->
    <section class="w-[34%] min-w-[320px] border-l flex flex-col" style="border-color:var(--border)">
      <div class="flex items-center gap-2 px-2 py-2 border-b" style="border-color:var(--border)">
        <span class="text-sm" style="color:var(--muted)">미리보기 / 콘솔</span>
        <div class="flex-1"></div>
        <button id="clearConsole" class="btn">콘솔 지우기</button>
      </div>
      <div class="h-1/2 min-h-[200px] border-b" style="border-color:var(--border)">
        <iframe id="preview" class="w-full h-full bg-white" sandbox="allow-scripts allow-same-origin"></iframe>
      </div>
      <div id="console" class="flex-1 overflow-auto p-2 text-sm console"></div>
    </section>
  </div>

  <footer class="px-3 py-2 text-xs" style="color:var(--muted);border-top:1px solid var(--border)">
    © <span id="year"></span> AREXMADO — 브라우저 로컬에 자동 저장됩니다.
  </footer>
</div>

<script>
(function(){
  const LS_KEY = 'arexmado-ide-project-v1-html';
  const FONT_KEY = 'arexmado-ide-fontsize-html';

  const DEFAULT_FILES = {
    'index.html': {name:'index.html', language:'html', content:`<!doctype html>\n<html lang="ko">\n<head>\n  <meta charset="utf-8">\n  <meta name="viewport" content="width=device-width, initial-scale=1">\n  <title>AREXMADO 온라인 IDE</title>\n  <link rel="stylesheet" href="style.css">\n</head>\n<body>\n  <div id="app">\n    <h1>AREXMADO 온라인 IDE 🚀</h1>\n    <p>왼쪽 파일을 편집하고 상단 ▶ 실행을 눌러보세요.</p>\n  </div>\n  <script src="main.js"><\/script>\n</body>\n</html>`},
    'style.css': {name:'style.css', language:'css', content:`:root{--bg:#0b1020;--fg:#e3e8f5;--acc:#7aa2f7} body{margin:0;background:var(--bg);color:var(--fg);font-family:ui-sans-serif,system-ui,Apple SD Gothic Neo,Pretendard,Malgun Gothic,Arial} h1{color:var(--acc)} #app{padding:24px}`},
    'main.js': {name:'main.js', language:'javascript', content:`console.log('Hello from AREXMADO IDE!');\nconst p=document.createElement('p');p.textContent='JS 로드 성공!';document.getElementById('app').appendChild(p);`},
    'hello.py': {name:'hello.py', language:'python', content:`print('파이썬도 됩니다!')\nprint('2+3=', 2+3)`},
  };

  const yearEl = document.getElementById('year');
  yearEl.textContent = new Date().getFullYear();

  // State
  let files = load(LS_KEY) || DEFAULT_FILES;
  let active = Object.keys(files)[0];
  let editor, monacoRef;

  // Elements
  const fileListEl = document.getElementById('fileList');
  const tabbarEl = document.getElementById('tabbar');
  const editorEl = document.getElementById('editor');
  const previewEl = document.getElementById('preview');
  const consoleEl = document.getElementById('console');
  const addFileBtn = document.getElementById('addFileBtn');
  const runBtn = document.getElementById('runBtn');
  const clearConsoleBtn = document.getElementById('clearConsole');
  const fontSizeInput = document.getElementById('fontSize');

  // Font size
  const savedFont = Number(localStorage.getItem(FONT_KEY)||14);
  fontSizeInput.value = savedFont;

  fontSizeInput.addEventListener('change', () => {
    const v = Number(fontSizeInput.value||14);
    localStorage.setItem(FONT_KEY, String(v));
    if (editor) editor.updateOptions({fontSize:v});
  });

  // Monaco init
  window.require = window.require || function(){};
  require.config(requireMonaco);
  require(['vs/editor/editor.main'], function(monaco){
    monacoRef = monaco;
    editor = monaco.editor.create(editorEl, {
      value: files[active].content,
      language: files[active].language,
      theme: 'vs-dark',
      fontSize: savedFont,
      minimap: {enabled:true},
      wordWrap: 'on',
      automaticLayout: true,
    });
    editor.getModel().onDidChangeContent(()=>{
      files[active].content = editor.getValue();
      save(LS_KEY, files);
    });
    render();
  });

  // Render UI
  function render(){
    // Sidebar file list
    fileListEl.innerHTML = '';
    Object.values(files).forEach(f=>{
      const li = document.createElement('li');
      li.className = 'file-item flex items-center justify-between px-2 py-1 hover:bg-slate-700/40' + (active===f.name?' bg-slate-700/60':'');
      const btn = document.createElement('button');
      btn.className = 'text-left truncate';
      btn.title = f.name; btn.textContent = f.name;
      btn.onclick = ()=>setActive(f.name);

      const actions = document.createElement('div');
      actions.className = 'opacity-0 hover:opacity-100 sm:opacity-100 flex items-center gap-1';
      const rn = document.createElement('button'); rn.className='text-xs'; rn.textContent='이름'; rn.onclick=()=>renameFile(f.name);
      const del = document.createElement('button'); del.className='text-xs text-red-400'; del.textContent='삭제'; del.onclick=()=>deleteFile(f.name);
      actions.append(rn,del);

      li.append(btn,actions);
      fileListEl.append(li);
    });

    // Tabs
    tabbarEl.innerHTML='';
    Object.values(files).forEach(f=>{
      const b=document.createElement('button');
      b.className='tab px-2 py-1 text-sm ' + (active===f.name? 'bg-slate-900 border border-slate-600' : 'opacity-70 hover:opacity-100');
      b.textContent = f.name; b.title=f.name; b.onclick=()=>setActive(f.name);
      tabbarEl.append(b);
    });

    // Editor model
    if (editor && monacoRef) {
      const model = monacoRef.editor.createModel(files[active].content, files[active].language);
      editor.setModel(model);
      model.onDidChangeContent(()=>{
        files[active].content = model.getValue();
        save(LS_KEY, files);
      });
    }
  }

  function setActive(name){
    if (!files[name]) return;
    active = name;
    render();
  }

  function renameFile(name){
    const next = prompt('새 파일 이름', name);
    if (!next || next===name) return;
    if (files[next]) return alert('이미 존재하는 파일명입니다.');
    files[next] = {...files[name], name:next, language: extToLang(next)};
    delete files[name];
    active = next;
    save(LS_KEY, files);
    render();
  }

  function deleteFile(name){
    if (!confirm(name+' 파일을 삭제할까요?')) return;
    delete files[name];
    const keys = Object.keys(files);
    active = keys[0]||'';
    save(LS_KEY, files);
    render();
  }

  addFileBtn.onclick = ()=>{
    const base = prompt('새 파일 이름(예: script.js, page.html, note.md, app.py)', 'new.js');
    if (!base) return;
    if (files[base]) return alert('이미 존재하는 파일입니다.');
    files[base] = {name:base, language:extToLang(base), content:''};
    active = base;
    save(LS_KEY, files);
    render();
  };

  // Run
  runBtn.onclick = async ()=>{
    clearConsole();
    const current = files[active];
    if (!current) return;

    if (current.language==='python'){
      await runPython(current.content);
      return;
    }

    const html = (files['index.html']&&files['index.html'].content)||'';
    const css  = (files['style.css']&&files['style.css'].content)||'';
    const js   = (files['main.js']&&files['main.js'].content)||'';

    const srcdoc = `<!doctype html>\n<html><head>\n<meta charset=\"utf-8\"/>\n<meta name=\"viewport\" content=\"width=device-width,initial-scale=1\"/>\n<style>${escapeHTML(css)}</style>\n</head>\n<body>\n${injectBody(html)}\n<script>(function(){\n  function post(type, text){ parent.postMessage({__arex_console:{type,text}}, '*'); }\n  const _log=console.log,_err=console.error,_warn=console.warn;\n  console.log=function(...a){post('log', a.map(String).join(' '));_log.apply(console,a)};\n  console.error=function(...a){post('error', a.map(String).join(' '));_err.apply(console,a)};\n  console.warn=function(...a){post('warn', a.map(String).join(' '));_warn.apply(console,a)};\n  window.addEventListener('error', e=>post('error', e.message));\n})();<\/script>\n<script>\n${js}\n<\/script>\n</body></html>`;

    previewEl.srcdoc = srcdoc;
  };

  // Console bridge
  window.addEventListener('message', (e)=>{
    if (e?.data?.__arex_console){
      pushLog(e.data.__arex_console.type, e.data.__arex_console.text);
    }
  });

  clearConsoleBtn.onclick = clearConsole;

  // Python (Pyodide)
  let pyReady=false, pyodide;
  async function ensurePyodide(){
    if (pyReady) return;
    pushLog('log','Pyodide 로드 중... (최초 1회)');
    if (!window.loadPyodide){
      await loadScript('https://cdn.jsdelivr.net/pyodide/v0.24.1/full/pyodide.js');
    }
    pyodide = await window.loadPyodide({
      stdout: (t)=>pushLog('log', t),
      stderr: (t)=>pushLog('error', t)
    });
    pyReady = true;
    pushLog('log','Pyodide 준비 완료');
  }

  async function runPython(code){
    try{
      await ensurePyodide();
      const out = await pyodide.runPythonAsync(code);
      pushLog('log', String(out ?? '(완료)'));
    }catch(err){
      pushLog('error', String(err));
    }
  }

  // Utils
  function escapeHTML(str){
    return String(str).replaceAll('&','&amp;').replaceAll('<','&lt;').replaceAll('>','&gt;').replaceAll('"','&quot;').replaceAll("'",'&#039;');
  }
  function injectBody(html){
    try{ const m = /<body[^>]*>([\s\s\S]*?)<\/body>/i.exec(html); if(m) return m[1]; }catch(e){}
    return html;
  }
  function loadScript(src){
    return new Promise((res,rej)=>{ const s=document.createElement('script'); s.src=src; s.onload=res; s.onerror=rej; document.head.appendChild(s); });
  }
  function pushLog(type,text){
    const line=document.createElement('div');
    line.className= type==='error' ? 'text-red-400' : (type==='warn' ? 'text-yellow-400' : 'text-slate-200');
    line.textContent=String(text);
    consoleEl.append(line);
    consoleEl.scrollTop = consoleEl.scrollHeight;
  }
  function clearConsole(){ consoleEl.innerHTML=''; }
  function save(k,v){ localStorage.setItem(k, JSON.stringify(v)); }
  function load(k){ try{ return JSON.parse(localStorage.getItem(k)||''); }catch(e){ return null; } }
  function extToLang(name){
    if (name.endsWith('.html')) return 'html';
    if (name.endsWith('.css')) return 'css';
    if (name.endsWith('.js')) return 'javascript';
    if (name.endsWith('.ts')) return 'typescript';
    if (name.endsWith('.json')) return 'json';
    if (name.endsWith('.md')) return 'markdown';
    if (name.endsWith('.py')) return 'python';
    return 'plaintext';
  }
})();
</script>
</body>
</html>
