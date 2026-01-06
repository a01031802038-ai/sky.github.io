<html lang="ko">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>쉐키나 찬양팀 곡 검색</title>
  <style>
    :root {
      --bd:#e5e7eb; --fg:#111827; --muted:#6b7280; --bg:#ffffff;
      --chip:#f3f4f6; --warn:#991b1b; --ok:#065f46;
      --shadow: 0 10px 30px rgba(17,24,39,.08);
    }
    * { box-sizing: border-box; }
    body {
      margin:0; font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Noto Sans KR", Arial, sans-serif;
      background: var(--bg); color: var(--fg);
    }
    .wrap { max-width: 1080px; margin: 0 auto; padding: 22px; }
    h1 { font-size: 20px; margin: 0 0 6px; letter-spacing: -.2px; }
    .sub { color: var(--muted); font-size: 13px; margin-bottom: 14px; line-height: 1.55; }
    .card {
      border:1px solid var(--bd); border-radius: 16px; padding: 14px; margin: 12px 0;
      box-shadow: var(--shadow);
      background: #fff;
    }
    label { font-size: 12px; color: var(--muted); display:block; margin-bottom: 6px; }
    .searchWrap { position: relative; }
    input[type="text"] {
      width:100%; padding: 12px 44px 12px 12px; border:1px solid var(--bd); border-radius: 12px;
      font-size: 15px; outline: none; transition: border-color .15s ease, box-shadow .15s ease;
    }
    input[type="text"]:focus { border-color:#cbd5e1; box-shadow: 0 0 0 4px rgba(148,163,184,.25); }
    .clearBtn {
      position:absolute; right:8px; top:50%; transform: translateY(-50%);
      border:1px solid var(--bd); background:#fff; border-radius: 10px;
      padding: 6px 10px; font-size: 12px; cursor:pointer;
    }
    .clearBtn:hover { background:#fafafa; }
    .hint { font-size: 12px; color: var(--muted); margin-top: 8px; line-height: 1.5; }
    .status { font-size: 12px; white-space:pre-wrap; margin-top: 10px; display:flex; align-items:center; gap:8px; }
    .status.ok { color: var(--ok); }
    .status.bad { color: var(--warn); }
    .spin {
      width: 14px; height: 14px; border-radius: 999px;
      border: 2px solid rgba(17,24,39,.2);
      border-top-color: rgba(17,24,39,.65);
      animation: spin 0.8s linear infinite;
      display:none;
    }
    .status.loading .spin { display:inline-block; }
    @keyframes spin { to { transform: rotate(360deg); } }

    .grid { display:grid; grid-template-columns: 1fr 1fr; gap: 12px; }
    @media (max-width: 900px) { .grid { grid-template-columns: 1fr; } }

    .list {
      border:1px solid var(--bd); border-radius: 14px; overflow:hidden; background:#fff;
      box-shadow: var(--shadow);
    }
    .list .head {
      padding:10px 12px; background:#fafafa; border-bottom:1px solid var(--bd);
      font-weight: 700; font-size: 13px; display:flex; align-items:center; justify-content:space-between; gap:10px;
    }
    .list .body { max-height: 420px; overflow:auto; scroll-behavior:smooth; }
    .item {
      padding:10px 12px; border-bottom:1px solid var(--bd); font-size: 14px;
      display:flex; justify-content:space-between; gap:10px; cursor:pointer;
      transition: background .12s ease;
    }
    .item:hover { background:#fafafa; }
    .item:last-child { border-bottom: 0; }
    .chip {
      font-size: 12px; background: var(--chip); padding: 2px 8px; border-radius: 999px;
      color: #374151; white-space:nowrap;
    }

    .empty { color: var(--muted); font-size: 13px; padding: 12px; }
    table { width:100%; border-collapse: collapse; }
    th, td { border-bottom:1px solid var(--bd); padding: 10px 10px; text-align:left; font-size: 14px; }
    th { font-size: 13px; color:#374151; background:#fafafa; position: sticky; top:0; z-index: 1; }

    /* 자동완성 */
    .ac {
      position:absolute; left:0; right:0; top: calc(100% + 8px);
      border:1px solid var(--bd); border-radius: 14px; overflow:hidden; background:#fff;
      box-shadow: var(--shadow);
      display:none;
      z-index: 10;
    }
    .acHead {
      padding:8px 10px; background:#fafafa; border-bottom:1px solid var(--bd);
      display:flex; justify-content:space-between; align-items:center; gap:10px;
      font-size: 12px; color:#374151;
    }
    .acBody { max-height: 260px; overflow:auto; }
    .acRow {
      padding:10px 12px; border-bottom:1px solid var(--bd);
      display:flex; justify-content:space-between; gap:10px;
      cursor:pointer; font-size: 14px;
    }
    .acRow:last-child { border-bottom:0; }
    .acRow:hover, .acRow.active { background:#f9fafb; }
    mark {
      background: rgba(253, 230, 138, .55);
      padding: 0 2px;
      border-radius: 4px;
    }

    details summary { cursor:pointer; color:#374151; font-weight:700; }
    details .hint { margin-top:10px; }
    code { background:#f3f4f6; padding: 1px 6px; border-radius: 6px; }
  </style>
</head>
<body>
  <div class="wrap">
    <h1>쉐키나 찬양팀 곡 검색</h1>
    <div class="sub">
      곡명 검색하면 해당 곡 세션(파트/담당자)이 전부 나옵니다. <b>띄어쓰기 무시</b> / <b>중복 제거</b><br/>
      추천 곡명은 항상 보이고, 오른쪽 결과표는 <b>검색어 입력했을 때만</b> 뜹니다.
    </div>

    <div class="card">
      <label>곡명 검색</label>

      <div class="searchWrap">
        <input id="q" type="text" placeholder="곡명 검색 (예: 꽃들 / 주님 사랑 등)" autocomplete="off" />
        <button id="clear" class="clearBtn" type="button" title="지우기">지우기</button>

        <div id="ac" class="ac" aria-hidden="true">
          <div class="acHead">
            <span>검색 후보</span>
            <span id="acMeta" class="chip">0</span>
          </div>
          <div id="acBody" class="acBody"></div>
        </div>
      </div>

      <div class="hint">
        • 타이핑하면 후보가 뜹니다 (↑↓, Enter 가능) • 추천 곡명 클릭해도 됨
      </div>

      <div id="status" class="status">
        <span class="spin" aria-hidden="true"></span>
        <span id="statusText"></span>
      </div>

      <details style="margin-top:10px;">
        <summary>문제 생길 때 확인</summary>
        <div class="hint">
          상태창에 오류가 뜨면 아래 정보(시도 URL/감지된 헤더)를 복사해서 보내면 바로 잡을 수 있어요.
          <div id="debug" style="margin-top:8px;"></div>
        </div>
      </details>
    </div>

    <div class="grid">
      <div class="list">
        <div class="head"><span>전체 곡 목록</span><span class="chip" id="songCount">0</span></div>
        <div class="body" id="songs"></div>
      </div>

      <div class="list">
        <div class="head">검색 결과 (파트 / 담당자)</div>
        <div class="body">
          <div id="resultsEmpty" class="empty">검색어를 입력하면 결과가 나와요.</div>
          <table id="resultsTable" style="display:none;">
            <thead>
              <tr><th style="width:40%;">파트</th><th>담당자</th></tr>
            </thead>
            <tbody id="results"></tbody>
          </table>
        </div>
      </div>
    </div>

    <div class="card">
      <div style="font-weight:700; margin-bottom:8px;">DB 시트 형식</div>
      <div class="hint">
        DB 탭 1행 헤더는 가능하면 이렇게: <code>곡명</code> / <code>파트(악기)</code> / <code>담당자</code><br/>
        (v5는 헤더가 꼬여도 1행을 헤더로 다시 잡아서 최대한 복구합니다)
      </div>
    </div>
  </div>

<script>
  // 팀 게시용 고정 시트
  const FIXED_SHEET_URL = 'https://docs.google.com/spreadsheets/d/1fyGPILf7sxYSBLx1gq7hFeZwV0enO0F3ufh-9DmbJp0/edit?gid=1851883768#gid=1851883768';
  const FIXED_GID = '1851883768';
  const FIXED_SHEET_NAME = 'DB';

  // 텍스트 정규화: 공백/제로폭/BOM 제거 + 소문자
  function norm(s) {
    return (s ?? "").toString()
      .replace(/[\s\u200B\u200C\u200D\uFEFF]+/g, "")
      .toLowerCase();
  }
  const trim = (s) => (s ?? "").toString().trim();

  const elQ = document.getElementById("q");
  const elClear = document.getElementById("clear");
  const elSongs = document.getElementById("songs");
  const elSongCount = document.getElementById("songCount");
  const elResultsEmpty = document.getElementById("resultsEmpty");
  const elResultsTable = document.getElementById("resultsTable");
  const elResults = document.getElementById("results");
  const elStatus = document.getElementById("status");
  const elStatusText = document.getElementById("statusText");
  const elDebug = document.getElementById("debug");

  const elAC = document.getElementById("ac");
  const elACBody = document.getElementById("acBody");
  const elACMeta = document.getElementById("acMeta");

  function setStatus(msg, ok=true, loading=false) {
    elStatusText.textContent = msg;
    elStatus.className = "status " + (loading ? "loading" : "") + " " + (ok ? "ok" : "bad");
  }

  function uniqueBy(arr, keyFn) {
    const seen = new Set();
    const out = [];
    for (const x of arr) {
      const k = keyFn(x);
      if (seen.has(k)) continue;
      seen.add(k);
      out.push(x);
    }
    return out;
  }

  let DB = [];
  let SONGS = [];

  function rebuildSongs() {
    SONGS = uniqueBy(DB, r => r.songKey).map(r => r.song).sort((a,b) => a.localeCompare(b, "ko"));
  }

  function escapeHTML(s) {
    return (s ?? "").toString()
      .replaceAll("&","&amp;").replaceAll("<","&lt;").replaceAll(">","&gt;")
      .replaceAll('"',"&quot;").replaceAll("'","&#039;");
  }

  function highlight(text, queryNorm) {
    if (!queryNorm) return escapeHTML(text);
    const raw = (text ?? "").toString();
    const q = queryNorm;
    // 공백 무시 검색을 시각적으로 하이라이트하기 위해, 원본 문자열을 순회하며 매칭 위치를 잡기 어렵다.
    // 대신 "공백 제거한 버전" 기준으로 대략적인 하이라이트: 간단/빠르게(곡명 길이 짧아서 실사용 OK)
    const rawNoSpace = raw.replace(/[\s\u200B\u200C\u200D\uFEFF]+/g, "");
    const idx = rawNoSpace.toLowerCase().indexOf(q);
    if (idx < 0) return escapeHTML(raw);

    // idx 위치를 원본에서 근사 매핑
    let start = -1, end = -1;
    let count = 0;
    for (let i=0; i<raw.length; i++) {
      const ch = raw[i];
      const isSpace = /[\s\u200B\u200C\u200D\uFEFF]/.test(ch);
      if (isSpace) continue;
      if (count === idx) start = i;
      if (count === idx + q.length - 1) { end = i; break; }
      count++;
    }
    if (start < 0 || end < 0) return escapeHTML(raw);

    const a = escapeHTML(raw.slice(0, start));
    const b = escapeHTML(raw.slice(start, end+1));
    const c = escapeHTML(raw.slice(end+1));
    return a + "<mark>" + b + "</mark>" + c;
  }

  function renderSongList() {
    const qn = norm(elQ.value);
    const filtered = qn ? SONGS.filter(s => norm(s).includes(qn)) : SONGS;

    elSongCount.textContent = filtered.length.toString();
    elSongs.innerHTML = filtered.length ? "" : `<div class="empty">아직 곡 목록을 못 불러왔어요.</div>`;

    filtered.slice(0, 400).forEach(song => {
      const div = document.createElement("div");
      div.className = "item";
      div.innerHTML = `<span>${highlight(song, qn)}</span><span class="chip">클릭</span>`;
      div.addEventListener("click", () => {
        elQ.value = song;
        closeAC();
        renderAll();
        elQ.focus();
      });
      elSongs.appendChild(div);
    });

    if (filtered.length > 400) {
      const more = document.createElement("div");
      more.className = "empty";
      more.textContent = "목록이 많아서 400개까지만 보여줘요. 검색어를 더 입력해 주세요.";
      elSongs.appendChild(more);
    }
  }

  function renderResults() {
    const qn = norm(elQ.value);
    if (!qn) {
      elResultsTable.style.display = "none";
      elResultsEmpty.style.display = "block";
      elResultsEmpty.textContent = "검색어를 입력하면 결과가 나와요.";
      elResults.innerHTML = "";
      return;
    }

    const matches = DB.filter(r => r.songKey.includes(qn));
    const uniq = uniqueBy(matches, r => `${r.partKey}||${r.personKey}`)
      .sort((a,b) => a.part.localeCompare(b.part, "ko"));

    elResults.innerHTML = "";
    if (!uniq.length) {
      elResultsTable.style.display = "none";
      elResultsEmpty.style.display = "block";
      elResultsEmpty.textContent = "결과 없음";
      return;
    }

    elResultsEmpty.style.display = "none";
    elResultsTable.style.display = "table";
    uniq.forEach(r => {
      const tr = document.createElement("tr");
      tr.innerHTML = `<td>${escapeHTML(r.part)}</td><td>${escapeHTML(r.person)}</td>`;
      elResults.appendChild(tr);
    });
  }

  function renderAll() {
    renderSongList();
    renderResults();
    renderAC(); // 자동완성도 같이
  }

  function extractSheetId(url) {
    const m = url.match(/spreadsheets\/d\/([a-zA-Z0-9-_]+)/);
    return m ? m[1] : null;
  }

  // GVIZ JSONP
  function loadGvizJSONP(url) {
    return new Promise((resolve, reject) => {
      const cbName = "__gviz_cb_" + Math.random().toString(36).slice(2);
      const timeout = setTimeout(() => {
        cleanup();
        reject(new Error("타임아웃: GVIZ 응답 없음"));
      }, 12000);

      let script = null;
      function cleanup() {
        clearTimeout(timeout);
        try { delete window[cbName]; } catch(e) {}
        if (script && script.parentNode) script.parentNode.removeChild(script);
      }

      window[cbName] = (payload) => {
        try {
          cleanup();
          if (payload && payload.status && payload.status !== "ok") {
            const msg = (payload.errors && payload.errors.length)
              ? payload.errors.map(e => e.message).join(" / ")
              : ("GVIZ status=" + payload.status);
            reject(new Error(msg));
            return;
          }
          const table = payload.table;
          const cols = table.cols.map(c => (c.label || "").trim());
          const rows = table.rows.map(r => r.c.map(c => (c && c.v !== null && c.v !== undefined) ? String(c.v) : ""));
          resolve({ cols, rows });
        } catch (e) {
          reject(new Error("파싱 실패: " + e.message));
        }
      };

      window.google = window.google || {};
      window.google.visualization = window.google.visualization || {};
      window.google.visualization.Query = window.google.visualization.Query || {};
      window.google.visualization.Query.setResponse = (payload) => window[cbName](payload);

      script = document.createElement("script");
      script.src = url;
      script.onerror = () => { cleanup(); reject(new Error("스크립트 로드 실패")); };
      document.head.appendChild(script);
    });
  }

  function findCol(headerNorm, candidates) {
    for (const c of candidates) {
      const i = headerNorm.findIndex(h => h === c);
      if (i >= 0) return i;
    }
    for (const c of candidates) {
      const i = headerNorm.findIndex(h => h.includes(c));
      if (i >= 0) return i;
    }
    return -1;
  }

  function toDBWithHeaderRecovery(cols, rows) {
    const songKeys = ["곡명","노래","노래제목","제목","song","title","name"];
    const partKeys = ["파트악기","파트","악기","instrument","part","세션"];
    const personKeys = ["담당자","담당","연주자","멤버","이름","person","member"];

    // 1) cols(label)로 매칭
    let header = cols.map(h => norm(h));
    let dataRows = rows;

    let idxSong = findCol(header, songKeys);
    let idxPart = findCol(header, partKeys);
    let idxPerson = findCol(header, personKeys);

    // 2) cols가 비어있거나 이상하면, 첫 행을 헤더로 복구
    const labelsBad = header.every(h => !h || /^[a-z]$/.test(h));
    if ((idxSong < 0 || idxPart < 0 || idxPerson < 0) && dataRows.length && labelsBad) {
      const firstRowHeader = dataRows[0].map(v => norm(v));
      const s2 = findCol(firstRowHeader, songKeys);
      const p2 = findCol(firstRowHeader, partKeys);
      const m2 = findCol(firstRowHeader, personKeys);
      if (s2 >= 0 && p2 >= 0 && m2 >= 0) {
        header = firstRowHeader;
        idxSong = s2; idxPart = p2; idxPerson = m2;
        dataRows = dataRows.slice(1);
      }
    }

    if (idxSong < 0 || idxPart < 0 || idxPerson < 0) {
      const seenCols = cols.filter(Boolean).slice(0, 12).join(", ") || "(빈 헤더)";
      const seenRow1 = (rows[0] ? rows[0].slice(0, 12).join(", ") : "(첫행 없음)");
      throw new Error("컬럼 매칭 실패\n감지된 헤더(label): " + seenCols + "\n첫 행(미리보기): " + seenRow1);
    }

    const out = [];
    for (const r of dataRows) {
      const song = trim(r[idxSong] ?? "");
      const part = trim(r[idxPart] ?? "");
      const person = trim(r[idxPerson] ?? "");
      if (!song || !part || !person) continue;
      out.push({ song, part, person, songKey: norm(song), partKey: norm(part), personKey: norm(person) });
    }
    return out;
  }

  async function connect() {
    const sheetId = extractSheetId(FIXED_SHEET_URL);
    if (!sheetId) {
      setStatus("시트 ID를 못 찾음(링크 형식 확인)", false);
      return;
    }

    const base = "https://docs.google.com/spreadsheets/d/" + sheetId + "/gviz/tq?tqx=out:json";
    const urlBySheet = base + "&sheet=" + encodeURIComponent(FIXED_SHEET_NAME);
    const urlByGid = base + "&gid=" + encodeURIComponent(FIXED_GID);

    elDebug.innerHTML =
      "<div><b>시도1</b> (탭이름): <code>" + urlBySheet + "</code></div>" +
      "<div style='margin-top:6px;'><b>시도2</b> (gid): <code>" + urlByGid + "</code></div>";

    setStatus("데이터 불러오는 중…", true, true);

    try {
      // 1) 탭 이름 방식
      try {
        const p1 = await loadGvizJSONP(urlBySheet);
        DB = toDBWithHeaderRecovery(p1.cols, p1.rows);
      } catch (e1) {
        // 2) gid 방식
        const p2 = await loadGvizJSONP(urlByGid);
        DB = toDBWithHeaderRecovery(p2.cols, p2.rows);
      }

      rebuildSongs();
      renderAll();
      setStatus("연결 완료 (" + DB.length + "행)", true, false);
    } catch (e) {
      setStatus("연결 실패\n" + e.message, false, false);
    }
  }

  // ===== 자동완성(부드럽게) =====
  let acItems = [];
  let acIndex = -1;

  function openAC() {
    if (!acItems.length) return closeAC();
    elAC.style.display = "block";
    elAC.setAttribute("aria-hidden", "false");
  }
  function closeAC() {
    elAC.style.display = "none";
    elAC.setAttribute("aria-hidden", "true");
    acIndex = -1;
  }

  function scrollACIntoView() {
    const rows = elACBody.querySelectorAll(".acRow");
    const row = rows[acIndex];
    if (!row) return;
    const top = row.offsetTop;
    const bottom = top + row.offsetHeight;
    if (top < elACBody.scrollTop) elACBody.scrollTop = top;
    else if (bottom > elACBody.scrollTop + elACBody.clientHeight) elACBody.scrollTop = bottom - elACBody.clientHeight;
  }

  function chooseAC(i) {
    const item = acItems[i];
    if (!item) return;
    elQ.value = item;
    closeAC();
    renderAll();
    elQ.focus();
  }

  function renderAC() {
    const qn = norm(elQ.value);

    if (!qn) {
      closeAC();
      elACBody.innerHTML = "";
      elACMeta.textContent = "0";
      return;
    }

    acItems = SONGS.filter(s => norm(s).includes(qn)).slice(0, 10);
    elACMeta.textContent = String(acItems.length);

    if (!acItems.length) {
      closeAC();
      return;
    }

    openAC();
    elACBody.innerHTML = "";
    acItems.forEach((song, idx) => {
      const row = document.createElement("div");
      row.className = "acRow" + (idx === acIndex ? " active" : "");
      row.innerHTML = `<span>${highlight(song, qn)}</span><span class="chip">Enter</span>`;
      row.addEventListener("mousedown", (e) => {
        // blur 전에 선택되도록 mousedown 사용
        e.preventDefault();
        chooseAC(idx);
      });
      elACBody.appendChild(row);
    });
  }

  function stepAC(dir) {
    if (elAC.style.display !== "block") return;
    if (!acItems.length) return;
    acIndex = acIndex + dir;
    if (acIndex < 0) acIndex = acItems.length - 1;
    if (acIndex >= acItems.length) acIndex = 0;

    const rows = elACBody.querySelectorAll(".acRow");
    rows.forEach((r, i) => r.classList.toggle("active", i === acIndex));
    scrollACIntoView();
  }

  // ===== 이벤트 =====
  elQ.addEventListener("input", () => {
    renderAll();
  });

  elQ.addEventListener("keydown", (e) => {
    if (e.key === "ArrowDown") {
      e.preventDefault();
      if (elAC.style.display !== "block") renderAC();
      stepAC(+1);
    } else if (e.key === "ArrowUp") {
      e.preventDefault();
      if (elAC.style.display !== "block") renderAC();
      stepAC(-1);
    } else if (e.key === "Enter") {
      if (elAC.style.display === "block" && acIndex >= 0) {
        e.preventDefault();
        chooseAC(acIndex);
      }
    } else if (e.key === "Escape") {
      closeAC();
    }
  });

  elQ.addEventListener("focus", () => {
    renderAC();
  });

  document.addEventListener("click", (e) => {
    // 자동완성 바깥 클릭 시 닫기
    if (!elAC.contains(e.target) && e.target !== elQ) closeAC();
  });

  elClear.addEventListener("click", () => {
    elQ.value = "";
    closeAC();
    renderAll();
    elQ.focus();
  });

  // ===== 시작 =====
  setStatus("데이터 불러오는 중…", true, true);
  connect();
  renderAll();
</script>
</body>
</html>
