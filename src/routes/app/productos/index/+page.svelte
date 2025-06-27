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
    // Si quieres redirigir al carrito, descomenta la siguiente línea:
    // goto('/app/carrrito');
  }

  function mostrarAlerta(msg, duracion = 3000) {
    alerta = msg;
    clearTimeout(alertaTimeout);
    alertaTimeout = setTimeout(() => alerta = '', duracion);
  }
</script>

<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet" />
<link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@400;600;700&family=Pacifico&display=swap" rel="stylesheet" />
<link href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.5/font/bootstrap-icons.css" rel="stylesheet" />

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
</style>

{#if !mostrarContenido}
  <div id="puzzle-overlay" aria-hidden="true" role="presentation"></div>
{/if}

{#if mostrarContenido}
  <div id="main-content" class="visible">
    <video class="video-inicio" autoplay muted loop playsinline preload="auto">
      <source src="/imagenes/GIN COCKTAIL B ROLL.mp4" type="video/mp4" />
      Tu navegador no soporta video HTML5.
    </video>

    <!-- Switch de Modo Oscuro/Claro -->
    <div class="form-check form-switch ms-3 my-2">
      <input class="form-check-input" type="checkbox" id="darkModeSwitch" aria-label="Activar modo oscuro" bind:checked={darkMode} on:change={() => setDarkMode(darkMode)} />
      <label class="form-check-label text-light" for="darkModeSwitch" id="darkModeLabel">{darkMode ? 'Modo Claro' : 'Modo Oscuro'}</label>
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