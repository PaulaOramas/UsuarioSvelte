<script>
  import { onMount } from 'svelte';

  let darkMode = false;

  // Mensajes de consumo responsable
  let mensajes = [
    {
      img: "/imagenes/historia1.jpg",
      titulo: "✅ Evita conducir si has ingerido alcohol.",
      texto: "Cuida tu vida y la de otros. Usa taxi o designa un conductor."
    },
    {
      img: "/imagenes/historia2.jpg",
      titulo: "✅ Respeta tu límite personal y el de los demás.",
      texto: "No todos toleran igual el alcohol. Sé empático y consciente."
    },
    {
      img: "/imagenes/historia3.jpg",
      titulo: "✅ Combina con alimentos y mantente hidratado.",
      texto: "Beber agua y comer mejora tu experiencia y reduce riesgos."
    },
    {
      img: "/imagenes/historia.jpg",
      titulo: "✅ No consumas alcohol si eres menor de edad.",
      texto: "El alcohol puede afectar el desarrollo físico y mental."
    }
  ];

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
<link href="https://fonts.googleapis.com/css2?family=Raleway&display=swap" rel="stylesheet" />
<link href="https://fonts.googleapis.com/css2?family=Pacifico&display=swap" rel="stylesheet" />

<div class="container mt-3 mb-2">
  <div class="row">
    <div class="col-12 d-flex">
      <div class="form-check form-switch">
        <input class="form-check-input" type="checkbox" id="darkModeSwitch" aria-label="Activar modo oscuro" bind:checked={darkMode} on:change={() => setDarkMode(darkMode)}>
        <label class="form-check-label text-dark" for="darkModeSwitch">{darkMode ? 'Modo Claro' : 'Modo Oscuro'}</label>
      </div>
    </div>
  </div>
</div>

<div class="container mt-5 mb-5">
  <h1 class="fw-bold text-center mb-4" style="font-family: 'Pacifico', cursive; color: #6b1e25;">
    Consumo Responsable
  </h1>
  <div class="row g-4" id="bloquesResponsables">
    {#each mensajes as mensaje}
      <div class="col-md-6 col-lg-6">
        <div class="bloque-responsable d-flex align-items-center shadow h-100 p-3 rounded-4 bg-white">
          <img src={mensaje.img} alt="Imagen" class="img-fluid rounded me-4" style="width: 120px; height: auto;" />
          <div>
            <h5 class="fw-bold mb-1">{mensaje.titulo}</h5>
            <p class="mb-0 small">{mensaje.texto}</p>
          </div>
        </div>
      </div>
    {/each}
  </div>
</div>

<style>
  .bloque-responsable {
    background: #fff8f9;
    border: 2px solid #a44b5a22;
    box-shadow: 0 2px 12px #a44b5a22;
    border-radius: 1.2rem;
    margin-bottom: 1.5rem;
    transition: box-shadow 0.2s;
  }
  .bloque-responsable:hover {
    box-shadow: 0 6px 24px #a44b5a44;
  }
  h1 {
    letter-spacing: 1.2px;
  }
</style>