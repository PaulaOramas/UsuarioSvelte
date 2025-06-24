<script>
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';

  let usuarioNombre = '';
  let carritoCantidad = 0;

  onMount(() => {
    // Cargar usuario desde localStorage
    usuarioNombre = localStorage.getItem('usuarioNombre') || '';
    // Cargar cantidad de productos en el carrito (opcional)
    const carrito = JSON.parse(localStorage.getItem('carrito') || '[]');
    carritoCantidad = carrito.length;
  });

  function irCuenta() {
    if (usuarioNombre) {
      goto('/app/cuenta/perfil');
    } else {
      goto('/app/cuenta/login');
    }
  }

  function irCarrito() {
    goto('/app/carrrito');
  }
</script>

<style>
  .nav-link {
    color: #4b145b !important;
    font-weight: 700 !important;
    font-family: 'Raleway', sans-serif !important;
    letter-spacing: 0.5px;
    transition: color 0.25s ease;
  }
  .nav-link:hover,
  .nav-link:focus {
    color: #333;
    text-decoration: underline;
    outline: none;
  }
  @media (max-width: 991.98px) {
    .dropdown-divider {
      border-color: #666;
    }
  }
  .carrito-indicator {
    position: absolute;
    top: 0;
    left: 1.2em;
    font-size: 0.8em;
    background: #dc3545;
    color: #fff;
    border-radius: 50%;
    padding: 0 6px;
    min-width: 18px;
    height: 18px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: 700;
    border: 2px solid #fff;
  }
</style>

<nav class="navbar navbar-expand-lg navbar-light px-3"
     style="background: linear-gradient(to right, #ffffff, #cccccc, #ffffff); font-family: 'Raleway', sans-serif;">
  <div class="container-fluid">
    <!-- Logo + Nombre -->
    <a class="navbar-brand d-flex align-items-center gap-2" href="/app/productos/index" aria-label="Ir a la página principal">
      <img src="/imagenes/logo.png" alt="Logo Trago Loco" style="height: 40px;" />
      <span class="fs-4" style="color: #000; font-family: 'Pacifico', cursive;">Trago Loco</span>
    </a>

    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav"
            aria-controls="navbarNav" aria-expanded="false" aria-label="Alternar menú de navegación">
      <span class="navbar-toggler-icon"></span>
    </button>

    <div class="collapse navbar-collapse justify-content-end" id="navbarNav">
      <ul class="navbar-nav align-items-lg-center gap-lg-3">
        <li class="nav-item">
          <button class="nav-link btn btn-link p-0" style="text-decoration:none;" on:click={irCuenta}>
            {usuarioNombre ? 'Mi Cuenta' : 'Cuenta'}
          </button>
        </li>
        <li class="nav-item">
          <a class="nav-link" href="/app/informacion/nuestraHistoria">Nuestra Historia</a>
        </li>
        <li class="nav-item">
          <a class="nav-link" href="/app/informacion/consumoResponsable">Consumo Responsable</a>
        </li>
        <li class="nav-item">
          <a class="nav-link" href="/app/informacion/contactanos">Contáctanos</a>
        </li>
        <li class="nav-item d-lg-none"><hr class="dropdown-divider my-2" /></li>
        <li class="nav-item position-relative">
          <button class="nav-link btn btn-link p-0" style="text-decoration:none; position:relative;" aria-label="Ver carrito" on:click={irCarrito}>
            🛒
            {#if carritoCantidad > 0}
              <span class="carrito-indicator">{carritoCantidad}</span>
            {/if}
          </button>
        </li>
      </ul>
      <div class="d-flex align-items-center ms-3">
        {#if usuarioNombre}
          <span class="fw-semibold text-dark">¡Hola, {usuarioNombre}!</span>
        {/if}
      </div>
    </div>
  </div>
</nav>
