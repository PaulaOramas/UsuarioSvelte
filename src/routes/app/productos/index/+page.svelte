<script>
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';

  let productos = [];
  let categorias = [];
  let categoriaSeleccionada = '';
  let busqueda = '';
  let mensajeError = '';
  let darkMode = false;
  let alerta = '';
  let alertaTimeout;
  let mostrarContenido = false;

  // Modo oscuro y carga inicial
  onMount(() => {
    setTimeout(() => {
      mostrarContenido = true;
      document.body.style.overflow = "auto";
    }, 1000);

    darkMode = localStorage.getItem('dark-mode') === 'enabled';
    setDarkMode(darkMode);

    cargarCategorias();
    cargarProductos();
  });

  function setDarkMode(enabled) {
    darkMode = enabled;
    if (darkMode) {
      document.body.classList.add('dark-mode');
      localStorage.setItem('dark-mode', 'enabled');
    } else {
      document.body.classList.remove('dark-mode');
      localStorage.setItem('dark-mode', 'disabled');
    }
  }

  async function cargarCategorias() {
    try {
      const res = await fetch('https://eltragolocorest.runasp.net/api/Categoria');
      if (!res.ok) throw new Error('Error en categorías');
      categorias = await res.json();
    } catch (e) {
      mensajeError = 'No se pudieron cargar las categorías';
    }
  }

  async function cargarProductos() {
    try {
      const res = await fetch('https://eltragolocorest.runasp.net/api/Producto');
      if (!res.ok) throw new Error('Error al obtener productos');
      productos = await res.json();
    } catch (e) {
      mensajeError = 'No se pudieron cargar los productos';
    }
  }

  function filtrarProductos() {
    return productos.filter(p => {
      const coincideCategoria = !categoriaSeleccionada || p.CAT_NOMBRE === categoriaSeleccionada;
      const coincideBusqueda = p.PROD_NOMBRE.toLowerCase().includes(busqueda.toLowerCase());
      return coincideCategoria && coincideBusqueda;
    });
  }

  function agregarAlCarrito(producto) {
    let carritoActual = JSON.parse(localStorage.getItem("carrito")) || [];
    let existente = carritoActual.find(p => p.PROD_ID === producto.PROD_ID);
    if (existente) {
      existente.Cantidad = (existente.Cantidad || 1) + 1;
    } else {
      carritoActual.push({
        PROD_ID: parseInt(producto.PROD_ID),
        PROD_NOMBRE: producto.PROD_NOMBRE,
        PROD_PRECIO: parseFloat(producto.PROD_PRECIO),
        PROD_IMG: producto.PROD_IMG,
        Cantidad: 1
      });
    }
    localStorage.setItem("carrito", JSON.stringify(carritoActual));
    carrito.set(carritoActual); // Actualiza el store de Svelte
    mostrarAlerta("Producto agregado al carrito");
    // Si quieres redirigir al carrito, descomenta la siguiente línea:
    // goto('/app/carrrito');
  }

  function mostrarAlerta(msg, duracion = 3000) {
    alerta = msg;
    clearTimeout(alertaTimeout);
    alertaTimeout = setTimeout(() => alerta = '', duracion);
  }
</script>

<!-- Bootstrap y CSS globales -->
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet" />
<link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@400;600;700&family=Pacifico&display=swap" rel="stylesheet" />
<link href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.5/font/bootstrap-icons.css" rel="stylesheet" />

<style>
  .hero-video-container {
    position: relative;
    width: 100vw;
    height: 100vh;
    overflow: hidden;
    display: flex;
    align-items: center;
    justify-content: center;
    background: #222;
  }
  .hero-video {
    width: 100vw;
    height: 100vh;
    object-fit: cover;
    object-position: center;
    filter: brightness(0.85);
  }
  .hero-overlay {
    position: absolute;
    top: 0; left: 0; right: 0; bottom: 0;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    color: #fff;
    background: rgba(0,0,0,0.25);
    text-shadow: 0 2px 8px #0008;
    z-index: 2;
  }
  .hero-title {
    font-family: 'Pacifico', cursive;
    font-size: 3rem;
    margin-bottom: 0.5em;
    letter-spacing: 2px;
  }
  .hero-subtitle {
    font-size: 1.5rem;
    font-family: 'Montserrat', sans-serif;
    font-weight: 600;
  }
  #main-content {
    margin-top: -5vh;
    background: #fff;
    border-radius: 2rem 2rem 0 0;
    box-shadow: 0 -4px 24px #0002;
    padding-top: 2rem;
    min-height: 80vh;
  }
  @media (max-width: 768px) {
    .hero-title { font-size: 2rem; }
    .hero-subtitle { font-size: 1rem; }
    #main-content { border-radius: 1rem 1rem 0 0; }
  }

  #main-content { display: none; }
  #main-content.visible { display: block; }
  #puzzle-overlay {
    position: fixed;
    inset: 0;
    background: url("/imagenes/coqtel.gif") center center no-repeat;
    background-size: cover;
    z-index: 11000;
    user-select: none;
    pointer-events: none;
  }
  .video-inicio { width: 100%; max-height: 350px; object-fit: cover; border-radius: 1rem; }
  .btn-morado { background: #6f42c1; color: #fff; }
  .btn-morado:hover { background: #5936a6; }
</style>

{#if !mostrarContenido}
  <div id="puzzle-overlay" aria-hidden="true" role="presentation"></div>
{/if}

{#if mostrarContenido}
  <!-- HERO VIDEO -->
  <section class="hero-video-container">
    <video class="hero-video" autoplay muted loop playsinline preload="auto">
      <source src="/imagenes/GIN COCKTAIL B ROLL.mp4" type="video/mp4" />
      Tu navegador no soporta video HTML5.
    </video>
    <div class="hero-overlay">
      <h1 class="hero-title">Bienvenido a El Trago Loco</h1>
      <p class="hero-subtitle">Descubre los mejores cócteles y productos exclusivos</p>
      <a href="#main-content" class="btn btn-morado btn-lg mt-4 shadow">Ver catálogo</a>
    </div>
  </section>

  <div id="main-content" class="visible">
    <div class="d-flex justify-content-end align-items-center gap-2 mt-2 me-3">
      <button
        class="btn-modo-oscuro"
        aria-label="Cambiar modo oscuro/claro"
        on:click={() => setDarkMode(!darkMode)}
        title={darkMode ? 'Modo claro' : 'Modo oscuro'}
      >
        {#if darkMode}
          <i class="bi bi-sun-fill"></i>
        {:else}
          <i class="bi bi-moon-stars-fill"></i>
        {/if}
      </button>
    </div>


    <div class="filtros-container d-flex gap-2 my-3">
      <select
        class="form-select"
        bind:value={categoriaSeleccionada}
        aria-label="Filtrar productos por categoría"
      >
        <option value="">Todas las categorías</option>
        {#each categorias as cat}
          <option value={cat.CAT_NOMBRE}>{cat.CAT_NOMBRE}</option>
        {/each}
      </select>
      <input
        class="form-control"
        type="search"
        placeholder="Buscar producto por nombre..."
        aria-label="Buscar producto por nombre"
        bind:value={busqueda}
      />
    </div>

    <main class="container">
      <h2 class="catalogo-titulo mb-4">Catálogo de Productos</h2>
      {#if mensajeError}
        <div class="alert alert-danger">{mensajeError}</div>
      {:else}
        <div class="row g-4 justify-content-center" id="contenedorProductos">
          {#if filtrarProductos().length === 0}
            <p class="text-center text-muted">No se encontraron productos.</p>
          {:else}
            {#each filtrarProductos() as item}
              <div class="col-md-3">
                <article class="card h-100 shadow-sm border-0">
                  <img src={item.PROD_IMG} alt={item.PROD_NOMBRE} class="img-fluid rounded" loading="lazy" style="max-height: 300px;" />
                  <div class="card-body text-center">
                    <h5 class="card-title text-dark fw-semibold">{item.PROD_NOMBRE}</h5>
                    <p class="card-text fw-bold">${parseFloat(item.PROD_PRECIO).toFixed(2)}</p>
                    <p class="text-muted small">Stock: {item.PROD_STOCK} unidades</p>
                    <a href={`/app/productos/detalle?id=${item.PROD_ID}`} class="btn btn-outline-dark btn-sm w-100 mb-2">Ver más</a>
                    <button on:click={() => agregarAlCarrito(item)} class="btn btn-morado btn-sm w-100 fw-semibold">🛒 Agregar al carrito</button>
                  </div>
                </article>
              </div>
            {/each}
          {/if}
        </div>
      {/if}
    </main>

    {#if alerta}
      <div class="position-fixed bottom-0 end-0 m-3 alert alert-success" style="z-index: 1050; min-width: 250px">
        {alerta}
      </div>
    {/if}
  </div>
{/if}