<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  let url = "https://docs.google.com/spreadsheets/d/e/2PACX-1vRDjBBwREsDHYRK2H6gqEWJn1_c_rqp9D_9ox_0V_bZsjr-VhrKTWRWWt7hjzmbKj4gnHw5ttb1dcL_/pub?gid=0&single=true&output=csv";
  let datos = [];
  let filtroBusqueda = "";
  let filtroTipo = "todas";
  let filtroCategoria = "";
  let orden = "desc";

  // Array completo de botones tal como en tu código original:
  let botones = [
    { url: "https://cdn.parlamentia.newtral.es/images/categories/educacion.svg", texto: "Educación" },
    { url: "https://cdn.parlamentia.newtral.es/images/categories/transicion-ecologica.svg", texto: "Transición Ecológica" },
    { url: "https://cdn.parlamentia.newtral.es/images/categories/interior-y-defensa.svg", texto: "Interior y Defensa" },
    { url: "https://cdn.parlamentia.newtral.es/images/categories/derechos-fundamentales-y-regeneracion-democratica.svg", texto: "Derechos fundamentales y Regeneración democrática" },
    { url: "https://cdn.parlamentia.newtral.es/images/categories/igualdad.svg", texto: "Igualdad" },
    { url: "https://cdn.parlamentia.newtral.es/images/categories/economia.svg", texto: "Economía" },
    { url: "https://cdn.parlamentia.newtral.es/images/categories/agricultura-pesca-y-alimentacion.svg", texto: "Agricultura, Pesca y Alimentación" },
    { url: "https://cdn.parlamentia.newtral.es/images/categories/politica-territorial-y-administracion-publica.svg", texto: "Política Territorial y Administración Pública" },
    { url: "https://cdn.parlamentia.newtral.es/images/categories/trabajo-inclusion-y-migraciones.svg", texto: "Trabajo, Inclusión y Migraciones" },
    { url: "https://cdn.parlamentia.newtral.es/images/categories/internacional-y-politica-exterior.svg", texto: "Internacional y Política Exterior" },
    { url: "https://cdn.parlamentia.newtral.es/images/categories/hacienda.svg", texto: "Hacienda" },
    { url: "https://cdn.parlamentia.newtral.es/images/categories/movilidad-y-vivienda.svg", texto: "Movilidad y Vivienda" },
    { url: "https://cdn.parlamentia.newtral.es/images/categories/sanidad-y-consumo.svg", texto: "Sanidad y Consumo" },
    { url: "https://cdn.parlamentia.newtral.es/images/categories/ciencia-y-tecnologia.svg", texto: "Ciencia y Tecnología" },
    { url: "https://cdn.parlamentia.newtral.es/images/categories/politica-social.svg", texto: "Política Social" },
    { url: "https://cdn.parlamentia.newtral.es/images/categories/cultura-y-deporte.svg", texto: "Cultura y Deporte" },
    { url: "https://cdn.parlamentia.newtral.es/images/categories/industria-turismo-y-comercio.svg", texto: "Industria, Turismo y Comercio" },
    { url: "https://cdn.parlamentia.newtral.es/images/categories/justicia.svg", texto: "Justicia" },
    { url: "https://cdn.parlamentia.newtral.es/images/categories/otros.svg", texto: "Otros" },
  ];

  // Carga datos CSV al montar
  onMount(async () => {
    const data = await d3.csv(url);
    datos = data.map((d, index) => ({
      id: `item-${index}`,
      titulo: d.TITULO,
      directiva: d.DIRECTIVA,
      fecha: d.FECHA_LIMITE,
      dias: d.DIAS,
      enlace: d.ENLACE,
      nombre: d.NOMBRE,
      categoria: d.CATEGORIA,
      foto: d.URL,
    }));
  });

  const normalizeText = text =>
    text.normalize('NFD').replace(/[\u0300-\u036f]/g, "").toLowerCase();

  function calculateDaysDifference(fechaString) {
    if (!fechaString) return 0;
    const today = new Date();
    const [day, month, year] = fechaString.split("/").map(Number);
    const fecha = new Date(year, month - 1, day);
    today.setHours(0, 0, 0, 0);
    return Math.ceil((fecha - today) / (1000 * 60 * 60 * 24));
  }

  function getDaysClass(dias) {
    if (dias > 60) return "positivo";
    if (dias >= 0) return "cercano";
    return "negativo";
  }

  function formatNumber(number) {
    return number.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ".");
  }

  function filtrarTarjetasPorCategoria(url) {
    filtroCategoria = url;
  }

  function verTodasTarjetas() {
    filtroCategoria = "";
  }

  function ordenarTarjetas(nuevoOrden) {
    orden = nuevoOrden;
  }

  // Tooltip
  let tooltip = { visible: false, x: 0, y: 0, contenido: "" };
  function mostrarTooltip(event, contenido) {
    tooltip = {
      visible: true,
      x: event.pageX + 10,
      y: event.pageY + 10,
      contenido
    };
  }
  function ocultarTooltip() {
    tooltip.visible = false;
  }

  // Tarjetas filtradas y ordenadas
  $: tarjetasFiltradas = datos
    .map(d => {
      const diasRestantes = calculateDaysDifference(d.fecha);
      return {
        ...d,
        diasRestantes,
        className: getDaysClass(diasRestantes)
      };
    })
    .filter(d => {
      const txt = normalizeText(`${d.titulo} ${d.directiva} ${d.fecha}`);
      if (filtroBusqueda && !txt.includes(normalizeText(filtroBusqueda))) return false;
      if (filtroTipo === "retrasadas" && d.className !== "negativo") return false;
      if (filtroTipo === "poco-tiempo" && d.className !== "cercano") return false;
      if (filtroTipo === "con-margen" && d.className !== "positivo") return false;
      if (filtroCategoria && d.foto !== filtroCategoria) return false;
      return true;
    })
    .sort((a, b) => {
      return orden === "asc" ? a.diasRestantes - b.diasRestantes : b.diasRestantes - a.diasRestantes;
    });
</script>

<style>
  body {
    font-family: 'Helvetica Neue', sans-serif;
    background-color: #f8f9fa;
    margin: 0; padding: 0;
    color: #343a40;
  }
  .card-img-container {
    position: absolute;
    top: 10px;
    left: 10px;
    width: 40px; height: 40px;
    overflow: hidden;
  }
  .card-img {
    width: 100%;
    height: 100%;
    object-fit: cover;
    border-radius: 50%;
  }
  .card-container {
    display: flex;
    flex-wrap: wrap;
    gap: 16px;
    justify-content: center;
    padding: 20px;
  }
  .card {
    width: 250px;
    box-shadow: 0 4px 6px rgba(0,0,0,0.1);
    border-radius: 12px;
    overflow: hidden;
    background: #fff;
    transition: transform 0.3s ease, box-shadow 0.3s ease;
    position: relative;
  }
  .card:hover {
    transform: translateY(-5px);
    box-shadow: 0 8px 16px rgba(0,0,0,0.2);
  }
  .card.positivo {
    border-left: 6px solid #28a745;
    background-color: #d4f8e8;
  }
  .card.cercano {
    border-left: 6px solid #ffc107;
    background-color: #fff3cd;
  }
  .card.negativo {
    border-left: 6px solid #dc3545;
    background-color: #f8d7da;
  }
  .dias-restantes {
    text-align: right;
    padding: 16px;
    font-size: 1.5em;
    font-weight: bold;
    color: #495057;
  }
  .numero-dias {
    display: block;
    font-size: 2em;
    color: #212529;
  }
  .texto-dias {
    font-size: 0.5em;
    color: #6c757d;
  }
  .contenido {
    padding: 16px;
  }
  .titulo {
    font-size: 1.2em;
    font-weight: bold;
    margin-bottom: 10px;
    color: #343a40;
  }
  .directiva {
    font-size: 1em;
    margin-bottom: 8px;
    color: black;
  }
  .directiva a {
    text-decoration: underline;
    color: inherit;
  }
  .directiva a:hover {
    text-decoration: underline;
  }
  .fecha {
    font-size: 0.9em;
    color: #6c757d;
  }
  .tooltip {
    position: absolute;
    padding: 12px 16px;
    background-color: #ffffff;
    color: #343a40;
    border: 2px solid #aaaaaa;
    border-radius: 8px;
    font-size: 0.95em;
    pointer-events: none;
    box-shadow: 0 4px 8px rgba(0,0,0,0.2);
    transition: opacity 0.2s ease, transform 0.2s ease;
    z-index: 1000;
    max-width: 250px;
  }
  .search-bar {
    display: flex;
    justify-content: center;
    gap: 20px;
    margin-bottom: 5px;
    padding: 10px;
    flex-wrap: wrap;
  }
  .search-bar input[type="text"] {
    width: 100%;
    padding: 10px;
    border: 2px solid #ced4da;
    border-radius: 50px;
    font-size: 1em;
    transition: border-color 0.3s ease, box-shadow 0.3s ease;
  }
  .search-bar input[type="text"]:focus {
    border-color: #01f3b3;
    box-shadow: 0 0 4px rgba(1,243,179,0.5);
    outline: none;
  }
  .search-bar select,
  .search-bar button {
    padding: 10px;
    font-size: 1em;
    border: 2px solid #aaaaaa;
    background-color: #aaaaaa;
    color: white;
    border-radius: 50px;
    cursor: pointer;
    transition: background-color 0.3s ease, transform 0.2s ease;
  }
  .search-bar button:hover, .search-bar select:hover {
    background-color: #dddddd;
  }
  .search-bar button:active, .search-bar select:active {
    transform: scale(0.95);
  }
  .filter-buttons {
    display: flex;
    margin-bottom: 20px;
    margin-top: 5px;
    gap: 20px;
    padding: 10px 0;
    overflow-x: auto;
  }
  .btn-filter {
    display: flex;
    align-items: center;
    gap: 8px;
    padding: 10px 15px;
    font-size: 1em;
    border: 2px solid black;
    border-radius: 50px;
    background-color: white;
    color: black;
    cursor: pointer;
    transition: all 0.3s ease;
    white-space: nowrap;
  }
  .btn-filter img {
    width: 20px;
    height: 20px;
  }
  .btn-filter:hover {
    background-color: #f0f0f0;
  }
  .btn-filter:active {
    transform: scale(0.95);
    background-color: #e0e0e0;
  }
  .reset-button {
    font-weight: bold;
  }
  @media(max-width:768px) {
    .btn-filter {
      width: 100%;
      text-align: center;
    }
    .search-bar {
      flex-direction: column;
      gap: 10px;
      align-items: stretch;
      padding: 10px;
    }
    .search-bar input[type="text"],
    .search-bar select,
    .search-bar button {
      width: calc(100% - 20px);
      margin: 0 auto;
      padding: 12px;
    }
    .card {
      width: 100%;
    }
  }
</style>

<div class="search-bar">
  <input type="text" bind:value={filtroBusqueda} placeholder="Buscar por título, directiva o fecha..." />
  <select bind:value={filtroTipo}>
    <option value="todas">Todas las tarjetas</option>
    <option value="retrasadas">Caducadas</option>
    <option value="poco-tiempo">Poco tiempo</option>
    <option value="con-margen">Con margen</option>
  </select>
  <button on:click={() => ordenarTarjetas('desc')}>Días restantes ↓</button>
  <button on:click={() => ordenarTarjetas('asc')}>Días restantes ↑</button>
</div>

<div class="filter-buttons">
  <button class="btn-filter reset-button" on:click={verTodasTarjetas}>Mostrar todas</button>
  {#each botones as btn}
    <button class="btn-filter" on:click={() => filtrarTarjetasPorCategoria(btn.url)}>
      <img src={btn.url} alt={btn.texto} />
      <span>{btn.texto}</span>
    </button>
  {/each}
</div>

<div class="card-container">
  {#each tarjetasFiltradas as item (item.id)}
    <div
      class="card {item.className}"
      data-nombre={item.nombre}
      data-foto={item.foto}
      on:mouseover={e => mostrarTooltip(e, item.nombre)}
      on:mousemove={e => mostrarTooltip(e, item.nombre)}
      on:mouseleave={ocultarTooltip}
    >
      <div class="card-img-container">
        <img src={item.foto} alt="Imagen" class="card-img" />
      </div>
      <div class="dias-restantes">
        <span class="numero-dias">{formatNumber(Math.abs(item.diasRestantes))}</span>
        <div class="texto-dias">{item.diasRestantes < 0 ? "días caducada" : "días de margen"}</div>
      </div>
      <div class="contenido">
        <div class="titulo">{item.titulo}</div>
        <div class="directiva">
          Directiva: <a href={item.enlace} target="_blank" rel="noopener noreferrer">{item.directiva}</a>
        </div>
        <div class="fecha">Fecha límite de transposición: {item.fecha}</div>
      </div>
    </div>
  {/each}

  {#if tarjetasFiltradas.length === 0}
    <p>No se encontraron resultados.</p>
  {/if}
</div>

{#if tooltip.visible}
  <div
    class="tooltip show"
    style="position: absolute; left: {tooltip.x}px; top: {tooltip.y}px; pointer-events:none;"
  >
    <strong>Título de la Directiva: </strong><br />{tooltip.contenido}
  </div>
{/if}