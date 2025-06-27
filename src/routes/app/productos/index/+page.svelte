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

    <div class="d-flex justify-content-end align-items-center gap-2 mb-3">
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