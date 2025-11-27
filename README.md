<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Movie Series Showcase</title>
  <style>
    :root{
      --bg:#0f1724; --card:#0b1220; --muted:#9aa4b2; --accent:#ff6b6b; --glass: rgba(255,255,255,0.03);
      --radius:14px; --gap:18px; font-family: Inter, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
    }
    *{box-sizing:border-box}
    html,body{height:100%;margin:0;background:linear-gradient(180deg,#071029 0%, #071629 60%);color:#e6eef6}
    .wrap{max-width:1100px;margin:36px auto;padding:24px}

    header{display:flex;gap:18px;align-items:center;justify-content:space-between}
    h1{margin:0;font-size:1.6rem;letter-spacing:-0.5px}
    p.lead{margin:4px 0 0;color:var(--muted);font-size:0.94rem}

    .controls{display:flex;gap:12px;align-items:center;margin:18px 0 22px}
    .search{flex:1;display:flex;gap:8px}
    .input,select{background:var(--card);border:1px solid rgba(255,255,255,0.03);padding:10px 12px;border-radius:10px;color:inherit}
    .btn{background:linear-gradient(90deg,var(--accent),#ff9a9a);border:none;padding:10px 14px;border-radius:10px;color:#071029;cursor:pointer;font-weight:600}

    .grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(220px,1fr));gap:18px}
    .card{background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));padding:14px;border-radius:var(--radius);box-shadow:0 6px 18px rgba(2,6,23,0.6);display:flex;flex-direction:column;gap:12px}
    .poster{height:140px;border-radius:12px;display:flex;align-items:end;padding:12px;color:white;font-weight:700;letter-spacing:0.3px}
    .poster .info{background:linear-gradient(180deg, rgba(0,0,0,0) 0%, rgba(0,0,0,0.45) 60%);padding:8px;border-radius:8px}

    .meta{display:flex;gap:8px;flex-wrap:wrap;color:var(--muted);font-size:0.88rem}
    .actions{margin-top:auto;display:flex;gap:8px}
    .tag{background:var(--glass);padding:6px 8px;border-radius:999px;font-size:0.8rem}

    /* modal */
    .modal-backdrop{position:fixed;inset:0;background:linear-gradient(180deg, rgba(0,0,0,0.55), rgba(0,0,0,0.6));display:none;align-items:center;justify-content:center;padding:20px}
    .modal{width:100%;max-width:880px;background:#071427;padding:18px;border-radius:12px;box-shadow:0 10px 40px rgba(0,0,0,0.7)}
    .modal .row{display:flex;gap:14px}
    .modal .left{flex:0 0 260px}
    .modal .right{flex:1}
    .close{background:transparent;border:0;color:var(--muted);font-weight:700;cursor:pointer}

    footer{margin-top:28px;color:var(--muted);font-size:0.9rem}

    /* responsive tweaks */
    @media (max-width:640px){.modal .row{flex-direction:column}.poster{height:180px}}
  </style>
</head>
<body>
  <div class="wrap">
    <header>
      <div>
        <h1>Movie Series Showcase</h1>
        <p class="lead">Browse, search, and explore popular movie series — built with plain HTML/CSS/JS.</p>
      </div>
      <div>
        <button id="view-favs" class="btn">View Favorites</button>
      </div>
    </header>

    <div class="controls">
      <div class="search">
        <input id="q" class="input" placeholder="Search series title, genre, or year..." />
        <select id="genre" class="input" style="flex:0 0 160px">
          <option value="all">All genres</option>
          <option value="action">Action</option>
          <option value="sci-fi">Sci‑Fi</option>
          <option value="fantasy">Fantasy</option>
          <option value="adventure">Adventure</option>
        </select>
      </div>
      <select id="sort" class="input" style="width:160px">
        <option value="popular">Sort: Popular</option>
        <option value="alpha">Sort: A→Z</option>
        <option value="year">Sort: Newest</option>
      </select>
    </div>

    <main>
      <div id="grid" class="grid" aria-live="polite"></div>
    </main>

    <footer>Tip: Click a card to view details and trailer. Favorites are saved locally in your browser.</footer>
  </div>

  <!-- modal -->
  <div id="modalBackdrop" class="modal-backdrop" role="dialog" aria-modal="true">
    <div class="modal" role="document">
      <div style="display:flex;justify-content:space-between;align-items:center">
        <strong id="modalTitle">Series Title</strong>
        <button id="modalClose" class="close">✕</button>
      </div>
      <div class="row" style="margin-top:12px">
        <div class="left">
          <div id="modalPoster" style="border-radius:10px;overflow:hidden;height:240px;background:#12263a;display:flex;align-items:center;justify-content:center;color:#e6eef6;font-weight:700"></div>
        </div>
        <div class="right">
          <p id="modalDesc" style="color:var(--muted)">Description</p>
          <div style="margin-top:12px" id="modalMeta" class="meta"></div>
          <div style="margin-top:14px">
            <button id="favToggle" class="btn">Add to favorites</button>
            <button id="playTrailer" class="input" style="margin-left:8px">Play trailer</button>
          </div>
          <div id="trailerWrap" style="margin-top:12px;display:none">
            <div style="position:relative;padding-top:56.25%">
              <iframe id="trailerFrame" src="" style="position:absolute;inset:0;border:0;width:100%;height:100%" allowfullscreen></iframe>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>

  <script>
    /* Sample data for movie series */
    const SERIES = [
      {
        id: 'mcu', title: 'Marvel Cinematic Universe', year: 2008, genre: 'action', seasons: 'Phase 1–6', rating: 'PG-13', popular: 99,
        desc: 'Interconnected superhero films and shows from Marvel Studios spanning multiple phases and characters.',
        trailer: 'https://www.youtube.com/embed/6ZfuNTqbHE8'
      },
      {id:'starwars', title:'Star Wars', year:1977, genre:'sci-fi', seasons:'Saga + Anthologies', rating:'PG-13', popular:95,
        desc:'A space-fantasy saga about the Skywalker family, the Jedi, and the struggle between the light and dark sides.',
        trailer:'https://www.youtube.com/embed/8Qn_spdM5Zg'
      },
      {id:'harry', title:'Harry Potter', year:2001, genre:'fantasy', seasons:'8 films', rating:'PG', popular:92,
        desc:'The cinematic adaptation of J.K. Rowling\'s novels following the young wizard Harry Potter and his friends.',
        trailer:'https://www.youtube.com/embed/VyHV0BRtdxo'
      },
      {id:'lotr', title:'The Lord of the Rings', year:2001, genre:'fantasy', seasons:'Trilogy + The Hobbit', rating:'PG-13', popular:90,
        desc:'Epic high fantasy series adapted from J.R.R. Tolkien\'s works; battles for Middle‑earth and the One Ring.',
        trailer:'https://www.youtube.com/embed/V75dMMIW2B4'
      },
      {id:'fast', title:'Fast & Furious', year:2001, genre:'action', seasons:'10+ films', rating:'PG-13', popular:85,
        desc:'High-octane action series centered on street racing, heists, and an extended found-family of heroes.',
        trailer:'https://www.youtube.com/embed/uisBaTkQAEs'
      }
    ];

    const grid = document.getElementById('grid');
    const q = document.getElementById('q');
    const genre = document.getElementById('genre');
    const sort = document.getElementById('sort');
    const modalBackdrop = document.getElementById('modalBackdrop');
    const modalTitle = document.getElementById('modalTitle');
    const modalDesc = document.getElementById('modalDesc');
    const modalPoster = document.getElementById('modalPoster');
    const modalMeta = document.getElementById('modalMeta');
    const trailerWrap = document.getElementById('trailerWrap');
    const trailerFrame = document.getElementById('trailerFrame');
    const favToggle = document.getElementById('favToggle');
    const viewFavs = document.getElementById('view-favs');
    const playTrailer = document.getElementById('playTrailer');

    // simple localStorage favorites
    const FAV_KEY = 'movie_series_favs_v1';
    function loadFavs(){try{return JSON.parse(localStorage.getItem(FAV_KEY)||'[]')}catch(e){return []}}
    function saveFavs(list){localStorage.setItem(FAV_KEY, JSON.stringify(list))}

    function isFav(id){return loadFavs().includes(id)}
    function toggleFav(id){const list=loadFavs(); if(list.includes(id)){const i=list.indexOf(id);list.splice(i,1)}else{list.push(id)} saveFavs(list)}

    function makePoster(series){
      // stylized CSS poster with initials
      const initials = series.title.split(' ').map(w=>w[0]).slice(0,3).join('');
      const grad = getGradientFor(series.id);
      return `
        <div class=poster style="background:${grad}">
          <div style="display:flex;flex-direction:column;gap:8px;width:100%">
            <div style="font-size:14px;opacity:0.92">${series.title}</div>
            <div class="info">${series.year} • ${series.rating}</div>
          </div>
        </div>
      `;
    }

    function getGradientFor(id){
      const opts = {
        mcu: 'linear-gradient(135deg,#3b82f6 0%, #9333ea 100%)',
        starwars: 'linear-gradient(135deg,#0ea5a4 0%, #0369a1 100%)',
        harry: 'linear-gradient(135deg,#f97316 0%, #ef4444 100%)',
        lotr: 'linear-gradient(135deg,#10b981 0%, #0ea5a4 100%)',
        fast: 'linear-gradient(135deg,#ef4444 0%, #f97316 100%)'
      };
      return opts[id] || 'linear-gradient(135deg,#475569,#1f2937)';
    }

    function render(list){
      grid.innerHTML = '';
      if(!list.length){grid.innerHTML = '<div style="color:var(--muted)">No results — try a different search.</div>';return}
      list.forEach(s=>{
        const el = document.createElement('div'); el.className='card';
        el.innerHTML = `
          ${makePoster(s)}
          <div style="display:flex;flex-direction:column;gap:6px">
            <div style="display:flex;justify-content:space-between;align-items:center">
              <div style="font-weight:700">${s.title}</div>
              <div class="tag">${s.genre}</div>
            </div>
            <div class="meta">${s.seasons} • ${s.year}</div>
            <div class="actions">
              <button class="input" data-id="${s.id}" data-action="open">Details</button>
              <button class="input" data-id="${s.id}" data-action="fav">${isFav(s.id)?'♥ Favorited':'☆ Add'}</button>
            </div>
          </div>
        `;
        // click handler
        el.addEventListener('click', (ev)=>{
          const action = ev.target.getAttribute('data-action');
          const id = ev.target.getAttribute('data-id');
          if(action==='fav'){ toggleFav(id); render(applyFilters()); return; }
          openModal(s);
        });
        grid.appendChild(el);
      })
    }

    function applyFilters(){
      const qq = q.value.trim().toLowerCase();
      const g = genre.value;
      let list = SERIES.filter(s=>{
        if(g!=='all' && s.genre!==g) return false;
        if(!qq) return true;
        return [s.title, s.genre, String(s.year), s.seasons, s.desc].join(' ').toLowerCase().includes(qq);
      });
      // sort
      if(sort.value==='alpha') list.sort((a,b)=>a.title.localeCompare(b.title));
      else if(sort.value==='year') list.sort((a,b)=>b.year - a.year);
      else list.sort((a,b)=>b.popular - a.popular);
      return list;
    }

    function openModal(series){
      modalTitle.textContent = series.title;
      modalDesc.textContent = series.desc;
      modalPoster.innerHTML = `<div style="width:100%;height:100%;display:flex;align-items:center;justify-content:center;font-size:22px;font-weight:800">${series.title}</div>`;
      modalMeta.innerHTML = `<div class="tag">${series.genre}</div><div class="tag">${series.year}</div><div class="tag">${series.rating}</div>`;
      favToggle.textContent = isFav(series.id)?'Remove from favorites':'Add to favorites';
      favToggle.onclick = ()=>{ toggleFav(series.id); favToggle.textContent = isFav(series.id)?'Remove from favorites':'Add to favorites'; }
      playTrailer.onclick = ()=>{ trailerFrame.src = series.trailer + '?rel=0&autoplay=1'; trailerWrap.style.display='block'; }

      modalBackdrop.style.display = 'flex';
      trailerWrap.style.display='none'; trailerFrame.src='';
    }

    document.getElementById('modalClose').addEventListener('click', ()=>{ modalBackdrop.style.display='none'; trailerFrame.src=''; });
    modalBackdrop.addEventListener('click', (e)=>{ if(e.target===modalBackdrop){ modalBackdrop.style.display='none'; trailerFrame.src=''; } });

    // live search
    [q,genre,sort].forEach(el=>el.addEventListener('input', ()=>render(applyFilters())));

    viewFavs.addEventListener('click', ()=>{
      const favs = loadFavs(); const list = SERIES.filter(s=>favs.includes(s.id)); render(list);
    });

    // initial render
    render(applyFilters());
  </script>
</body>
</html>

