<script>
  import { onMount } from 'svelte';

  // Estado principal
  let usuarioActual = '';
  let usuarioData = null;
  let facturas = [];
  let detalleFactura = [];
  let totalFactura = 0;
  let modalFacturaNumero = '';
  let mostrarModal = false;
  let mensaje = '';
  let mostrarError = false;

  // Login
  let cedula = '';
  let contrasena = '';
  let mensajeLogin = '';
  let mostrarMensajeLogin = false;

  // Cargar usuario y perfil al montar
  onMount(async () => {
    usuarioActual = localStorage.getItem('usuario') || '';
    if (usuarioActual) {
      await cargarPerfil();
    }
  });

  async function cargarPerfil() {
    try {
      const res = await fetch(`https://eltragolocorest.runasp.net/api/Usuario?ciRuc=${usuarioActual}`);
      if (!res.ok) throw new Error('No se pudo obtener el usuario');
      usuarioData = await res.json();
      await cargarFacturas();
    } catch (err) {
      mensaje = '❌ Error al obtener los datos del usuario.';
      mostrarError = true;
    }
  }

  async function cargarFacturas() {
    try {
      const res = await fetch('https://eltragolocorest.runasp.net/api/Factura');
      if (!res.ok) throw new Error('No se pudo obtener las facturas');
      const todas = await res.json();
      facturas = todas.filter(f => f.USU_CI_RUC === usuarioActual && f.FAC_ESTADO === 'PAG');
    } catch (err) {
      mensaje = '❌ Error al cargar las facturas.';
      mostrarError = true;
    }
  }

  async function verDetalleFactura(facNumero) {
    modalFacturaNumero = facNumero;
    detalleFactura = [];
    totalFactura = 0;
    mostrarModal = true;
    try {
      const res = await fetch(`https://eltragolocorest.runasp.net/api/DetalleFactura/${facNumero}`);
      if (!res.ok) throw new Error('No se pudo obtener el detalle');
      const detalles = await res.json();
      detalleFactura = await Promise.all(detalles.map(async d => {
        const prodRes = await fetch(`https://eltragolocorest.runasp.net/api/Producto/${d.PROD_ID}`);
        const prod = await prodRes.json();
        const subtotal = d.DXF_CANTIDAD * d.DXF_PRECIO_UNIT;
        totalFactura += subtotal;
        return {
          nombre: prod.PROD_NOMBRE,
          cantidad: d.DXF_CANTIDAD,
          precio: d.DXF_PRECIO_UNIT,
          subtotal
        };
      }));
    } catch (err) {
      mensaje = '❌ Error al cargar los detalles de la factura.';
      mostrarError = true;
      mostrarModal = false;
    }
  }

  function cerrarSesion() {
    localStorage.removeItem('usuario');
    window.location.reload();
  }

  function ciudadNombre(cod) {
    switch (cod) {
      case "UIO": return "Quito";
      case "CUE": return "Cuenca";
      case "GYE": return "Guayaquil";
      default: return cod || "Desconocido";
    }
  }

  // Login handler
  async function login(e) {
    e.preventDefault();
    mensajeLogin = '';
    mostrarMensajeLogin = false;

    if (!cedula.match(/^\d{10}$|^\d{13}$/)) {
      mensajeLogin = 'La cédula o RUC debe tener 10 o 13 dígitos numéricos.';
      mostrarMensajeLogin = true;
      return;
    }
    if (!contrasena) {
      mensajeLogin = 'La contraseña es obligatoria.';
      mostrarMensajeLogin = true;
      return;
    }

    try {
      const res = await fetch(`https://eltragolocorest.runasp.net/api/Login/Usuario/${cedula}`);
      if (!res.ok) {
        mensajeLogin = 'Usuario no encontrado.';
        mostrarMensajeLogin = true;
        return;
      }
      const data = await res.json();
      if (!data || !data.PASSWORD) {
        mensajeLogin = 'Usuario o contraseña incorrectos.';
        mostrarMensajeLogin = true;
        contrasena = '';
        return;
      }
      if (data.PASSWORD === contrasena) {
        localStorage.setItem('usuario', cedula);
        window.location.reload();
      } else {
        mensajeLogin = 'Usuario o contraseña incorrectos.';
        mostrarMensajeLogin = true;
        contrasena = '';
      }
    } catch (err) {
      mensajeLogin = '❌ Error de conexión. Intenta de nuevo.';
      mostrarMensajeLogin = true;
    }
  }
</script>


<main class="container my-5 p-4 rounded shadow bg-white">
  <h1 class="text-center mb-4 titulo-principal">Perfil de Usuario</h1>

  {#if mostrarError}
    <div class="alert alert-danger">{mensaje}</div>
  {/if}

  {#if usuarioActual && usuarioData}
    <h2 class="mb-3 sub-titulo text-center">Datos Personales</h2>
    <div class="row gx-5 g-3 datos-personales">
      <div class="col-md-6 text-start">
        <div class="mb-2"><span class="fw-bold">Cédula/RUC:</span> <span class="text-dark">{usuarioData.USU_CI_RUC}</span></div>
        <div class="mb-2"><span class="fw-bold">Nombre:</span> <span class="text-dark">{usuarioData.USU_NOMBRE}</span></div>
        <div class="mb-2"><span class="fw-bold">Email:</span> <span class="text-dark">{usuarioData.USU_EMAIL}</span></div>
      </div>
      <div class="col-md-6 text-start">
        <div class="mb-2 ps-md-4"><span class="fw-bold">Teléfono:</span> <span class="text-dark">{usuarioData.USU_TELEFONO}</span></div>
        <div class="mb-2 ps-md-4"><span class="fw-bold">Dirección:</span> <span class="text-dark">{usuarioData.USU_DIRECCION}</span></div>
        <div class="mb-2 ps-md-4"><span class="fw-bold">Ciudad:</span> <span class="text-dark">{ciudadNombre(usuarioData.USU_CIUDAD)}</span></div>
      </div>
    </div>

    <h2 class="mb-3 sub-titulo text-center">Mis Facturas</h2>
    <table class="table table-bordered table-striped align-middle text-center">
      <thead class="table-dark">
        <tr>
          <th class="text-white">ID Factura</th>
          <th class="text-white">Fecha</th>
          <th class="text-white">Total</th>
          <th class="text-white">Detalles</th>
        </tr>
      </thead>
      <tbody>
        {#each facturas as factura}
          <tr>
            <td>{factura.FAC_NUMERO}</td>
            <td>{factura.FAC_FECHA.split('T')[0]}</td>
            <td>${factura.FAC_TOTAL.toFixed(2)}</td>
            <td>
              <button class="btn btn-sm btn-outline-morado" on:click={() => verDetalleFactura(factura.FAC_NUMERO)}>
                Ver detalles
              </button>
            </td>
          </tr>
        {/each}
        {#if facturas.length === 0}
          <tr>
            <td colspan="4">No tienes facturas registradas.</td>
          </tr>
        {/if}
      </tbody>
    </table>

    <button class="btn btn-danger" on:click={cerrarSesion}>Cerrar Sesión</button>
  {:else}
    <div class="login-container">
      <h2 class="mb-4 text-dark">Iniciar Sesión</h2>
      {#if mostrarMensajeLogin}
        <div class="alert alert-danger" role="alert">{mensajeLogin}</div>
      {/if}
      <form id="formLogin" on:submit|preventDefault={login} novalidate>
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
            autocomplete="username"
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
            autocomplete="current-password"
          />
          <div id="contrasenaHelp" class="form-text">Ingrese su contraseña.</div>
        </div>
        <button type="submit" class="btn btn-morado w-100 fw-bold">Ingresar</button>
        <div class="mt-3">
          <span>¿No tienes cuenta?</span>
          <a href="/app/cuenta/registro" class="text-decoration-none fw-bold text-danger">Regístrate aquí</a>
        </div>
      </form>
    </div>
  {/if}

  {#if mostrarModal}
    <div
      class="modal fade show"
      style="display:block; background:rgba(0,0,0,0.4);"
      role="button"
      aria-label="Cerrar modal"
      on:click={() => mostrarModal = false}
      on:keydown={(e) => { if (e.key === 'Enter' || e.key === ' ') mostrarModal = false; }}
      tabindex="0"
    >
      <div class="modal-dialog modal-lg modal-dialog-centered" on:click|stopPropagation>
        <div class="modal-content">
          <div class="modal-header bg-morado text-white">
            <h5 class="modal-title">Detalle de Factura {modalFacturaNumero}</h5>
            <button type="button" class="btn-close btn-close-white" aria-label="Cerrar" on:click={() => mostrarModal = false}></button>
          </div>
          <div class="modal-body">
            <table class="table table-bordered text-center">
              <thead>
                <tr>
                  <th>Producto</th>
                  <th>Cantidad</th>
                  <th>Precio Unitario</th>
                  <th>Subtotal</th>
                </tr>
              </thead>
              <tbody>
                {#each detalleFactura as d}
                  <tr>
                    <td>{d.nombre}</td>
                    <td>{d.cantidad}</td>
                    <td>${d.precio.toFixed(2)}</td>
                    <td>${d.subtotal.toFixed(2)}</td>
                  </tr>
                {/each}
                <tr>
                  <td colspan="3" class="text-end">Total:</td>
                  <td>${totalFactura.toFixed(2)}</td>
                </tr>
              </tbody>
            </table>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" on:click={() => mostrarModal = false}>Cerrar</button>
          </div>
        </div>
      </div>
    </div>
  {/if}
</main>

<style>
  body {
  background: #fff8f0;
}

main.container {
  background: #fff;
  border-radius: 1.5rem;
  box-shadow: 0 8px 32px #a44b5a22;
  padding: 2.5rem 2rem;
  margin-top: 2rem;
  margin-bottom: 2rem;
}

  body.fondo-crema {
    background-color: #f8f9fa;
  }
  .titulo-principal {
    font-family: 'Pacifico', cursive;
    font-size: 2.2rem;
    color: #6b1e25;
    margin-bottom: 0.5rem;
    letter-spacing: 1.5px;
  }
  .sub-titulo {
    color: #a44b5a;
    font-weight: 700;
    font-size: 1.3rem;
    margin-bottom: 1.2rem;
  }
  .btn-outline-morado {
    border-color: #6b1e25;
    color: #6b1e25;
    background-color: transparent;
    font-weight: bold;
  }
  .btn-outline-morado:hover {
    background-color: #6b1e25;
    color: white;
  }
  .modal-header.bg-morado {
    background-color: #6b1e25;
  }
  .modal-header.bg-morado .modal-title {
    color: white;
  }
  .modal-content {
    border-radius: 15px;
  }
  .table-dark {
    background-color: #6b1e25;
    color: white;
  }
  .table-dark th {
    background-color: #6b1e25;
    color: white !important;
    font-weight: bold;
  }
  .table-dark td {
    border: 1px solid #6b1e25;
  }
  .login-container {
    background: #fff;
    padding: 2rem 2.5rem;
    border-radius: 1rem;
    box-shadow: 0 10px 30px rgba(107, 30, 37, 0.15);
    max-width: 400px;
    width: 100%;
    text-align: center;
    margin: 3rem auto 2rem auto;
  }
  .login-container h2 {
    font-family: 'Pacifico', cursive;
    color: #6b1e25;
    margin-bottom: 1.5rem;
    font-size: 2rem;
  }
  input {
    border: 1px solid #6b1e25;
    border-radius: 8px;
    padding: 0.8rem;
    width: 100%;
    margin-bottom: 1.2rem;
    font-size: 1rem;
  }
  .form-text {
    color: #6b1e25;
    font-size: 0.9rem;
  }
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
  a {
    color: #a44b5a;
    text-decoration: underline;
    font-weight: 600;
  }
  a:hover {
    color: #6b1e25;
  }
  .btn-danger {
    background: #a44b5a;
    border: none;
    color: #fff;
    font-weight: 700;
    border-radius: 8px;
    padding: 0.5rem 1.5rem;
    margin-top: 1.5rem;
  }
  .btn-danger:hover {
    background: #6b1e25;
  }
  .alert {
    margin-top: 1rem;
    padding: 0.8rem;
    border-radius: 10px;
  }
  .alert-danger {
    background-color: #f8d7da;
    color: #721c24;
    border: 1px solid #f5c6cb;
  }
  .datos-personales .fw-bold {
    color: #6b1e25;
  }
  .datos-personales span.text-dark {
    color: #333 !important;
  }
  .datos-personales {
    margin-bottom: 2rem;
    font-size: 1.08rem;
  }
  .table {
    border-radius: 12px;
    overflow: hidden;
    background: #fff;
    box-shadow: 0 2px 12px #a44b5a22;
  }

  .table thead {
    background: #6b1e25;
  }

  .table thead th {
    color: #fff !important;
    font-weight: 700;
    font-size: 1.08rem;
    border: none;
  }

  .table-striped tbody tr:nth-of-type(odd) {
    background-color: #f8f9fa;
  }

  .table-striped tbody tr:nth-of-type(even) {
    background-color: #fff8f0;
  }

  .table td, .table th {
    border: none;
    vertical-align: middle;
  }

  .btn-outline-morado {
    border-color: #a44b5a;
    color: #a44b5a;
    background-color: transparent;
    font-weight: bold;
    transition: background 0.2s, color 0.2s;
    border-radius: 8px;
    padding: 0.3rem 1.1rem;
  }

  .btn-outline-morado:hover {
    background-color: #a44b5a;
    color: #fff;
  }

  .modal-body table th {
  background: #6b1e25 !important;
  color: #fff !important;
  font-weight: bold;
  font-size: 1.08rem;
  border: none;
}
.modal-body table td {
  color: #333;
  font-size: 1rem;
  background: #fff;
  border: none;
}

  @media (max-width: 600px) {
  main.container {
    padding: 1rem 0.2rem;
  }
  .login-container {
    padding: 1.2rem 0.5rem;
  }
}
</style>