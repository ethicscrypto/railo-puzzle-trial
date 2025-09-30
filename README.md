# railo-puzzle-trial

<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Railo Puzzle — Black & White</title>
  <style>
    :root{
      --bg:#ffffff; /* white background */
      --fg:#000000; /* black foreground */
      --accent:#111111;
      --card:#f6f6f6;
      --radius:12px;
      font-family: Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
    }
    html,body{height:100%;}
    body{
      margin:0;display:flex;align-items:center;justify-content:center;
      background:linear-gradient(180deg,var(--bg) 0%, #f2f2f2 100%);
      color:var(--fg);
    }
    .wrap{
      width:980px; max-width:96vw; display:grid; grid-template-columns: 1fr 360px; gap:28px; align-items:start;
    }
    .game-card{
      background:var(--card); border-radius:var(--radius); padding:18px; box-shadow:0 10px 30px rgba(0,0,0,0.06);
    }
    header{display:flex;align-items:center;gap:12px;margin-bottom:12px}
    .logo{width:56px;height:56px;border-radius:12px;background:var(--fg);display:flex;align-items:center;justify-content:center;color:var(--bg);font-weight:700;font-size:22px}
    h1{font-size:20px;margin:0}
    .controls{display:flex;gap:8px;margin-bottom:12px;flex-wrap:wrap}
    button{background:var(--fg);color:var(--bg);border:none;padding:10px 14px;border-radius:10px;cursor:pointer;font-weight:600}
    .small{background:transparent;color:var(--fg);border:1px solid #ddd;padding:8px 10px}

    /* puzzle area */
    .board-wrap{display:flex;gap:18px}
    .canvas-wrap{background:var(--bg);border-radius:10px;padding:10px;display:flex;align-items:center;justify-content:center}
    #puzzle{background:var(--bg);display:block;border-radius:6px}

    /* sidebar */
    .sidebar{background:transparent}
    .card{background:var(--card);padding:14px;border-radius:12px;margin-bottom:12px}
    label{display:block;font-size:13px;margin-bottom:6px}
    input[type=range]{width:100%}
    .stats{display:flex;gap:8px;flex-wrap:wrap}
    .stat{flex:1;background:#fff;padding:10px;border-radius:8px;border:1px solid #eee;text-align:center}
    .leaderboard{max-height:240px;overflow:auto;padding:8px}
    .leaderboard li{display:flex;justify-content:space-between;padding:8px;border-bottom:1px dashed #eee}

    footer{font-size:12px;color:#666;margin-top:6px}

    /* piece highlight */
    .piece-selected{outline:3px solid #000}

    /* responsive */
    @media (max-width:900px){
      .wrap{grid-template-columns:1fr;}
      .sidebar{order:2}
    }
  </style>
</head>
<body>
  <div class="wrap">
    <div class="game-card">
      <header>
        <div class="logo">R</div>
        <div>
          <h1>Railo Puzzle — Match the image</h1>
          <div style="font-size:13px;color:#444">Black & White, simple, attractive — complete the puzzle as fast as possible</div>
        </div>
      </header>

      <div class="controls">
        <button id="newBtn">New Puzzle</button>
        <button id="shuffleBtn" class="small">Shuffle</button>
        <button id="solveBtn" class="small">Show Solution</button>
        <button id="connectTwitter">Connect Twitter</button>
        <button id="shareBtn">Share Result</button>
      </div>

      <div class="board-wrap">
        <div class="canvas-wrap game-area">
          <!-- Canvas where puzzle pieces are drawn -->
          <canvas id="puzzle" width="640" height="640"></canvas>
        </div>
        <div class="sidebar">
          <div class="card">
            <label>Difficulty (grid size)</label>
            <input id="gridRange" type="range" min="2" max="6" value="4">
            <div style="font-size:13px;margin-top:8px">Pieces per row: <span id="gridVal">4</span> × <span id="gridVal">4</span></div>
          </div>

          <div class="card">
            <div style="display:flex;gap:8px;align-items:center;justify-content:space-between;margin-bottom:8px">
              <div>
                <div style="font-size:12px;color:#666">Timer</div>
                <div id="timer" style="font-size:18px;font-weight:700">00:00.000</div>
              </div>
              <div>
                <div style="font-size:12px;color:#666">Moves</div>
                <div id="moves" style="font-size:18px;font-weight:700">0</div>
              </div>
            </div>
            <div style="font-size:13px;color:#666">Select a piece then click a target piece to swap. Snap into place when correct.</div>
          </div>

          <div class="card">
            <div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:8px">
              <div style="font-size:13px;color:#666">Your name</div>
              <input id="playerName" placeholder="Anonymous" style="border:1px solid #ddd;padding:6px;border-radius:8px">
            </div>
            <div style="font-size:13px;color:#666;margin-bottom:8px">When you finish, your time will be saved to the leaderboard (fastest first).</div>
            <div style="display:flex;gap:8px">
              <button id="saveScore">Save Score</button>
            </div>
          </div>

          <div class="card">
            <h3 style="margin:0 0 8px 0">Leaderboard (Fastest)</h3>
            <ul id="leaderboard" class="leaderboard"></ul>
          </div>

          <footer>Built with ♥ — Black & White theme • Shows <strong>Railo!</strong> on completion</footer>
        </div>
      </div>
    </div>

  </div>

  <script>
    // --- Configuration ---
    // Replace this image filename with your provided image file in the same folder
    const IMAGE_SRC = 'puzzle-image.jpg';

    // If the conversation uploaded an image to /mnt/data you can save it as 'puzzle-image.jpg'
    // (When deploying, place the image file next to this HTML file.)

    const canvas = document.getElementById('puzzle');
    const ctx = canvas.getContext('2d');
    const gridRange = document.getElementById('gridRange');
    const gridValDisplays = document.querySelectorAll('#gridVal');
    const timerEl = document.getElementById('timer');
    const movesEl = document.getElementById('moves');
    const leaderboardEl = document.getElementById('leaderboard');
    const playerNameInput = document.getElementById('playerName');

    let img = new Image();
    img.crossOrigin = 'anonymous';
    img.src = IMAGE_SRC;

    // Game state
    let grid = 4;
    let pieces = []; // {sx, sy, xIndex, yIndex, correctIndex}
    let pieceSize = canvas.width / grid;
    let selected = null;
    let moves = 0;
    let startTime = null;
    let timerInterval = null;
    let solved = false;

    function formatTime(ms){
      const s = Math.floor(ms/1000);
      const mm = Math.floor(s/60).toString().padStart(2,'0');
      const ss = (s%60).toString().padStart(2,'0');
      const msRem = (ms%1000).toString().padStart(3,'0');
      return `${mm}:${ss}.${msRem}`;
    }

    function startTimer(){
      startTime = performance.now();
      if(timerInterval) clearInterval(timerInterval);
      timerInterval = setInterval(()=>{
        const now = performance.now();
        timerEl.textContent = formatTime(now - startTime);
      }, 50);
    }
    function stopTimer(){
      if(timerInterval) clearInterval(timerInterval);
      timerInterval = null;
    }

    // Build pieces once image is loaded
    img.onload = ()=>{
      resetPuzzle();
      draw();
      renderLeaderboard();
    };

    function resetPuzzle(){
      grid = parseInt(gridRange.value);
      pieceSize = canvas.width / grid;
      pieces = [];
      selected = null;
      moves = 0; movesEl.textContent = moves;
      solved = false;
      timerEl.textContent = '00:00.000';
      stopTimer();

      // Create pieces with their correct positions
      let index = 0;
      for(let y=0;y<grid;y++){
        for(let x=0;x<grid;x++){
          pieces.push({
            sx: x * (img.width / grid),
            sy: y * (img.height / grid),
            correctIndex: index,
            currentIndex: index
          });
          index++;
        }
      }
      draw();
    }

    function shufflePieces(){
      // Fisher-Yates shuffle of currentIndex
      for(let i=pieces.length-1;i>0;i--){
        const j = Math.floor(Math.random()*(i+1));
        const tmp = pieces[i].currentIndex;
        pieces[i].currentIndex = pieces[j].currentIndex;
        pieces[j].currentIndex = tmp;
      }
      moves = 0; movesEl.textContent = moves;
      selected = null;
      solved = false;
      timerEl.textContent = '00:00.000';
      stopTimer();
    }

    function draw(){
      ctx.clearRect(0,0,canvas.width,canvas.height);
      // background frame (black & white style)
      ctx.fillStyle = '#fff';
      ctx.fillRect(0,0,canvas.width,canvas.height);

      // draw each piece to its current grid slot
      for(let i=0;i<pieces.length;i++){
        const p = pieces[i];
        const ci = p.currentIndex;
        const cx = (ci % grid) * pieceSize;
        const cy = Math.floor(ci / grid) * pieceSize;
        // draw the corresponding src rectangle from the original image
        ctx.drawImage(img,
          p.sx, p.sy, img.width/grid, img.height/grid,
          cx, cy, pieceSize, pieceSize
        );
        // draw a subtle divider
        ctx.strokeStyle = '#00000020';
        ctx.strokeRect(cx+0.5, cy+0.5, pieceSize-1, pieceSize-1);
      }

      if(selected !== null){
        // highlight selected piece
        const p = pieces[selected];
        const ci = p.currentIndex;
        const cx = (ci % grid) * pieceSize;
        const cy = Math.floor(ci / grid) * pieceSize;
        ctx.lineWidth = 3;
        ctx.strokeStyle = '#000';
        ctx.strokeRect(cx+3, cy+3, pieceSize-6, pieceSize-6);
      }
    }

    function getPieceAt(px, py){
      const x = Math.floor(px / pieceSize);
      const y = Math.floor(py / pieceSize);
      if(x<0||y<0||x>=grid||y>=grid) return -1;
      const idx = y*grid + x;
      // find piece whose currentIndex == idx
      return pieces.findIndex(p=>p.currentIndex === idx);
    }

    canvas.addEventListener('click', (e)=>{
      if(solved) return;
      const rect = canvas.getBoundingClientRect();
      const x = (e.clientX - rect.left) * (canvas.width/rect.width);
      const y = (e.clientY - rect.top) * (canvas.height/rect.height);
      const pieceIdx = getPieceAt(x,y);
      if(pieceIdx < 0) return;

      if(selected === null){
        selected = pieceIdx;
        draw();
        return;
      }

      if(selected === pieceIdx){
        selected = null; draw(); return;
      }

      // swap currentIndex between selected and clicked
      const a = pieces[selected].currentIndex;
      const b = pieces[pieceIdx].currentIndex;
      pieces[selected].currentIndex = b;
      pieces[pieceIdx].currentIndex = a;
      moves++; movesEl.textContent = moves;
      selected = null;
      draw();

      // start timer on first move
      if(moves === 1 && !timerInterval){ startTimer(); }

      // check solved
      if(isSolved()){
        onSolved();
      }
    });

    function isSolved(){
      return pieces.every(p => p.currentIndex === p.correctIndex);
    }

    function onSolved(){
      solved = true;
      stopTimer();
      const elapsed = performance.now() - startTime;
      // Show Railo message (as requested)
      setTimeout(()=>{ alert('Railo! Puzzle complete — Time: ' + formatTime(elapsed)); }, 100);
      // Store automatically to leaderboard (fastest time)
      addScore({name: playerNameInput.value || 'Anonymous', time: Math.round(elapsed), date: Date.now()});
    }

    // Leaderboard storage (localStorage). Fastest first.
    const LB_KEY = 'railo_puzzle_leaderboard_v1';
    function getLeaderboard(){
      try{
        return JSON.parse(localStorage.getItem(LB_KEY)) || [];
      }catch(e){return []}
    }
    function saveLeaderboard(list){ localStorage.setItem(LB_KEY, JSON.stringify(list)); }
    function addScore(entry){
      const list = getLeaderboard();
      list.push(entry);
      list.sort((a,b)=>a.time - b.time);
      const top = list.slice(0,20);
      saveLeaderboard(top);
      renderLeaderboard();
    }
    function renderLeaderboard(){
      const list = getLeaderboard();
      leaderboardEl.innerHTML = '';
      if(list.length===0){ leaderboardEl.innerHTML = '<li style="padding:8px;color:#666">No scores yet — be the first!</li>'; return; }
      list.forEach((r,i)=>{
        const li = document.createElement('li');
        li.innerHTML = `<div style=\"font-weight:700\">${i+1}. ${escapeHtml(r.name)}</div><div>${formatTime(r.time)}</div>`;
        leaderboardEl.appendChild(li);
      });
    }

    function escapeHtml(s){ return String(s).replace(/[&<>\"]/g, c=>({"&":"&amp;","<":"&lt;",">":"&gt;","\"":"&quot;"})[c]); }

    // Buttons
    document.getElementById('newBtn').addEventListener('click', ()=>{ resetPuzzle(); draw(); });
    document.getElementById('shuffleBtn').addEventListener('click', ()=>{ shufflePieces(); draw(); });
    document.getElementById('solveBtn').addEventListener('click', ()=>{ // show solved arrangement briefly
      const backup = pieces.map(p=>p.currentIndex);
      // place correctly
      pieces.forEach(p=> p.currentIndex = p.correctIndex);
      draw();
      setTimeout(()=>{ pieces.forEach((p,i)=> p.currentIndex = backup[i]); draw(); }, 1200);
    });

    gridRange.addEventListener('input', ()=>{
      gridValDisplays.forEach(el=>el.textContent = gridRange.value);
    });
    gridRange.addEventListener('change', ()=>{ resetPuzzle(); draw(); });

    document.getElementById('saveScore').addEventListener('click', ()=>{
      if(!solved){ alert('Complete the puzzle first to save your score.'); return; }
      const list = getLeaderboard();
      // already saved on completion, but allow custom name change
      renderLeaderboard();
      alert('Score saved.');
    });

    // Twitter connect (placeholder) — requires server OAuth
    let twitterConnected = false;
    document.getElementById('connectTwitter').addEventListener('click', ()=>{
      // For a real connect, you need a backend OAuth flow. Placeholder here.
      twitterConnected = !twitterConnected;
      document.getElementById('connectTwitter').textContent = twitterConnected? 'Twitter: Connected':'Connect Twitter';
      if(twitterConnected){ playerNameInput.value = '@your_twitter_handle'; }
      alert('This UI simulates connecting Twitter. Implement OAuth on server to fully connect.');
    });

    // Share
    document.getElementById('shareBtn').addEventListener('click', ()=>{
      if(!solved){ alert('Finish the puzzle to share your result.'); return; }
      const list = getLeaderboard();
      const best = list[0];
      const text = `I completed the Railo puzzle in ${formatTime(best.time)}! Can you beat my time?`;
      const url = encodeURIComponent(location.href);
      const shareUrl = `https://twitter.com/intent/tweet?text=${encodeURIComponent(text)}&url=${url}`;
      window.open(shareUrl,'_blank');
    });

    // preload fallback: if image fails to load, draw a placeholder
    img.onerror = ()=>{
      // draw simple geometric placeholder using the black & white theme
      ctx.fillStyle = '#fff'; ctx.fillRect(0,0,canvas.width,canvas.height);
      ctx.fillStyle = '#000'; ctx.fillRect(40,40,canvas.width-80,canvas.height-80);
      ctx.fillStyle = '#fff'; ctx.fillRect(80,80,canvas.width-160,canvas.height-160);
      ctx.fillStyle = '#000'; ctx.font = '20px Arial'; ctx.textAlign='center'; ctx.fillText('PLACEHOLDER IMAGE', canvas.width/2, canvas.height/2);
    }

    // initial UI values
    gridValDisplays.forEach(el=>el.textContent = gridRange.value);

  </script>
</body>
</html>
