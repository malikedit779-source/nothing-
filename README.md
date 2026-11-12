
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="theme-color" content="#071d32">
  <meta name="description" content="Live weather updates, forecasts and maps across Pakistan.">
  <title>Mausam Pakistan 🇵🇰</title>

  <link rel="manifest" href="manifest.json">
  <link rel="icon" href="icon.svg" type="image/svg+xml">
  <link rel="stylesheet"
    href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">

  <style>
    :root {
      --bg:#071d32;
      --panel:#102c46;
      --panel2:#163953;
      --text:#f4f8ff;
      --muted:#a9bfd0;
      --green:#35e0b0;
      --blue:#42b9ff;
    }

    * { box-sizing:border-box; }

    body {
      margin:0;
      font-family:Arial,sans-serif;
      background:var(--bg);
      color:var(--text);
      min-height:100vh;
      transition:background .5s;
    }

    body.sunny {
      background:linear-gradient(140deg,#173c59,#071d32);
    }

    body.rainy {
      background:linear-gradient(140deg,#102c46,#061522);
    }

    body.cloudy {
      background:linear-gradient(140deg,#243e52,#071d32);
    }

    header {
      padding:18px;
      display:flex;
      justify-content:space-between;
      align-items:center;
      flex-wrap:wrap;
      gap:12px;
      background:#061727;
      border-bottom:1px solid #24445e;
    }

    h1 { margin:0;font-size:23px; }
    h1 span { color:var(--green); }

    button,select,input {
      border:0;
      border-radius:10px;
      padding:11px;
      font-size:14px;
    }

    button {
      background:var(--green);
      color:#062238;
      font-weight:bold;
      cursor:pointer;
    }

    button.secondary {
      background:#23445d;
      color:white;
    }

    main {
      max-width:1200px;
      margin:auto;
      padding:18px;
    }

    .toolbar {
      display:flex;
      gap:10px;
      flex-wrap:wrap;
      margin-bottom:18px;
    }

    input,select {
      background:#183850;
      color:white;
      min-width:150px;
    }

    .hero {
      background:linear-gradient(135deg,#164365,#102c46);
      border:1px solid #315775;
      border-radius:20px;
      padding:24px;
      display:grid;
      grid-template-columns:1fr 1fr;
      gap:15px;
      align-items:center;
      margin-bottom:18px;
    }

    .city { color:var(--green);font-size:19px; }
    .temperature { font-size:64px;font-weight:bold;margin:12px 0; }
    .condition { color:var(--muted);font-size:18px; }

    .weather-icon { font-size:65px; }

    .stats {
      display:grid;
      grid-template-columns:repeat(3,1fr);
      gap:12px;
      margin-bottom:20px;
    }

    .stat,.card {
      background:var(--panel);
      border:1px solid #25455f;
      border-radius:15px;
      padding:16px;
    }

    .stat small,.muted { color:var(--muted); }
    .stat strong { display:block;font-size:23px;margin-top:9px; }

    h2 { font-size:19px;margin:24px 0 12px; }

    #map {
      height:390px;
      border-radius:16px;
      border:1px solid #315775;
      z-index:1;
    }

    .province-grid,.forecast-grid,.city-grid {
      display:grid;
      grid-template-columns:repeat(auto-fit,minmax(145px,1fr));
      gap:12px;
    }

    .forecast-card {
      background:var(--panel);
      border-radius:13px;
      padding:14px;
      text-align:center;
      border:1px solid #25455f;
    }

    .forecast-card .emoji { font-size:27px;margin:8px; }

    .province-card {
      cursor:pointer;
      transition:.2s;
    }

    .province-card:hover {
      border-color:var(--green);
      transform:translateY(-2px);
    }

    .alert {
      border-left:4px solid #ffb547;
      background:#352e20;
      padding:13px;
      border-radius:9px;
      margin:9px 0;
    }

    .official {
      display:inline-block;
      color:var(--green);
      margin:8px 12px 8px 0;
    }

    footer {
      text-align:center;
      color:var(--muted);
      padding:25px;
    }

    .loading { color:var(--green); }

    @media(max-width:650px) {
      .hero { grid-template-columns:1fr; }
      .stats { grid-template-columns:1fr 1fr; }
      .temperature { font-size:53px; }
      #map { height:320px; }
    }
  </style>
</head>

<body>
<header>
  <h1>🇵🇰 <span>Mausam</span> Pakistan</h1>
  <div class="toolbar">
    <button class="secondary" onclick="toggleLanguage()" id="langBtn">اردو</button>
    <button class="secondary" onclick="enableNotifications()">🔔 Alerts</button>
    <button id="installBtn" hidden onclick="installApp()">📲 Install App</button>
  </div>
</header>

<main>
  <div class="toolbar">
    <input id="search" placeholder="Search Pakistan city...">
    <button onclick="searchCity()">Search</button>
    <select id="citySelect" onchange="selectCity(this.value)">
      <option value="Islamabad">Islamabad</option>
    </select>
    <button class="secondary" onclick="refreshWeather()">↻ Refresh</button>
  </div>

  <section class="hero">
    <div>
      <div class="city" id="cityName">Islamabad, Pakistan</div>
      <div class="temperature" id="temperature">--°C</div>
      <div class="condition" id="condition">Loading weather...</div>
      <p class="muted" id="updated">Please wait</p>
    </div>
    <div style="text-align:center">
      <div class="weather-icon" id="weatherIcon">🌤️</div>
      <p id="feelsLike">Feels like --°C</p>
      <p id="sunTimes" class="muted">Sunrise -- | Sunset --</p>
    </div>
  </section>

  <section class="stats">
    <div class="stat">
      <small>💧 Humidity</small>
      <strong id="humidity">--%</strong>
    </div>
    <div class="stat">
      <small>💨 Wind Speed</small>
      <strong id="wind">-- km/h</strong>
    </div>
    <div class="stat">
      <small>🌡️ Pressure</small>
      <strong id="pressure">-- hPa</strong>
    </div>
  </section>

  <h2>🗺️ Pakistan Weather Map</h2>
  <div id="map"></div>

  <h2>🏙️ Weather Across Pakistan</h2>
  <div class="city-grid" id="cityGrid"></div>

  <h2>📍 Province-wise Weather</h2>
  <div class="province-grid" id="provinceGrid"></div>

  <h2>📅 7-Day Forecast</h2>
  <div class="forecast-grid" id="forecast"></div>

  <h2>⚠️ Weather Alerts</h2>
  <p class="muted">
    These are app-generated weather threshold notices, not official government warnings.
  </p>
  <div id="alerts">
    <p class="muted">Loading weather notices...</p>
  </div>

  <h2>🏛️ Official Pakistan Weather Sources</h2>
  <div class="card">
    <p>Check government announcements directly from these sources:</p>
    <a class="official" target="_blank" rel="noopener" href="https://weather.gov.pk/">🌦️ Pakistan Meteorological Department</a>
    <a class="official" target="_blank" rel="noopener" href="https://www.ndma.gov.pk/ew">🚨 NDMA Early Warnings</a>
    <a class="official" target="_blank" rel="noopener" href="https://www.ndma.gov.pk/advisories">📢 NDMA Advisories</a>
  </div>
</main>

<footer>
  <p>🇵🇰 Mausam Pakistan</p>
  <p>Weather data by Open-Meteo. Always verify emergency warnings through official sources.</p>
</footer>

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<script>
const cities = [
  {name:"Islamabad",lat:33.6844,lon:73.0479,province:"Islamabad"},
  {name:"Karachi",lat:24.8607,lon:67.0011,province:"Sindh"},
  {name:"Lahore",lat:31.5204,lon:74.3587,province:"Punjab"},
  {name:"Peshawar",lat:34.0151,lon:71.5249,province:"Khyber Pakhtunkhwa"},
  {name:"Quetta",lat:30.1798,lon:66.9750,province:"Balochistan"},
  {name:"Multan",lat:30.1575,lon:71.5249,province:"Punjab"},
  {name:"Faisalabad",lat:31.4504,lon:73.1350,province:"Punjab"},
  {name:"Rawalpindi",lat:33.5651,lon:73.0169,province:"Punjab"},
  {name:"Hyderabad",lat:25.3960,lon:68.3578,province:"Sindh"},
  {name:"Sukkur",lat:27.7052,lon:68.8574,province:"Sindh"},
  {name:"Gwadar",lat:25.1264,lon:62.3225,province:"Balochistan"},
  {name:"Gilgit",lat:35.9208,lon:74.3144,province:"Gilgit-Baltistan"},
  {name:"Skardu",lat:35.2971,lon:75.6333,province:"Gilgit-Baltistan"},
  {name:"Muzaffarabad",lat:34.3700,lon:73.4711,province:"Azad Kashmir"},
  {name:"Abbottabad",lat:34.1688,lon:73.2215,province:"Khyber Pakhtunkhwa"},
  {name:"Sialkot",lat:32.4945,lon:74.5229,province:"Punjab"},
  {name:"Bahawalpur",lat:29.3956,lon:71.6836,province:"Punjab"},
  {name:"Dera Ismail Khan",lat:31.8320,lon:70.9024,province:"Khyber Pakhtunkhwa"}
];

const provinceNames = [
  "Punjab","Sindh","Khyber Pakhtunkhwa",
  "Balochistan","Islamabad","Gilgit-Baltistan","Azad Kashmir"
];

let currentCity = cities[0];
let weatherCache = {};
let map;
let markers = [];
let language = "en";
let installPrompt = null;

const $ = id => document.getElementById(id);

function weatherDescription(code) {
  if(code === 0) return ["Clear sky","☀️"];
  if([1,2].includes(code)) return ["Partly cloudy","🌤️"];
  if(code === 3) return ["Overcast","☁️"];
  if([45,48].includes(code)) return ["Fog","🌫️"];
  if([51,53,55,56,57].includes(code)) return ["Drizzle","🌦️"];
  if([61,63,65,66,67,80,81,82].includes(code)) return ["Rain","🌧️"];
  if([71,73,75,77,85,86].includes(code)) return ["Snow","❄️"];
  if([95,96,99].includes(code)) return ["Thunderstorm","⛈️"];
  return ["Weather conditions","🌡️"];
}

function setWeatherBackground(code) {
  document.body.classList.remove("sunny","rainy","cloudy");
  if(code === 0 || code === 1) document.body.classList.add("sunny");
  else if([61,63,65,80,81,82,95,96,99].includes(code))
    document.body.classList.add("rainy");
  else document.body.classList.add("cloudy");
}

async function getWeather(city) {
  const url = `https://api.open-meteo.com/v1/forecast?latitude=${city.lat}&longitude=${city.lon}&current=temperature_2m,relative_humidity_2m,apparent_temperature,is_day,precipitation,weather_code,wind_speed_10m,surface_pressure&daily=weather_code,temperature_2m_max,temperature_2m_min,sunrise,sunset&timezone=auto&forecast_days=7`;

  const response = await fetch(url);
  if(!response.ok) throw new Error("Weather service unavailable");
  const data = await response.json();

  weatherCache[city.name] = data;
  return data;
}

async function selectCity(name) {
  const city = cities.find(c => c.name === name);
  if(!city) return;
  currentCity = city;
  $("citySelect").value = city.name;
  await loadCity(city);
}

async function loadCity(city) {
  $("cityName").textContent = city.name + ", Pakistan";
  $("temperature").textContent = "--°C";
  $("condition").textContent = "Loading...";
  try {
    const data = await getWeather(city);
    const c = data.current;
    const [description,icon] = weatherDescription(c.weather_code);

    $("temperature").textContent = Math.round(c.temperature_2m) + "°C";
    $("condition").textContent = description;
    $("weatherIcon").textContent = icon;
    $("humidity").textContent = c.relative_humidity_2m + "%";
    $("wind").textContent = c.wind_speed_10m + " km/h";
    $("pressure").textContent = Math.round(c.surface_pressure) + " hPa";
    $("feelsLike").textContent = "Feels like " + Math.round(c.apparent_temperature) + "°C";
    $("updated").textContent = "Updated: " + new Date(c.time).toLocaleString();
    $("sunTimes").textContent =
      "Sunrise " + new Date(data.daily.sunrise[0]).toLocaleTimeString([], {hour:"2-digit",minute:"2-digit"}) +
      " | Sunset " + new Date(data.daily.sunset[0]).toLocaleTimeString([], {hour:"2-digit",minute:"2-digit"});

    setWeatherBackground(c.weather_code);
    showForecast(data);
    showAlerts();
    checkWeatherNotifications();
  } catch(error) {
    $("condition").textContent = "Could not load weather. Check your internet.";
    console.error(error);
  }
}

function showForecast(data) {
  $("forecast").innerHTML = "";
  data.daily.time.forEach((day,i) => {
    const [desc,icon] = weatherDescription(data.daily.weather_code[i]);
    const date = new Date(day+"T12:00:00");
    $("forecast").innerHTML += `
      <div class="forecast-card">
        <b>${date.toLocaleDateString("en",{weekday:"short"})}</b>
        <div class="emoji">${icon}</div>
        <small>${desc}</small>
        <p><b>${Math.round(data.daily.temperature_2m_max[i])}°</b> /
        ${Math.round(data.daily.temperature_2m_min[i])}°</p>
      </div>`;
  });
}

function showAlerts() {
  const data = weatherCache[currentCity.name];
  if(!data) return;
  const c = data.current;
  let notices = [];

  if([95,96,99].includes(c.weather_code))
    notices.push("Thunderstorm conditions detected. Check official warnings.");
  if(c.wind_speed_10m >= 50)
    notices.push("High wind threshold reached. Use caution and check official updates.");
  if(c.temperature_2m >= 40)
    notices.push("High temperature threshold reached. Stay hydrated and avoid prolonged heat exposure.");
  if(c.precipitation >= 10)
    notices.push("Heavy precipitation threshold reached. Check local weather updates.");

  $("alerts").innerHTML = notices.length
    ? notices.map(x=>`<div class="alert">⚠️ ${x}</div>`).join("")
    : '<p class="muted">No app-generated weather threshold notices for this city right now.</p>';
}

function setupCitySelect() {
  $("citySelect").innerHTML = cities.map(c =>
    `<option value="${c.name}">${c.name}</option>`
  ).join("");
}

function renderCityCards() {
  $("cityGrid").innerHTML = cities.map(c => `
    <div class="card province-card" onclick="selectCity('${c.name}')">
      <b>${c.name}</b>
      <p class="muted">${c.province}</p>
      <strong id="temp-${c.name.replaceAll(" ","-")}">Loading...</strong>
    </div>
  `).join("");
}

async function loadAllCityTemperatures() {
  for(const city of cities) {
    try {
      const data = await getWeather(city);
      const [desc,icon] = weatherDescription(data.current.weather_code);
      const el = $("temp-"+city.name.replaceAll(" ","-"));
      if(el) el.textContent = `${icon} ${Math.round(data.current.temperature_2m)}°C`;
    } catch(e) {
      const el = $("temp-"+city.name.replaceAll(" ","-"));
      if(el) el.textContent = "Unavailable";
    }
  }
  showProvinceCards();
  updateMapMarkers();
}

function showProvinceCards() {
  $("provinceGrid").innerHTML = provinceNames.map(name => {
    const members = cities.filter(c => c.province === name);
    const values = members
      .map(c => weatherCache[c.name]?.current?.temperature_2m)
      .filter(v => Number.isFinite(v));
    const avg = values.length
      ? Math.round(values.reduce((a,b)=>a+b,0)/values.length)+"°C average"
      : "Loading...";
    return `<div class="card province-card">
      <b>${name}</b><p class="muted">${members.length} listed cities</p>
      <strong>${avg}</strong>
    </div>`;
  }).join("");
}

function initMap() {
  map = L.map("map").setView([30.5,69.3],5);

  L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
    attribution:"&copy; OpenStreetMap contributors",
    maxZoom:18
  }).addTo(map);

  loadProvinceBoundaries();

  cities.forEach(city => {
    const marker = L.marker([city.lat,city.lon]).addTo(map);
    marker.bindPopup(`<b>${city.name}</b><br>${city.province}`);
    marker.on("click",()=>selectCity(city.name));
    markers.push({city,marker});
  });
}

async function loadProvinceBoundaries() {
  const url =
    "https://services1.arcgis.com/HmwnYiJTBZ4UkySc/ArcGIS/rest/services/" +
    "Enrollment_Cluster_Analysis_Map_WFL1/FeatureServer/1/query" +
    "?where=1%3D1&outFields=ADM1_EN&returnGeometry=true&outSR=4326&f=geojson";

  try {
    const response = await fetch(url);
    if(!response.ok) throw new Error("Boundary service unavailable");
    const geoData = await response.json();

    L.geoJSON(geoData, {
      style: {
        color:"#35e0b0",
        weight:2,
        opacity:.9,
        fillColor:"#35e0b0",
        fillOpacity:.07
      },
      onEachFeature:(feature,layer)=>{
        const name = feature.properties?.ADM1_EN || "Region";
        layer.bindPopup("<b>🇵🇰 "+name+"</b><br>Pakistan");
      }
    }).addTo(map);
  } catch(error) {
    console.warn("Province boundaries could not load:",error);
  }
}

function updateMapMarkers() {
  markers.forEach(({city,marker})=>{
    const data = weatherCache[city.name];
    if(data) {
      const [desc,icon] = weatherDescription(data.current.weather_code);
      marker.setPopupContent(
        `<b>${city.name}</b><br>${icon} ${Math.round(data.current.temperature_2m)}°C<br>${desc}`
      );
    }
  });
}

async function searchCity() {
  const query = $("search").value.trim().toLowerCase();
  if(!query) return;

  const match = cities.find(c => c.name.toLowerCase() === query);
  if(match) {
    await selectCity(match.name);
    return;
  }

  const partial = cities.find(c => c.name.toLowerCase().includes(query));
  if(partial) {
    await selectCity(partial.name);
    return;
  }

  alert("City not found in the Pakistan city list.");
}

async function refreshWeather() {
  await loadCity(currentCity);
  await loadAllCityTemperatures();
}

function toggleLanguage() {
  language = language === "en" ? "ur" : "en";
  $("langBtn").textContent = language === "en" ? "اردو" : "English";
  document.documentElement.lang = language;
  $("search").placeholder = language === "en"
    ? "Search Pakistan city..."
    : "پاکستان کا شہر تلاش کریں...";
}

async function enableNotifications() {
  if(!("Notification" in window)) {
    alert("Notifications are not supported in this browser.");
    return;
  }

  const permission = await Notification.requestPermission();
  if(permission === "granted") {
    alert("Notifications enabled. Keep the app open for weather checks.");
    checkWeatherNotifications();
  } else {
    alert("Notification permission was not granted.");
  }
}

function checkWeatherNotifications() {
  if(!("Notification" in window) || Notification.permission !== "granted") return;

  cities.forEach(city => {
    const data = weatherCache[city.name];
    if(!data) return;

    const c = data.current;
    let warning = "";

    if([95,96,99].includes(c.weather_code))
      warning = "Thunderstorm conditions detected.";
    else if(c.wind_speed_10m >= 50)
      warning = "High wind threshold reached.";
    else if(c.temperature_2m >= 40)
      warning = "High temperature threshold reached.";

    if(!warning) return;

    const key = "weather-alert-"+city.name;
    const signature = warning+"-"+c.time;

    if(localStorage.getItem(key) === signature) return;

    new Notification("Mausam Pakistan 🇵🇰", {
      body:city.name+": "+warning
    });

    localStorage.setItem(key,signature);
  });
}

if("serviceWorker" in navigator) {
  window.addEventListener("load",()=>{
    navigator.serviceWorker.register("./sw.js")
      .catch(error=>console.error("Service worker:",error));
  });
}

window.addEventListener("beforeinstallprompt",event=>{
  event.preventDefault();
  installPrompt = event;
  $("installBtn").hidden = false;
});

async function installApp() {
  if(!installPrompt) {
    alert("Open your browser menu and select Add to Home Screen.");
    return;
  }

  installPrompt.prompt();
  await installPrompt.userChoice;
  installPrompt = null;
  $("installBtn").hidden = true;
}

window.addEventListener("appinstalled",()=>{
  $("installBtn").hidden = true;
});

async function startApp() {
  setupCitySelect();
  renderCityCards();
  initMap();
  await loadCity(currentCity);
  await loadAllCityTemperatures();
}

startApp();
</script>
</body>
</html>

