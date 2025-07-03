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
  let cargando = true;
  let paginaActual = 1; // Página inicial
  const productosPorPagina = 6; // Número de productos por página

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
    cargando = true;
    try {
      const res = await fetch('https://eltragolocorest.runasp.net/api/Producto');
      if (!res.ok) throw new Error('Error al obtener productos');
      productos = await res.json();
    } catch (e) {
      mensajeError = 'No se pudieron cargar los productos';
    } finally {
      cargando = false;
    }
  }

  function filtrarProductos() {
    return productos.filter(p => {
      // Si la categoría seleccionada está vacía, mostrar todos
      if (!categoriaSeleccionada) return p.PROD_NOMBRE.toLowerCase().includes(busqueda.toLowerCase());
      // Si el producto tiene CAT_NOMBRE, filtra por nombre
      if (p.CAT_NOMBRE) {
        return p.CAT_NOMBRE === categoriaSeleccionada &&
          p.PROD_NOMBRE.toLowerCase().includes(busqueda.toLowerCase());
      }
      // Si el producto tiene CAT_ID, filtra por id (opcional, si usas id en el select)
      if (p.CAT_ID) {
        return p.CAT_ID == categoriaSeleccionada &&
          p.PROD_NOMBRE.toLowerCase().includes(busqueda.toLowerCase());
      }
      return false;
    });
  }

  function agregarAlCarrito(producto) {
    if (producto.PROD_STOCK <= 0) {
      mostrarAlerta("Este producto no tiene stock disponible", 3000);
      return;
    }

    let carrito = JSON.parse(localStorage.getItem("carrito")) || [];
    let existente = carrito.find(p => p.PROD_ID === producto.PROD_ID);
    if (existente) {
      existente.Cantidad = (existente.Cantidad || 1) + 1;
    } else {
      carrito.push({
        PROD_ID: parseInt(producto.PROD_ID),
        PROD_NOMBRE: producto.PROD_NOMBRE,
        PROD_PRECIO: parseFloat(producto.PROD_PRECIO),
        PROD_IMG: producto.PROD_IMG,
        Cantidad: 1
      });
    }
    localStorage.setItem("carrito", JSON.stringify(carrito));
    mostrarAlerta("Producto agregado al carrito");
  }

  function mostrarAlerta(msg, duracion = 3000) {
    alerta = msg;
    clearTimeout(alertaTimeout);
    alertaTimeout = setTimeout(() => alerta = '', duracion);
  }

  function obtenerProductosPaginados() {
    const inicio = (paginaActual - 1) * productosPorPagina;
    const fin = inicio + productosPorPagina;
    return filtrarProductos().slice(inicio, fin);
  }
</script>


<style>
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
  .btn-modo-oscuro {
  background: #fff;
  border: 2px solid #4B2563;
  color: #4B2563;
  border-radius: 50%;
  width: 44px;
  height: 44px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.5rem;
  box-shadow: 0 2px 8px #4b256320;
  transition: background 0.2s, color 0.2s, border 0.2s;
  cursor: pointer;
}
.btn-modo-oscuro:hover {
  background: #4B2563;
  color: #fff;
  border-color: #a44b5a;
}

.hero-video {
  width: 100vw;
  height: 100vh;
  min-height: 100vh;
  max-height: 100vh;
  overflow: hidden;
  border-radius: 0;
  position: relative;
  margin-left: calc(-1 * (100vw - 100%) / 2);
  margin-right: calc(-1 * (100vw - 100%) / 2);
  background: #222;
}

.video-hero {
  width: 100vw;
  height: 100vh;
  min-height: 100vh;
  max-height: 100vh;
  object-fit: cover;
  display: block;
  filter: brightness(0.7);
}

.hero-overlay {
  z-index: 2;
  pointer-events: none;
  padding: 2rem 1rem;
  background: linear-gradient(180deg,rgba(0,0,0,0.45) 0%,rgba(0,0,0,0.15) 100%);
  height: 100vh;
  min-height: 100vh;
  max-height: 100vh;
}

.hero-title {
  font-family: 'Pacifico', cursive;
  font-size: 2.5rem;
  text-shadow: 0 4px 24px #000a, 0 1px 0 #fff4;
  letter-spacing: 2px;
  color: #fff;
}

.hero-subtitle {
  font-size: 1.2rem;
  font-weight: 500;
  text-shadow: 0 2px 8px #0008;
  color: #fff;
}

.btn-lg {
  padding: 0.75rem 1.5rem;
  font-size: 1.125rem;
  border-radius: 0.5rem;
}

@media (max-width: 768px) {
  .hero-video, .video-hero, .hero-overlay {
    min-height: 60vh;
    height: 60vh;
    max-height: 70vh;
    border-radius: 0;
  }
  .hero-title {
    font-size: 1.4rem;
  }
  .hero-subtitle {
    font-size: 1rem;
  }
}
</style>

<!-- Hero con video de fondo y overlay a pantalla completa -->
<section class="hero-video position-relative mb-0">
  <video class="video-hero" autoplay muted loop playsinline preload="auto">
    <source src="/imagenes/GIN COCKTAIL B ROLL.mp4" type="video/mp4" />
    Tu navegador no soporta video HTML5.
  </video>
  <div class="hero-overlay position-absolute top-0 start-0 w-100 h-100 d-flex flex-column justify-content-center align-items-center">
    <h1 class="hero-title">Bienvenido a El Trago Loco</h1>
    <p class="hero-subtitle">Descubre los mejores cócteles y productos exclusivos</p>
    <a href="#main-content" class="btn btn-morado btn-lg mt-4 shadow" style="pointer-events:auto;scroll-behavior:smooth;">Ver catálogo</a>
  </div>
</section>

{#if !mostrarContenido}
  <div id="puzzle-overlay" aria-hidden="true" role="presentation"></div>
{/if}

{#if mostrarContenido}
  <!-- Contenido principal -->
  <div id="main-content" class="visible fondo-crema">
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
      <h2>Catálogo de Productos</h2>
      {#if mensajeError}
        <div class="alert alert-danger">{mensajeError}</div>
      {:else if cargando}
        <div class="text-center my-5">
          <div class="spinner-border text-secondary mb-2" role="status"></div>
          <div>Cargando productos...</div>
        </div>
      {:else}
        <div class="row g-4 justify-content-center" id="contenedorProductos">
          {#if obtenerProductosPaginados().length === 0}
            <p class="text-center text-muted">No se encontraron productos.</p>
          {:else}
            {#each obtenerProductosPaginados() as item}
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

      <div class="d-flex justify-content-center mt-4">
        <button
          class="btn btn-outline-secondary me-2"
          on:click={() => paginaActual = Math.max(1, paginaActual - 1)}
          disabled={paginaActual === 1}
        >
          Anterior
        </button>
        <span class="align-self-center">Página {paginaActual} de {Math.ceil(filtrarProductos().length / productosPorPagina)}</span>
        <button
          class="btn btn-outline-secondary ms-2"
          on:click={() => paginaActual = Math.min(Math.ceil(filtrarProductos().length / productosPorPagina), paginaActual + 1)}
          disabled={paginaActual === Math.ceil(filtrarProductos().length / productosPorPagina)}
        >
          Siguiente
        </button>
      </div>
    </main>

    {#if alerta}
      <div class="position-fixed bottom-0 end-0 m-3 alert alert-success" style="z-index: 1050; min-width: 250px">
        {alerta}
      </div>
    {/if}
  </div>
{/if}