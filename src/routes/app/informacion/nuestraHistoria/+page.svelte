<script>
  import { onMount } from 'svelte';

  let darkMode = false;

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
</script>

<!-- Bootstrap y CSS -->
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet" />
<link href="/css/oscuro.css" rel="stylesheet" />
<link href="/css/Site.css" rel="stylesheet" />
<link href="https://fonts.googleapis.com/css2?family=Raleway&display=swap" rel="stylesheet" />
<link href="https://fonts.googleapis.com/css2?family=Pacifico&display=swap" rel="stylesheet" />

<!-- Switch de modo oscuro -->
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

<!-- Carrusel Bootstrap -->
<div id="carouselTragoLoco" class="carousel slide mb-4" data-bs-ride="carousel" aria-label="Carrusel Nuestra Historia">
  <div class="carousel-inner">
    <div class="carousel-item active">
      <img src="/imagenes/bienvenida2.png" class="d-block w-100" alt="Bienvenida a Trago Loco" />
    </div>
    <div class="carousel-item">
      <img src="/imagenes/cokmejorado.png" class="d-block w-100" alt="Variedad de tragos" />
    </div>
    <div class="carousel-item">
      <img src="/imagenes/vodka.jpg" class="d-block w-100" alt="Selección de vodkas" />
    </div>
    <div class="carousel-item">
      <img src="/imagenes/botellas.jpg" class="d-block w-100" alt="Colección de botellas" />
    </div>
  </div>
  <button class="carousel-control-prev" type="button" data-bs-target="#carouselTragoLoco" data-bs-slide="prev" aria-label="Anterior">
    <span class="carousel-control-prev-icon" aria-hidden="true"></span>
  </button>
  <button class="carousel-control-next" type="button" data-bs-target="#carouselTragoLoco" data-bs-slide="next" aria-label="Siguiente">
    <span class="carousel-control-next-icon" aria-hidden="true"></span>
  </button>
</div>

<main class="historia-wrapper">
  <!-- Creadores -->
  <section class="creadores d-flex justify-content-center gap-4 mb-4" aria-label="Creadores de Trago Loco">
    <article class="creador-card text-center">
      <img src="/imagenes/danahe.jpg" alt="Danahe, cofundadora de Trago Loco" class="rounded-circle mb-2" style="width: 140px; height: 140px; object-fit: cover;" />
      <h3>Danahé Andrade</h3>
      <p>Co-fundadora & Especialista en selección de productos</p>
    </article>
    <article class="creador-card text-center">
      <img src="/imagenes/pau.png" alt="Paula, cofundadora de Trago Loco" class="rounded-circle mb-2" style="width: 140px; height: 140px; object-fit: cover;" />
      <h3>Paula Ormas</h3>
      <p>Co-fundadora & Atención al cliente</p>
    </article>
  </section>

  <!-- Texto historia -->
  <section class="historia-texto text-center">
    <h1 style="font-family: 'Pacifico', cursive; color: #6b1e25;">Nuestra Historia</h1>
    <p class="lead">
      Bienvenido a <strong>Trago Loco</strong>, donde la pasión por los mejores licores nos llevó a crear un espacio único para los amantes del buen gusto.
    </p>
    <p>
      Nuestra historia comienza en <strong>2024</strong>, cuando decidimos ofrecer productos de alta calidad, seleccionados cuidadosamente de las mejores bodegas y destilerías del mundo.
      Desde entonces, nos hemos comprometido a brindar experiencias memorables, combinando tradición, calidad y un servicio excepcional.
    </p>
    <p>
      En <strong>Trago Loco</strong> no solo vendemos licores, compartimos momentos, celebraciones y emociones.<br />
      <em>Gracias por ser parte de nuestra historia.</em>
    </p>
  </section>
</main>

<style>
  .historia-wrapper {
    background: #fff8f9;
    border-radius: 1.5rem;
    padding: 2.5rem 1.5rem;
    margin: 2rem auto 2rem auto;
    max-width: 900px;
    box-shadow: 0 8px 32px rgba(107,30,37,0.13);
  }
  .creadores {
    gap: 2.5rem;
    margin-bottom: 2rem;
  }
  .creador-card {
    background: #fff;
    border-radius: 1rem;
    box-shadow: 0 2px 12px #a44b5a22;
    padding: 1.5rem 2rem;
    min-width: 220px;
    max-width: 300px;
  }
  .creador-card h3 {
    color: #6b1e25;
    font-family: 'Pacifico', cursive;
    margin-top: 1rem;
    margin-bottom: 0.5rem;
  }
  .creador-card p {
    color: #a44b5a;
    font-weight: 500;
    margin-bottom: 0;
  }
  .historia-texto h1 {
    font-size: 2.5rem;
    margin-bottom: 1.2rem;
  }
  .historia-texto p {
    font-size: 1.15rem;
    color: #4c145c;
    margin-bottom: 1rem;
  }
  @media (max-width: 768px) {
    .creadores {
      flex-direction: column;
      align-items: center;
      gap: 1.5rem;
    }
    .historia-wrapper {
      padding: 1.2rem 0.5rem;
    }
  }
</style>