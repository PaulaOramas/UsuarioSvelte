<script>
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';

  let carrito = [];
  let usuario = '';
  let darkMode = false;

  // Cargar carrito y usuario al montar
  onMount(() => {
    carrito = JSON.parse(localStorage.getItem('carrito')) || [];
    usuario = localStorage.getItem('usuario') || '';
    darkMode = localStorage.getItem('dark-mode') === 'enabled';
    setDarkMode(darkMode);
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

  function modificarCantidad(index, cambio) {
    carrito[index].Cantidad = (carrito[index].Cantidad || carrito[index].CANTIDAD || 1) + cambio;
    if (carrito[index].Cantidad <= 0) {
      carrito.splice(index, 1);
    }
    localStorage.setItem('carrito', JSON.stringify(carrito));
    carrito = [...carrito];
  }

  function eliminarProducto(index) {
    carrito.splice(index, 1);
    localStorage.setItem('carrito', JSON.stringify(carrito));
    carrito = [...carrito];
  }

  function pagar() {
    if (!carrito.length) {
      alert('❗ No tienes productos en el carrito.');
      return;
    }
    const total = carrito.reduce((sum, p) => sum + p.PROD_PRECIO * (p.Cantidad || p.CANTIDAD || 1), 0);
    const iva = total * 0.12;
    const totalConIva = total + iva;

    alert(`✅ ¡Gracias por tu compra!\n\nSubtotal: $${total.toFixed(2)}\nIVA (12%): $${iva.toFixed(2)}\nTotal: $${totalConIva.toFixed(2)}`);
    localStorage.removeItem('carrito');
    carrito = [];
    setTimeout(() => {
      goto('/app/productos/index');
    }, 1000);
  }
</script>

<!-- Bootstrap y CSS -->
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet" />
<link href="https://fonts.googleapis.com/css2?family=Raleway&display=swap" rel="stylesheet" />
<link href="https://fonts.googleapis.com/css2?family=Pacifico&display=swap" rel="stylesheet" />


<style>
  .historia-fondo {
    background-color: #4c145c;
    padding: 1rem 1.5rem;
    border-radius: 15px;
    margin-bottom: 1.5rem;
    color: white !important;
    text-align: center;
  }
  .card-producto {
    display: flex;
    align-items: center;
    gap: 2rem;
    border-radius: 15px;
    background: white;
    padding: 1.5rem 2rem;
    box-shadow: 0 0 12px rgba(0, 0, 0, 0.1);
    width: 100%;
    max-width: 900px;
    box-sizing: border-box;
    color: black;
    margin: auto;
    margin-bottom: 1.5rem;
  }
  .card-producto img {
    width: 180px;
    height: 180px;
    object-fit: contain;
    border-radius: 15px;
  }
  .info-producto {
    flex: 1;
    display: flex;
    flex-direction: column;
    justify-content: center;
    gap: 0.4rem;
  }
  .info-producto h5 {
    font-size: 1.5rem;
    font-weight: 700;
    margin-bottom: 0.5rem;
  }
  .precio {
    font-size: 1.5rem;
    font-weight: 700;
    color: #111;
    text-shadow: 0 1px 2px rgba(0, 0, 0, 0.15);
    margin-bottom: 1rem;
  }
 
  .subtotal {
    font-weight: 700;
    font-size: 1.2rem;
    margin-top: 0.75rem;
  }
  .btn-eliminar {
    color: #dc3545;
    border-color: #dc3545;
  }
  .btn-eliminar:hover {
    background-color: #dc3545;
    color: white;
  }
  #opcionesPago {
    font-size: 1.5rem;
    font-weight: 700;
    color: black;
    margin-top: 2rem;
    text-align: right;
  }
  #opcionesPago button,
  #opcionesPago a {
    font-size: 1.25rem;
    padding: 0.5rem 1.5rem;
  }
  @media (max-width: 576px) {
    .card-producto {
      flex-direction: column;
      max-width: 100%;
      text-align: center;
      padding: 1rem 0.5rem;
      gap: 1rem;
    }
    .card-producto img {
      width: 100% !important;
      max-width: 250px;
      height: auto !important;
      margin: 0 auto;
      display: block;
    }
    #opcionesPago {
      text-align: center;
    }
  }
</style>

<div class="container mt-2 mb-2">
  <div class="row">
    <div class="col-12 d-flex">
      <div class="form-check form-switch">
        <input class="form-check-input" type="checkbox" id="darkModeSwitch" aria-label="Activar modo oscuro" bind:checked={darkMode} on:change={() => setDarkMode(darkMode)}>
        <label class="form-check-label text-dark" for="darkModeSwitch" id="darkModeLabel">{darkMode ? 'Modo Claro' : 'Modo Oscuro'}</label>
      </div>
    </div>
  </div>
</div>


<main class="container mt-5 mb-5">
  <section class="historia-fondo text-center p-3 rounded-4 mb-4 shadow-sm btn-outline-light">
    <h2 class="text-white fw-bold mb-0">🛒 Tu Carrito</h2>
  </section>

  {#if carrito.length === 0}
    <div class="alert alert-info text-center">
      🛒 Tu carrito está vacío. ¡Agrega productos desde el catálogo!
    </div>
  {:else}
    {#each carrito as item, index}
      <div class="card-producto">
        <div class="text-center me-md-4 mb-3 mb-md-0">
          <img src={item.PROD_IMG} alt={item.PROD_NOMBRE} />
        </div>
        <div class="info-producto">
          <h5>{item.PROD_NOMBRE}</h5>
          <p class="precio">${item.PROD_PRECIO.toFixed(2)}</p>
          <div class="acciones-carrito mt-2 mb-2 d-flex justify-content-center align-items-center flex-wrap">
            <button class="btn btn-outline-danger btn-sm me-2" on:click={() => modificarCantidad(index, -1)} disabled={(item.Cantidad || item.CANTIDAD || 1) <= 1}>-</button>
            <span class="mx-2">{item.Cantidad || item.CANTIDAD || 1}</span>
            <button class="btn btn-outline-success btn-sm ms-2" on:click={() => modificarCantidad(index, 1)}>+</button>
          </div>
          <p class="subtotal">Subtotal: ${(item.PROD_PRECIO * (item.Cantidad || item.CANTIDAD || 1)).toFixed(2)}</p>
          <button class="btn btn-outline-danger btn-sm btn-eliminar" on:click={() => eliminarProducto(index)}>🗑 Eliminar</button>
        </div>
      </div>
    {/each}

    <div id="opcionesPago">
      <div><strong>Subtotal:</strong> ${carrito.reduce((sum, p) => sum + p.PROD_PRECIO * (p.Cantidad || p.CANTIDAD || 1), 0).toFixed(2)}</div>
      <div><strong>IVA (12%):</strong> ${(carrito.reduce((sum, p) => sum + p.PROD_PRECIO * (p.Cantidad || p.CANTIDAD || 1), 0) * 0.12).toFixed(2)}</div>
      <div><strong>Total:</strong> ${(carrito.reduce((sum, p) => sum + p.PROD_PRECIO * (p.Cantidad || p.CANTIDAD || 1), 0) * 1.12).toFixed(2)}</div>
      {#if usuario}
        <button class="btn btn-success fw-semibold mt-3" on:click={pagar}>💳 Pagar</button>
      {:else}
        <a href="/app/cuenta/login" class="btn btn-warning fw-semibold mt-3">🔐 Inicia sesión para pagar</a>
      {/if}
    </div>
  {/if}
</main>