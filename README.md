
<html lang="ko">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>쉐키나 찬양팀 곡 검색</title>
  <style>
    :root { --bd:#e5e7eb; --fg:#111827; --muted:#6b7280; --bg:#ffffff; --chip:#f3f4f6; --warn:#991b1b; --ok:#065f46; }
    body { margin:0; font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Noto Sans KR", Arial, sans-serif; background: var(--bg); color: var(--fg); }
    .wrap { max-width: 1040px; margin: 0 auto; padding: 20px; }
    h1 { font-size: 20px; margin: 0 0 6px; }
    .sub { color: var(--muted); font-size: 13px; margin-bottom: 14px; line-height: 1.5; }
    .card { border:1px solid var(--bd); border-radius: 14px; padding: 14px; margin: 12px 0; }
    label { font-size: 12px; color: var(--muted); display:block; margin-bottom: 6px; }
    input[type="text"] { width:100%; padding: 12px 12px; border:1px solid var(--bd); border-radius: 10px; font-size: 15px; }
    .grid { display:grid; grid-template-columns: 1fr 1fr; gap: 12px; }
    @media (max-width: 800px) { .grid { grid-template-columns: 1fr; } }
    .list { border:1px solid var(--bd); border-radius: 12px; overflow:hidden; }
    .list .head { padding:10px 12px; background:#fafafa; border-bottom:1px solid var(--bd); font-weight: 700; font-size: 13px; display:flex; align-items:center; justify-content:space-between; gap:10px; }
    .list .body { max-height: 380px; overflow:auto; }
    .item { padding:10px 12px; border-bottom:1px solid var(--bd); font-size: 14px; display:flex; justify-content:space-between; gap:10px; cursor:pointer; }
    .item:hover { background:#fafafa; }
    .item:last-child { border-bottom: 0; }
    .chip { font-size: 12px; background: var(--chip); padding: 2px 8px; border-radius: 999px; color: #374151; white-space:nowrap; }
    table { width:100%; border-collapse: collapse; }
    th, td { border-bottom:1px solid var(--bd); padding: 10px 10px; text-align:left; font-size: 14px; }
    th { font-size: 13px; color:#374151; background:#fafafa; position: sticky; top:0; }
    .empty { color: var(--muted); font-size: 13px; padding:  12px; }
    .status { font-size: 12px; white-space:pre-wrap; }
    .status.ok { color: var(--ok); }
    .status.bad { color: var(--warn); }
    .k { font-weight:700; }
    .hint { font-size: 12px; color: var(--muted); margin-top: 8px; line-height: 1.45; }
    code { background:#f3f4f6; padding: 1px 6px; border-radius: 6px; }
    details summary { cursor:pointer; color:#374151; font-weight:700; }
    details .hint { margin-top:10px; }
  </style>
</head>
<body>
  <div class="wrap">
    <h1>쉐키나 찬양팀 곡 검색</h1>
    <div class="sub">
      곡명 검색하면 해당 곡 세션(파트/담당자)이 전부 나옵니다. <span class="k">띄어쓰기 무시</span> / <span class="k">중복 제거</span><br/>
      결과표는 검색어 입력했을 때만 뜨고, 추천 곡명은 항상 보입니다.
    </div>

    <div class="card">
      <label>곡명 검색</label>
      <input id="q" type="text" placeholder="곡명 검색 (예: 꽃들 / 주님 사랑 등)" autocomplete="off" />
      <div class="hint">추천 곡명을 클릭해도 돼요.</div>
      <div id="status" class="status" style="margin-top:10px;"></div>

      <details style="margin-top:10px;">
        <summary>문제 생길 때 확인</summary>
        <div class="hint">
          상태창에 오류가 뜨면, 아래 정보(시도 URL/감지된 헤더)를 복사해서 보내면 바로 잡을 수 있어요.
          <div id="debug" style="margin-top:8px;"></div>
        </div>
      </details>
    </div>

    <div class="grid">
      <div class="list">
        <div class="head"><span>추천 곡명</span><span class="chip" id="songCount">0</span></div>
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
      <div class="k" style="margin-bottom:8px;">DB 시트 형식</div>
      <div class="hint">
        DB 탭 1행 헤더는 가능하면 이렇게: <code>곡명</code> / <code>파트(악기)</code> / <code>담당자</code><br/>
        (v4는 헤더가 꼬여도 1행을 헤더로 다시 잡아서 최대한 복구합니다)
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
  const elSongs = document.getElementById("songs");
  const elSongCount = document.getElementById("songCount");
  const elResultsEmpty = document.getElementById("resultsEmpty");
  const elResultsTable = document.getElementById("resultsTable");
  const elResults = document.getElementById("results");
  const elStatus = document.getElementById("status");
  const elDebug = document.getElementById("debug");

  function setStatus(msg, ok=true) {
    elStatus.textContent = msg;
    elStatus.className = "status " + (ok ? "ok" : "bad");
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

  function renderSongs() {
    const q = norm(elQ.value);
    const filtered = q ? SONGS.filter(s => norm(s).includes(q)) : SONGS;
    elSongCount.textContent = filtered.length.toString();
    elSongs.innerHTML = filtered.length ? "" : `<div class="empty">아직 곡 목록을 못 불러왔어요.</div>`;

    filtered.slice(0, 400).forEach(song => {
      const div = document.createElement("div");
      div.className = "item";
      div.innerHTML = `<span>${song}</span><span class="chip">클릭</span>`;
      div.addEventListener("click", () => {
        elQ.value = song;
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
    const q = norm(elQ.value);
    if (!q) {
      elResultsTable.style.display = "none";
      elResultsEmpty.style.display = "block";
      elResultsEmpty.textContent = "검색어를 입력하면 결과가 나와요.";
      elResults.innerHTML = "";
      return;
    }

    const matches = DB.filter(r => r.songKey.includes(q));
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
      tr.innerHTML = `<td>${r.part}</td><td>${r.person}</td>`;
      elResults.appendChild(tr);
    });
  }

  function renderAll() {
    renderSongs();
    renderResults();
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

    // 1) 기본: cols(label)로 매칭
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
        dataRows = dataRows.slice(1); // 헤더 행 제거
      }
    }

    // 3) 그래도 실패면, 디버그용 헤더 노출
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
      "<div><span class='k'>시도1</span> (탭이름): <code>" + urlBySheet + "</code></div>" +
      "<div style='margin-top:6px;'><span class='k'>시도2</span> (gid): <code>" + urlByGid + "</code></div>";

    setStatus("데이터 불러오는 중…", true);

    try {
      // 1) 탭 이름 방식
      try {
        const p1 = await loadGvizJSONP(urlBySheet);
        DB = toDBWithHeaderRecovery(p1.cols, p1.rows);
        rebuildSongs(); renderAll();
        setStatus("연결 완료: " + DB.length + "행", true);
        return;
      } catch (e1) {
        // 2) gid 방식
        const p2 = await loadGvizJSONP(urlByGid);
        DB = toDBWithHeaderRecovery(p2.cols, p2.rows);
        rebuildSongs(); renderAll();
        setStatus("연결 완료: " + DB.length + "행", true);
        return;
      }
    } catch (e) {
      setStatus("연결 실패\n" + e.message, false);
    }
  }

  elQ.addEventListener("input", renderAll);
  connect();
  renderAll();
</script>
</body>
</html>
