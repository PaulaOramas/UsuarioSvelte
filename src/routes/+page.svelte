<script>
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';

  let anio = '';
  let mes = '';
  let dia = '';
  let mensajeError = '';
  let darkMode = false;

  onMount(() => {
    darkMode = localStorage.getItem('dark-mode') === 'enabled';
    setDarkMode(darkMode);
  });

  function setDarkMode(enabled) {
    darkMode = enabled;
    if (darkMode) {
      document.body.classList.add('dark-mode');
      document.body.classList.remove('fondo-verificacion'); // quita fondo en oscuro
      localStorage.setItem('dark-mode', 'enabled');
    } else {
      document.body.classList.remove('dark-mode');
      document.body.classList.add('fondo-verificacion'); // pone fondo en claro
      localStorage.setItem('dark-mode', 'disabled');
    }
  }

  function verificarEdad() {
    const anioNum = parseInt(anio);
    const mesNum = parseInt(mes) - 1;
    const diaNum = parseInt(dia);

    const fechaNacimiento = new Date(anioNum, mesNum, diaNum);
    const hoy = new Date();

    let edad = hoy.getFullYear() - fechaNacimiento.getFullYear();
    const m = hoy.getMonth() - fechaNacimiento.getMonth();
    if (m < 0 || (m === 0 && hoy.getDate() < fechaNacimiento.getDate())) {
      edad--;
    }

    if (edad >= 18) {
      mensajeError = '';
      localStorage.setItem("autorizado", "true");
      goto('/app/productos/index');
    } else {
      mensajeError = "Debes ser mayor de 18 años para ingresar.";
    }
  }
</script>

<!-- Bootstrap 5 CDN -->
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet" />

<div class="fondo-centrado">
  <div id="form-wrapper" class="shadow-lg">
    <img src="/imagenes/logo.png" alt="Logo Trago Loco" id="logo" class="mb-3" style="max-width:120px;" />
    <h2 class="mb-4 text-center">Cuéntanos: ¿Cuándo naciste?</h2>
    <form on:submit|preventDefault={verificarEdad} id="formEdad">
      <div class="d-flex gap-2 mb-3 justify-content-center">
        <input type="number" bind:value={anio} class="form-control text-center" placeholder="AAAA" required min="1900" max="2099" style="max-width:90px;" />
        <input type="number" bind:value={mes} class="form-control text-center" placeholder="MM" required min="1" max="12" style="max-width:70px;" />
        <input type="number" bind:value={dia} class="form-control text-center" placeholder="DD" required min="1" max="31" style="max-width:70px;" />
      </div>
      <button type="submit" class="btn-verificar w-100">Entrar</button>
    </form>
    <div id="mensajeError" class="text-danger mt-2 text-center">{mensajeError}</div>
  </div>
</div>

<!-- Botón flotante de modo oscuro/claro -->
<button
  id="darkModeSwitch"
  aria-label="Activar modo oscuro"
  style="
    position: fixed;
    top: 1rem;
    right: 1rem;
    z-index: 9999;
    background: #232323;
    color: #fff;
    border: none;
    border-radius: 50%;
    width: 44px;
    height: 44px;
    display: flex;
    align-items: center;
    justify-content: center;
    box-shadow: 0 2px 8px rgba(0,0,0,0.2);
    font-size: 1.3rem;
  "
  on:click={() => setDarkMode(!darkMode)}
>
  <span id="darkModeIcon">{darkMode ? '☀️' : '🌙'}</span>
</button>

<style>
  /* Fondo centrado y responsivo */
  .fondo-centrado {
    min-height: 100vh;
    min-width: 100vw;
    display: flex;
    align-items: center;
    justify-content: center;
    /* El fondo se aplica por clase al body, no aquí */
  }

  /* Fondo de imagen solo en modo claro */
  :global(body.fondo-verificacion) {
    background: url('/imagenes/colores.jpg') center center/cover no-repeat fixed !important;
  }

  /* Centrado y estilos del cuadro */
  #form-wrapper {
    background: rgba(255,255,255,0.92);
    border-radius: 18px;
    padding: 2.5rem 2rem 2rem 2rem;
    max-width: 370px;
    width: 100%;
    margin: auto;
    box-shadow: 0 4px 32px rgba(0,0,0,0.12);
  }
  :global(body.dark-mode) #form-wrapper {
    background: #232323;
    color: #fff;
  }
  .btn-verificar {
    background: #4c145c;
    color: #fff;
    font-weight: 700;
    border-radius: 8px;
    border: none;
    padding: 0.6rem 0;
    margin-top: 0.5rem;
    transition: background 0.2s;
  }
  .btn-verificar:hover {
    background: #6f42c1;
  }
  #logo {
    display: block;
    margin: 0 auto 1rem auto;
  }
</style>