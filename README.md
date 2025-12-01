<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Vitvisor — Álbum Multimedia Local</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 0; background:#121212; color:#eaeaea; }
    header { background:#000; color:#fff; padding:25px; text-align:center; font-size:30px; letter-spacing:1px; font-weight:bold; border-bottom:3px solid #cc0000; }

    nav { display:flex; justify-content:center; gap:40px; background:#1a1a1a; padding:18px; border-bottom:2px solid #cc0000; }
    nav a { color:#fff; text-decoration:none; font-weight:bold; font-size:20px; padding:8px 16px; border-radius:10px; transition:0.2s; }
    nav a:hover { background:#cc0000; }

    section { padding:35px; min-height:70vh; display:none; }
    section.active { display:block; animation:fadeIn 0.4s ease-in-out; }
    h2 { text-align:center; margin-bottom:25px; font-size:28px; color:#cc0000; }

    @keyframes fadeIn { from{opacity:0;} to{opacity:1;} }

    /* GRID */
    .grid { display:grid; grid-template-columns:repeat(auto-fill, minmax(230px, 1fr)); gap:28px; }

    /* MINIATURAS VIDEOS */
    .video-card { background:#1f1f1f; border-radius:14px; box-shadow:0 4px 12px rgba(0,0,0,0.25); overflow:hidden; cursor:pointer; transition:0.25s; }
    .video-card:hover { transform:scale(1.05); box-shadow:0 0 15px #cc0000; }
    .video-thumb { width:100%; height:150px; object-fit:cover; }
    .video-info { padding:12px; font-weight:bold; text-align:center; color:#fff; }

    /* MINIATURAS REVISTAS */
    .mag-card { background:#1f1f1f; border-radius:14px; box-shadow:0 4px 12px rgba(0,0,0,0.25); cursor:pointer; transition:0.25s; overflow:hidden; }
    .mag-card:hover { transform:scale(1.05); box-shadow:0 0 15px #cc0000; }
    .mag-cover { width:100%; height:280px; object-fit:cover; }
    .mag-info { padding:12px; font-weight:bold; text-align:center; color:#fff; }

    /* VISOR */
    .viewer { position:fixed; top:0; left:0; width:100%; height:100%; background:rgba(0,0,0,0.9); display:none; align-items:center; justify-content:center; z-index:9999; }
    .viewer-content { background:#000; padding:10px; border-radius:12px; max-width:90%; max-height:90%; }
    .viewer video, .viewer img { max-width:100%; max-height:100%; border-radius:10px; }
    .close-btn { position:absolute; top:25px; right:25px; font-size:40px; color:#fff; cursor:pointer; font-weight:bold; }
    .nav-btn { color:white; background:#cc0000; border:none; padding:10px 20px; border-radius:8px; cursor:pointer; font-size:20px; }
    /* Text Reader Style v2 */
  .text-page {
    font-family: "Georgia", serif;
    font-size: 1.15rem;
    line-height: 1.75;
    color: #222;
    background: #f5f1e8;
    padding: 40px;
    border-radius: 16px;
    margin: 0 auto;
    max-width: 750px;
    box-shadow: 0 4px 14px rgba(0,0,0,0.15);
  }
  .text-viewer-dark .text-page {
    background: #1e1e1e;
    color: #e2e2e2;
    box-shadow: 0 4px 20px rgba(0,0,0,0.4);
  }
</style>
</head>
<body>
<header>Álbum Multimedia Vitvisor (Local)</header>

<nav>
  <a onclick="showTab('videos')">Videos</a>
  <a onclick="showTab('revistas')">Revistas</a>
</nav>

<section id="videos" class="active">
  <h2>Videos</h2>
  <div class="grid" id="videoGrid"></div>
</section>

<section id="revistas">
  <h2>Revistas</h2>
  <div class="grid" id="magGrid"></div>
</section>

<div class="viewer" id="viewer">
  <span class="close-btn" onclick="closeViewer()">✕</span>
  <div class="viewer-content" id="viewerContent"></div>
</div>

<script>
/* CONFIGURACIÓN PARA USO LOCAL */
/* Cambia las rutas por carpetas reales de tu PC */

const VIDEOS = [
  { title:"Instalación Windows 10", thumb:"videos/thumb_win10.jpg", src:"videos/clideo_editor_0954606d19f84e6cae179f3981e5a873.mp4" },
  { title:"Montaje de PC", thumb:"videos/thumb_pc.jpg", src:"videos/pc.mp4" },
  { title:"Linux – Nivel Básico", thumb:"videos/thumb_linux.jpg", src:"videos/linux.mp4" },
  { title:"Curso Redes — Parte 1", thumb:"videos/thumb_redes.jpg", src:"videos/redes1.mp4" }
];

const MAGAZINES = [
  {
    title: "Guía Redes — Nº1",
    cover: "revistas/redes1/portada.jpg",
    pages: [
      { type: "text", content: "Introducción a redes: conceptos fundamentales, topologías y protocolos base." },
      { type: "text", content: "Configuración IP: direccionamiento, máscara, puerta de enlace y pruebas con ping." },
      { type: "text", content: "Seguridad básica: firewall, NAT, filtrado y contramedidas habituales." }
    ]
  },
  {
    title: "Hardware Profesional — Nº4",
    cover: "revistas/hardware4/portada.jpg",
    pages: [
      { type: "text", content: "Componentes internos: CPU, RAM, GPU, PSU y su impacto en rendimiento." },
      { type: "text", content: "Mantenimiento preventivo: limpieza, termal paste, monitorización térmica." },
      { type: "text", content: "Montaje óptimo: flujo de aire, cableado y compatibilidades críticas." }
    ]
  },
  {
    title: "Seguridad Informática — Nº7",
    cover: "revistas/seguridad7/portada.jpg",
    pages: [
      { type: "text", content: "Amenazas actuales: phishing, malware, ingeniería social." },
      { type: "text", content: "Buenas prácticas: contraseñas, MFA, backups y políticas de acceso." },
      { type: "text", content: "Red Team vs Blue Team: roles, estrategias y análisis de incidentes." }
    ]
  }
];

function showTab(tab){
  document.querySelectorAll('section').forEach(s => s.classList.remove('active'));
  document.getElementById(tab).classList.add('active');
}

function loadVideos(){
  const grid = document.getElementById("videoGrid");
  VIDEOS.forEach(v =>{
    const card = document.createElement("div");
    card.className = "video-card";
    card.innerHTML = `<img src="${v.thumb}" class="video-thumb"><div class="video-info">${v.title}</div>`;
    card.onclick = ()=> openVideo(v.src);
    grid.appendChild(card);
  });
}

function loadMagazines(){
  const grid = document.getElementById("magGrid");
  MAGAZINES.forEach(m =>{
    const card = document.createElement("div");
    card.className = "mag-card";
    card.innerHTML = `<img src="${m.cover}" class="mag-cover"><div class="mag-info">${m.title}</div>`;
    card.onclick = ()=> openMagazine(m.pages);
    grid.appendChild(card);
  });
}

function openVideo(src){
  const viewer = document.getElementById("viewer");
  const content = document.getElementById("viewerContent");
  content.innerHTML = `<video controls autoplay src="${src}"></video>`;
  viewer.style.display = "flex";
}

let currentPages = [];
let currentPage = 0;

function openMagazine(pages){
  currentPages = pages;
  currentPage = 0;
  updateMagazinePage();
  document.getElementById("viewer").style.display = "flex";
}

function updateMagazinePage(){
  const content = document.getElementById("viewerContent");
  content.innerHTML = `
    <img src="${currentPages[currentPage]}">
    <div style="text-align:center; margin-top:18px;">
      <button class="nav-btn" onclick="prevPage()">◀</button>
      <span style="color:white; margin:0 15px; font-size:20px;">${currentPage+1} / ${currentPages.length}</span>
      <button class="nav-btn" onclick="nextPage()">▶</button>
    </div>`;
}

function prevPage(){ if(currentPage>0){ currentPage--; updateMagazinePage(); } }
function nextPage(){ if(currentPage<currentPages.length-1){ currentPage++; updateMagazinePage(); } }

function closeViewer(){ document.getElementById("viewer").style.display = "none"; }

loadVideos();
loadMagazines();
</script>

</body>
</html>
