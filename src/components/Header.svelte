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
  nav.navbar {
    background: linear-gradient(90deg, #fff 0%, #ece6f3 50%, #fff 100%);
    box-shadow: 0 2px 12px #0001;
    font-family: 'Montserrat', sans-serif;
    border-bottom: 2px solid #4B2563;
  }
  .navbar-brand span {
    font-family: 'Montserrat', sans-serif;
    color: #4B2563;
    letter-spacing: 1.5px;
    font-size: 2rem;
    font-weight: 800;
    text-shadow: 0 2px 8px #e0d2ee;
    text-transform: uppercase;
    letter-spacing: 2px;
  }
  .nav-link {
    color: #4B2563 !important;
    font-weight: 600 !important;
    font-family: 'Montserrat', sans-serif !important;
    letter-spacing: 0.5px;
    font-size: 1.08rem;
    transition: color 0.25s, background 0.25s;
    border-radius: 0.5em;
    padding: 0.4em 1em;
  }
  .nav-link:hover,
  .nav-link:focus {
    color: #fff !important;
    background: #4B2563 !important;
    text-decoration: none;
    outline: none;
  }
  .carrito-indicator {
    position: absolute;
    top: -0.5em;
    left: 1.5em;
    font-size: 0.8em;
    background: #7c3aed;
    color: #fff;
    border-radius: 50%;
    padding: 0 7px;
    min-width: 20px;
    height: 20px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: 700;
    border: 2px solid #fff;
    box-shadow: 0 2px 8px #0002;
  }
  .fw-semibold.text-dark {
    font-family: 'Montserrat', sans-serif;
    color: #4B2563 !important;
    font-size: 1.1rem;
    margin-left: 0.5em;
    font-weight: 700;
    text-shadow: 0 2px 8px #e0d2ee;
    letter-spacing: 1px;
  }
  .navbar-toggler {
    border-color: #4B2563;
  }
  .navbar-toggler-icon {
    background-image: url("data:image/svg+xml,%3csvg viewBox='0 0 30 30' xmlns='http://www.w3.org/2000/svg'%3e%3cpath stroke='rgba(75,37,99,0.7)' stroke-width='2' stroke-linecap='round' stroke-miterlimit='10' d='M4 7h22M4 15h22M4 23h22'/%3e%3c/svg%3e");
  }
  @media (max-width: 991.98px) {
    .dropdown-divider {
      border-color: #4B2563;
    }
    .navbar-brand span {
      font-size: 1.3rem;
    }
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
