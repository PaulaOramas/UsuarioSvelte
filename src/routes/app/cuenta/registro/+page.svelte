<script>
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';

  // Campos del formulario
  let cedula = '';
  let nombre = '';
  let email = '';
  let contrasena = '';
  let telefono = '';
  let genero = '';
  let ciudad = '';
  let sector = '';
  let direccion = '';
  let mostrarContrasena = false;

  // Validaciones y mensajes
  let mensajeError = '';
  let mensajeExito = '';

  // Sectores por ciudad
  const sectoresPorCiudad = {
    Quito: ["La Mariscal", "Cumbayá", "Carcelén", "La Floresta"],
    Guayaquil: ["Urdesa", "Samborondón", "Alborada", "Centro"],
    Cuenca: ["El Ejido", "Totoracocha", "Baños", "Miraflores"]
  };
  let sectores = [];

  $: sectores = ciudad && sectoresPorCiudad[ciudad] ? sectoresPorCiudad[ciudad] : [];

  function togglePassword() {
    mostrarContrasena = !mostrarContrasena;
  }

  function validarCampos() {
    if (!cedula.match(/^\d{10}$|^\d{13}$/)) return 'La cédula o RUC debe tener 10 o 13 dígitos.';
    if (!nombre.match(/^[a-zA-ZáéíóúÁÉÍÓÚÑñ\s]+$/)) return 'El nombre solo puede contener letras y espacios.';
    if (!email.match(/^[^\s@]+@[^\s@]+\.[^\s@]+$/)) return 'El correo electrónico debe tener un formato válido.';
    if (!contrasena) return 'La contraseña es obligatoria.';
    if (!telefono.match(/^\d{9}$/)) return 'El teléfono debe tener exactamente 9 dígitos.';
    if (!genero) return 'El género es obligatorio.';
    if (!ciudad) return 'Debes seleccionar una ciudad.';
    if (!sector) return 'Debes seleccionar un sector.';
    if (!direccion) return 'La dirección es obligatoria.';
    return '';
  }

  async function registrar(e) {
    e.preventDefault();
    mensajeError = '';
    mensajeExito = '';

    const error = validarCampos();
    if (error) {
      mensajeError = error;
      return;
    }

    const usuarioDTO = {
      USU_CI_RUC: cedula,
      USU_NOMBRE: nombre,
      USU_EMAIL: email,
      USU_TELEFONO: telefono,
      USU_DIRECCION: direccion,
      USU_CIUDAD: ciudad,
      USU_SECTOR_ENTREGA: sector,
      USU_GENERO: genero,
      USU_FECHA_REGISTRO: new Date().toISOString(),
      USU_ESTADO: "ACT"
    };

    try {
      const response = await fetch('https://eltragolocorest.runasp.net/api/Usuario', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(usuarioDTO)
      });
      if (!response.ok) throw new Error(await response.text() || 'Error al registrar los datos del usuario');

      const contrasenaDTO = {
        USU_CI_RUC: cedula,
        PASSWORD: contrasena,
        LOG_ROL: "CLI"
      };
      const responseContrasena = await fetch('https://eltragolocorest.runasp.net/api/Login/Insertar', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(contrasenaDTO)
      });
      if (!responseContrasena.ok) throw new Error(await responseContrasena.text() || 'Error al registrar la contraseña');

      mensajeExito = "✅ Usuario registrado exitosamente.";
      setTimeout(() => goto('/app/cuenta/login'), 1200);
    } catch (error) {
      mensajeError = "❌ " + error.message;
    }
  }
</script>

<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet" />
<link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@400;600&family=Pacifico&display=swap" rel="stylesheet" />


<main class="registro-main">
  <h2>📝 Crear Cuenta</h2>

  {#if mensajeError}
    <div class="alert alert-danger">{mensajeError}</div>
  {/if}
  {#if mensajeExito}
    <div class="alert alert-success">{mensajeExito}</div>
  {/if}

  <form on:submit|preventDefault={registrar} novalidate>
    <div class="mb-3">
      <label for="cedula" class="form-label">Cédula o RUC</label>
      <input id="cedula" name="cedula" type="text" class="form-control" maxlength="13"
        bind:value={cedula}
        pattern="\d{10,13}"
        placeholder="Ej. 0912345678"
        required />
    </div>

    <div class="mb-3">
      <label for="nombre" class="form-label">Nombre completo</label>
      <input id="nombre" name="nombre" type="text" class="form-control"
        bind:value={nombre}
        placeholder="Tu nombre completo"
        required />
    </div>

    <div class="mb-3">
      <label for="email" class="form-label">Correo electrónico</label>
      <input id="email" name="email" type="email" class="form-control"
        bind:value={email}
        placeholder="ejemplo@correo.com"
        required />
    </div>

    <div class="mb-3 position-relative">
      <label for="contrasena" class="form-label">Contraseña</label>
      <div style="position:relative; display:inline-block; width:100%;">
        <input id="contrasena" name="contrasena" type={mostrarContrasena ? "text" : "password"} class="form-control"
          bind:value={contrasena}
          placeholder="********"
          required
          style="padding-right:2.5rem;" />
        <button
          type="button"
          tabindex="-1"
          aria-label={mostrarContrasena ? "Ocultar contraseña" : "Mostrar contraseña"}
          style="position:absolute; top:50%; right:1rem; transform:translateY(-50%); background:transparent; border:none; outline:none; padding:0;"
          on:click={togglePassword}
        >
          <svg width="26" height="26" viewBox="0 0 24 24" fill="none" stroke="#a44b5a"
            stroke-width="2" stroke-linecap="round" stroke-linejoin="round" style="transition:0.2s;">
            <ellipse cx="12" cy="12" rx="8" ry="5" />
            <circle cx="12" cy="12" r="2.2" fill="#a44b5a" />
            <path d="M3 3L21 21" stroke="#a44b5a" stroke-width="2"
              style="opacity:{mostrarContrasena ? 1 : 0};transition:opacity 0.2s;" />
          </svg>
        </button>
      </div>
    </div>

    <div class="mb-3">
      <label for="telefono" class="form-label">Teléfono</label>
      <div class="input-group">
        <span class="input-group-text">🇪🇨 +593</span>
        <input id="telefono" name="telefono" type="tel" class="form-control"
          bind:value={telefono}
          pattern="[0-9]{9}" maxlength="9"
          placeholder="Ej. 991234567"
          required />
      </div>
    </div>

    <div class="mb-3">
      <label for="genero" class="form-label">Género</label>
      <select id="genero" name="genero" class="form-select" bind:value={genero} required>
        <option value="" disabled selected>Seleccionar</option>
        <option>Masculino</option>
        <option>Femenino</option>
        <option>Otro</option>
      </select>
    </div>

    <div class="mb-3">
      <label for="ddlCiudad" class="form-label">Ciudad</label>
      <select id="ddlCiudad" name="ciudad" class="form-select" bind:value={ciudad} required>
        <option value="" disabled selected>Selecciona tu ciudad</option>
        <option value="Quito">Quito</option>
        <option value="Guayaquil">Guayaquil</option>
        <option value="Cuenca">Cuenca</option>
      </select>
    </div>

    <div class="mb-3">
      <label for="ddlSector" class="form-label">Sector de entrega</label>
      <select id="ddlSector" name="sector" class="form-select" bind:value={sector} required>
        <option value="" disabled selected>Selecciona una ciudad primero</option>
        {#each sectores as s}
          <option value={s}>{s}</option>
        {/each}
      </select>
    </div>

    <div class="mb-3">
      <label for="direccion" class="form-label">Dirección</label>
      <input id="direccion" name="direccion" type="text" class="form-control"
        bind:value={direccion}
        placeholder="Tu Dirección"
        required />
    </div>

    <button type="submit">Registrarse</button>
  </form>
</main>

<style>
  main.registro-main {
    background: white;
    max-width: 600px;
    width: 90%;
    margin: 3rem auto 2rem auto;
    padding: 2.5rem 3rem;
    border-radius: 1rem;
    box-shadow: 0 15px 35px rgba(107, 30, 37, 0.25);
    position: relative;
  }
  main.registro-main h2 {
    font-family: 'Pacifico', cursive;
    font-size: 2.5rem;
    color: #6b1e25;
    text-align: center;
    margin-bottom: 1.5rem;
    letter-spacing: 1.2px;
  }
  label {
    font-weight: 600;
    color: #6b1e25;
  }
  input.form-control,
  select.form-select {
    border-radius: 0.6rem;
    border: 2px solid #a44b5a;
    background-color: #fff8f9;
    transition: border-color 0.3s ease, box-shadow 0.3s ease;
    font-size: 1rem;
    padding: 0.65rem 1rem;
  }
  input.form-control:focus,
  select.form-select:focus {
    border-color: #6b1e25;
    box-shadow: 0 0 12px #6b1e25aa;
    outline: none;
    background-color: #fff;
  }
  button[type="submit"] {
    background: linear-gradient(45deg, #6b1e25, #a44b5a);
    color: #f5f5dc;
    font-weight: 700;
    border-radius: 2rem;
    padding: 0.75rem 0;
    font-size: 1.15rem;
    border: none;
    width: 100%;
    margin-top: 1.5rem;
    box-shadow: 0 6px 15px rgba(107, 30, 37, 0.4);
    transition: background 0.4s ease;
    cursor: pointer;
  }
  button[type="submit"]:hover,
  button[type="submit"]:focus {
    background: linear-gradient(45deg, #a44b5a, #6b1e25);
    color: #fff;
    outline: none;
    box-shadow: 0 0 20px #a44b5aaa;
  }
</style>