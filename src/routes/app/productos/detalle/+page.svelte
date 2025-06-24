<script>
  import { onMount } from 'svelte';
  import { page } from '$app/stores';
  import { goto } from '$app/navigation';

  let producto = null;
  let cantidad = 1;
  let error = '';

  // Obtén el id de la URL
  $: id = $page.url.searchParams.get('id');

  onMount(async () => {
    if (!id || isNaN(Number(id))) {
      error = 'ID de producto inválido.';
      return;
    }
    try {
      const res = await fetch(`https://eltragolocorest.runasp.net/api/Producto/${id}`);
      if (!res.ok) throw new Error('Producto no encontrado');
      producto = await res.json();
    } catch {
      error = 'Producto no encontrado o error al cargar.';
    }
  });

  function agregarAlCarrito() {
    if (!producto) return;
    if (cantidad < 1 || isNaN(cantidad)) return;
    if (cantidad > producto.PROD_STOCK) return;

    const carrito = JSON.parse(localStorage.getItem('carrito') || '[]');
    const existente = carrito.find(p => Number(p.PROD_ID) === Number(producto.PROD_ID));
    const precio = parseFloat(producto.PPROD_PRECIO || producto.PROD_PRECIO);

    if (existente) {
      existente.Cantidad = (existente.Cantidad || 1) + cantidad;
    } else {
      carrito.push({
        PROD_ID: Number(producto.PROD_ID),
        PROD_NOMBRE: producto.PROD_NOMBRE,
        PROD_PRECIO: precio,
        PROD_IMG: producto.PROD_IMG,
        Cantidad: cantidad
      });
    }
    localStorage.setItem('carrito', JSON.stringify(carrito));
    goto('/app/carrrito');
  }
</script>

<!-- Bootstrap y CSS -->
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet" />
<link href="/css/oscuro.css" rel="stylesheet" />
<link href="/css/Site.css" rel="stylesheet" />

<div class="container mt-5">
  <div class="row justify-content-center">
    <div class="col-12 col-lg-10">
      <div class="row">
        {#if error}
          <p class="text-center text-danger">{error}</p>
        {:else if !producto}
          <p class="text-center text-secondary">Cargando producto...</p>
        {:else}
          <div class="col-md-6 text-center mb-4">
            <img src={producto.PROD_IMG} alt={producto.PROD_NOMBRE} class="img-fluid rounded shadow" style="max-height: 350px;" />
          </div>
          <div class="col-md-6">
            <h2 class="text-dark fw-bold">{producto.PROD_NOMBRE}</h2>
            <p class="lead text-dark">{producto.PROD_DESCRIPCION}</p>
            <h4 class="text-dark mb-3 fw-bold">
              ${parseFloat(producto.PPROD_PRECIO || producto.PROD_PRECIO).toFixed(2)}
            </h4>
            <div class="mb-3">
              <label for="cantidad" class="text-dark fw-semibold">Cantidad:</label>
              <input
                type="number"
                id="cantidad"
                min="1"
                max={producto.PROD_STOCK}
                bind:value={cantidad}
                class="form-control w-50 d-inline-block"
                required
              />
            </div>
            <button class="btn btn-morado w-100 fw-semibold" on:click={agregarAlCarrito}>
              🛒 Agregar al carrito
            </button>
            <div class="mt-4 text-center">
              <a href="/app/productos/index" class="btn btn-outline-dark text-dark fw-semibold">← Volver al catálogo</a>
            </div>
          </div>
        {/if}
      </div>
    </div>
  </div>
</div>

<style>
  .btn-morado { background: #6f42c1; color: #fff; }
  .btn-morado:hover { background: #5936a6; }
</style>