<script>
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';

  let cedula = '';
  let contrasena = '';
  let mensaje = '';
  let usuarioActual = '';
  let mostrarError = false;

  onMount(() => {
    usuarioActual = localStorage.getItem('usuario') || '';
  });

  function login(e) {
    e.preventDefault();
    mensaje = '';
    mostrarError = false;

    // Validación básica
    if (!cedula.match(/^\d{10,13}$/) || !contrasena) {
      mensaje = 'Por favor, complete correctamente todos los campos.';
      mostrarError = true;
      return;
    }

    // Aquí puedes agregar validación real con API si lo deseas
    localStorage.setItem('usuario', cedula);
    usuarioActual = cedula;
    // Si tienes un store global para el usuario, actualízalo aquí

    goto('/app/productos/index');
  }

  function cerrarSesion() {
    localStorage.removeItem('usuario');
    usuarioActual = '';
    // Si tienes un store global para el usuario, actualízalo aquí
  }
</script>

<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet" />
<link href="https://fonts.googleapis.com/css2?family=Pacifico&display=swap" rel="stylesheet" />

<div class="container my-5 p-4 rounded shadow bg-white" style="max-width: 400px;">
  {#if usuarioActual}
    <div class="alert alert-info fw-semibold" role="alert">
      Ya has iniciado sesión como <span class="text-primary">{usuarioActual}</span>.<br />
      <button type="button" class="btn btn-sm btn-outline-danger mt-2" on:click={cerrarSesion}>Cerrar sesión</button>
    </div>
  {:else}
    <h2 class="mb-4 text-dark" style="font-family: 'Pacifico', cursive;">Iniciar Sesión</h2>
    {#if mostrarError}
      <div class="alert alert-danger" role="alert">{mensaje}</div>
    {/if}
    <form on:submit|preventDefault={login} novalidate>
      <div class="mb-3 text-start">
        <label for="cedula" class="form-label">Cédula:</label>
        <input
          type="text"
          id="cedula"
          name="cedula"
          class="form-control"
          bind:value={cedula}
          required
          maxlength="13"
          pattern="\d{10,13}"
          title="Ingrese una cédula o RUC válida"
          aria-describedby="cedulaHelp"
        />
        <div id="cedulaHelp" class="form-text">Ingrese su cédula o RUC (10-13 dígitos).</div>
      </div>
      <div class="mb-3 text-start">
        <label for="contrasena" class="form-label">Contraseña:</label>
        <input
          type="password"
          id="contrasena"
          name="contrasena"
          class="form-control"
          bind:value={contrasena}
          required
          aria-describedby="contrasenaHelp"
        />
        <div id="contrasenaHelp" class="form-text">Ingrese su contraseña.</div>
      </div>
      <button type="submit" class="btn btn-morado w-100 fw-bold">Ingresar</button>
      <div class="mt-3">
        <span>¿No tienes cuenta?</span>
        <a href="/app/cuenta/registro" class="text-decoration-none fw-bold text-danger">Regístrate aquí</a>
      </div>
    </form>
  {/if}
</div>

<style>
  .btn-morado {
    background-color: #6b1e25;
    color: #f5f5dc;
    font-weight: 700;
    border-radius: 30px;
    padding: 0.6rem 1.5rem;
    transition: background-color 0.3s ease;
    border: none;
    margin-top: 1rem;
    width: 100%;
    cursor: pointer;
  }
  .btn-morado:hover,
  .btn-morado:focus {
    background-color: #a44b5a;
    color: #fff;
    outline: none;
    box-shadow: 0 0 12px #a44b5aaa;
  }
</style>