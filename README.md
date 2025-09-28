<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Travel Explorer — Destinations & Weather</title>
  <style>
    :root{--bg:#0f1724;--card:#0b1220;--muted:#9aa4b2;--accent:#4f46e5}
    *{box-sizing:border-box}
    body{margin:0;font-family:Inter,system-ui,Segoe UI,Roboto,'Helvetica Neue',Arial;background:linear-gradient(180deg,#071028 0%,#071429 60%);color:#e6eef8;min-height:100vh;display:flex;align-items:center;justify-content:center;padding:28px}
    .app{width:100%;max-width:980px}
    header{display:flex;align-items:center;justify-content:space-between;margin-bottom:18px}
    h1{font-size:20px;margin:0}
    p.lead{margin:4px 0 0;color:var(--muted);font-size:13px}

    .search{display:flex;gap:8px}
    .search input{padding:10px 12px;border-radius:10px;border:1px solid rgba(255,255,255,0.06);background:rgba(255,255,255,0.02);color:inherit;min-width:220px}
    .search button{background:var(--accent);border:0;padding:10px 14px;border-radius:10px;color:white;cursor:pointer}

    .grid{display:grid;grid-template-columns:360px 1fr;gap:18px}

    .panel{background:linear-gradient(180deg,rgba(255,255,255,0.02),rgba(255,255,255,0.01));padding:14px;border-radius:12px;border:1px solid rgba(255,255,255,0.03)}
    .list{display:flex;flex-direction:column;gap:10px;max-height:520px;overflow:auto;padding-right:6px}
    .dest{display:flex;gap:10px;align-items:center;padding:8px;border-radius:9px;cursor:pointer}
    .dest:hover{background:rgba(255,255,255,0.01)}
    .thumb{width:56px;height:56px;border-radius:8px;flex:0 0 56px;background:#123;background-size:cover;background-position:center}
    .meta{flex:1}
    .meta strong{display:block}
    .meta small{color:var(--muted)}

    .main-photo{height:320px;border-radius:10px;background-size:cover;background-position:center;margin-bottom:12px;border:1px solid rgba(255,255,255,0.03)}

    .weather-row{display:flex;align-items:center;gap:12px}
    .weather-info{flex:1}
    .temp{font-size:40px;font-weight:700}
    .conditions{color:var(--muted)}

    footer{margin-top:10px;color:var(--muted);font-size:13px}

    @media (max-width:900px){.grid{grid-template-columns:1fr;}.main-photo{height:220px}}
  </style>
</head>
<body>
  <div class="app">
    <header>
      <div>
        <h1>Travel Explorer</h1>
        <p class="lead">Browse destinations, view photos, and check current weather.</p>
      </div>
      <div class="search">
        <input id="cityInput" placeholder="Search city (e.g. Paris)" />
        <button id="searchBtn">Search</button>
      </div>
    </header>

    <section class="grid">
      <aside class="panel">
        <h3 style="margin-top:0">Popular destinations</h3>
        <div class="list" id="destList"></div>
      </aside>

      <main class="panel">
        <div id="photo" class="main-photo"></div>

        <div class="weather-row">
          <div class="weather-info">
            <div class="temp" id="temp">—°C</div>
            <div class="conditions" id="conditions">Search for a city to see live weather.</div>
          </div>
          <div id="iconWrap"></div>
        </div>

        <div style="margin-top:12px;color:var(--muted);font-size:14px" id="details"></div>

        <footer>
          NOTE: This demo calls public image sources and OpenWeatherMap for live weather. Replace the API key in the script before publishing.
        </footer>
      </main>
    </section>
  </div>

  <script>
    const OPENWEATHER_API_KEY = 'da2399aaa9020e952a9e4f675e86219a';

    const DESTINATIONS = [
      {name:'Paris, France', query:'paris'},
      {name:'Tokyo, Japan', query:'tokyo'},
      {name:'New York, USA', query:'new+york'},
      {name:'Bali, Indonesia', query:'bali'},
      {name:'Cape Town, South Africa', query:'cape+town'},
      {name:'Sydney, Australia', query:'sydney+opera+house'},
      {name:'Rio de Janeiro, Brazil', query:'rio+de+janeiro'},
    ];

    const destList = document.getElementById('destList');
    const photo = document.getElementById('photo');
    const tempEl = document.getElementById('temp');
    const condEl = document.getElementById('conditions');
    const detailsEl = document.getElementById('details');
    const iconWrap = document.getElementById('iconWrap');
    const cityInput = document.getElementById('cityInput');
    const searchBtn = document.getElementById('searchBtn');

    function createDestItem(d){
      const el = document.createElement('div');
      el.className='dest';
      el.innerHTML = `
        <div class="thumb" style="background-image:url('https://source.unsplash.com/56x56/?${d.query}');"></div>
        <div class="meta"><strong>${d.name}</strong><small>Click to view</small></div>
      `;
      el.addEventListener('click', ()=> selectDestination(d));
      return el;
    }

    function populateList(){
      destList.innerHTML='';
      for(const d of DESTINATIONS) destList.appendChild(createDestItem(d));
    }

    async function selectDestination(d){
      // Show photo
      let photoUrl = 'https://source.unsplash.com/1400x700/?'+d.query;
      photo.style.backgroundImage = `url('${photoUrl}')`;

      // Fetch weather
      const city = d.name.split(',')[0];
      if(!OPENWEATHER_API_KEY){
        tempEl.textContent = '—°C';
        condEl.textContent = 'API key missing';
        detailsEl.textContent = '';
        iconWrap.innerHTML = '';
        return;
      }

      try{
        condEl.textContent = 'Loading weather...';
        const url = `https://api.openweathermap.org/data/2.5/weather?q=${encodeURIComponent(city)}&appid=${OPENWEATHER_API_KEY}&units=metric`;
        const res = await fetch(url);
        if(!res.ok) throw new Error('Weather fetch failed');
        const data = await res.json();

        tempEl.textContent = Math.round(data.main.temp) + '°C';
        condEl.textContent = `${data.weather[0].main} — ${data.weather[0].description}`;
        detailsEl.textContent = `Humidity: ${data.main.humidity}%  •  Wind: ${data.wind.speed} m/s  •  Local: ${new Date((data.dt + data.timezone) * 1000).toUTCString().replace('GMT','')}`;
        const icon = data.weather[0].icon;
        iconWrap.innerHTML = `<img src="https://openweathermap.org/img/wn/${icon}@2x.png" alt="icon" style="width:68px;height:68px">`;
      }catch(err){
        condEl.textContent = 'Unable to get weather.';
        detailsEl.textContent = err.message;
        iconWrap.innerHTML='';
      }
    }

    async function searchCity(){
      const q = cityInput.value.trim();
      if(!q) return;
      const d = {name: q, query: q.split(' ').join('+')};
      selectDestination(d);
    }

    searchBtn.addEventListener('click', searchCity);
    cityInput.addEventListener('keydown', e=>{ if(e.key==='Enter') searchCity(); });

    // Initialize
    populateList();
    window.addEventListener('load', () => {
      selectDestination(DESTINATIONS[0]); // Paris by default
    });
  </script>
</body>
</html>
