<script>
  import { onMount } from 'svelte';

  let darkMode = false;

  // Preguntas frecuentes (puedes editar o agregar más)
  let faqs = [
    {
      pregunta: "¿Qué métodos de pago aceptan?",
      respuesta: "Aceptamos tarjetas de crédito, débito y transferencias bancarias en Ecuador."
    },
    {
      pregunta: "¿Hacen envíos a todo el país?",
      respuesta: "Sí, realizamos envíos en todo el Ecuador, con entregas de 24 a 72 horas dependiendo del sector."
    },
    {
      pregunta: "¿Qué pasa si un producto llega dañado?",
      respuesta: "Nos comprometemos a reemplazar cualquier producto dañado. Solo debes contactarnos en un plazo de 24 horas."
    },
    {
      pregunta: "¿Puedo comprar sin registrarme?",
      respuesta: "No, para realizar compras debes crear una cuenta. Esto nos ayuda a garantizar una entrega segura."
    }
  ];

  // Formulario de contacto
  let nombre = '';
  let correo = '';
  let mensaje = '';
  let mensajeAlerta = '';
  let tipoAlerta = '';

  onMount(() => {
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

  function enviarContacto(e) {
    e.preventDefault();
    if (nombre && correo && mensaje) {
      mensajeAlerta = `✅ Gracias, ${nombre}. Tu mensaje ha sido enviado correctamente.`;
      tipoAlerta = 'success';
      nombre = '';
      correo = '';
      mensaje = '';
    } else {
      mensajeAlerta = "❗ Por favor completa todos los campos.";
      tipoAlerta = 'danger';
    }
    setTimeout(() => mensajeAlerta = '', 3500);
  }
</script>

<!-- Bootstrap & CSS -->
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet" />
<link href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.10.5/font/bootstrap-icons.css" rel="stylesheet" />
<link href="https://fonts.googleapis.com/css2?family=Raleway&display=swap" rel="stylesheet" />
<link href="https://fonts.googleapis.com/css2?family=Pacifico&display=swap" rel="stylesheet" />

<div class="container mt-3 mb-2">
  <div class="row">
    <div class="col-12 d-flex">
      <div class="form-check form-switch">
        <input class="form-check-input" type="checkbox" id="darkModeSwitch" aria-label="Activar modo oscuro" bind:checked={darkMode} on:change={() => setDarkMode(darkMode)}>
        <label class="form-check-label text-dark" for="darkModeSwitch" id="darkModeLabel">{darkMode ? 'Modo Claro' : 'Modo Oscuro'}</label>
      </div>
    </div>
  </div>
</div>

<div class="container mt-5 mb-5">

  <!-- Preguntas Frecuentes -->
  <div class="row justify-content-center mb-5">
    <div class="col-md-10">
      <div class="p-5 rounded-4 shadow-lg bg-light">
        <h2 class="text-dark fw-bold mb-4 text-center"><i class="bi bi-question-circle-fill me-2"></i>Preguntas Frecuentes</h2>
        <div class="accordion" id="faqAccordion">
          {#each faqs as item, index}
            <div class="accordion-item faq-card">
              <h2 class="accordion-header" id={"faq" + index}>
                <button class="accordion-button collapsed" type="button" data-bs-toggle="collapse" data-bs-target={"#collapse" + index} aria-expanded="false" aria-controls={"collapse" + index}>
                  {item.pregunta}
                </button>
              </h2>
              <div id={"collapse" + index} class="accordion-collapse collapse" data-bs-parent="#faqAccordion">
                <div class="accordion-body">{item.respuesta}</div>
              </div>
            </div>
          {/each}
        </div>
        <p class="mt-4 text-center text-dark small fst-italic">
          ¿No encontraste tu pregunta? Escríbenos abajo y te responderemos con gusto.
        </p>
      </div>
    </div>
  </div>

  <!-- Formulario de contacto -->
  <div class="row justify-content-center">
    <div class="col-md-8">
      <div class="bg-light p-5 rounded-4 shadow-lg">
        <h1 class="fw-bold text-dark text-center mb-3"><i class="bi bi-envelope-fill me-2"></i>Contáctanos</h1>
        <p class="fw-semibold text-center mb-4 text-dark">
          ¿Tienes alguna duda o deseas comunicarte con nosotros? ¡Escríbenos! ✉️
        </p>

        {#if mensajeAlerta}
          <div class="alert alert-{tipoAlerta} text-center">{mensajeAlerta}</div>
        {/if}

        <form on:submit|preventDefault={enviarContacto} autocomplete="off">
          <div class="mb-3">
            <label for="Nombre" class="form-label fw-semibold text-dark">Nombre</label>
            <input type="text" class="form-control" id="Nombre" bind:value={nombre} placeholder="Tu nombre completo" required />
          </div>
          <div class="mb-3">
            <label for="Email" class="form-label fw-semibold text-dark">Correo electrónico</label>
            <input type="email" class="form-control" id="Email" bind:value={correo} placeholder="ejemplo@correo.com" required />
          </div>
          <div class="mb-3">
            <label for="Mensaje" class="form-label fw-semibold text-dark">Mensaje</label>
            <textarea class="form-control" id="Mensaje" rows="4" bind:value={mensaje} placeholder="Escribe tu mensaje aquí..." required></textarea>
          </div>
          <button type="submit" class="btn btn-morado w-100 fw-bold">
            <i class="bi bi-send-fill me-1"></i> Enviar mensaje
          </button>
        </form>
      </div>
    </div>
  </div>
</div>

<style>
  .faq-card {
    border-radius: 1rem;
    margin-bottom: 1rem;
    box-shadow: 0 2px 12px #a44b5a22;
    border: none;
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
  .contacto-body {
    background: #f8f5f0;
  }
</style>