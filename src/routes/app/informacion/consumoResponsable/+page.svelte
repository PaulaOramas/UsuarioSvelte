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

<div class="container mt-5 mb-5">
  <h1 class="fw-bold text-center mb-4" style="font-family: 'Pacifico', cursive; color: #6b1e25;">
    Consumo Responsable
  </h1>

  <div class="d-flex justify-content-end align-items-center gap-2 mb-3" style="max-width:900px;margin:auto;">
    <button
      class="btn-modo-oscuro"
      aria-label="Cambiar modo oscuro/claro"
      on:click={() => setDarkMode(!darkMode)}
      title={darkMode ? 'Modo claro' : 'Modo oscuro'}
    >
      {#if darkMode}
        <i class="bi bi-sun-fill"></i>
      {:else}
        <i class="bi bi-moon-stars-fill"></i>
      {/if}
    </button>
  </div>

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
  .btn-modo-oscuro {
    background: #fff;
    border: 2px solid #4B2563;
    color: #4B2563;
    border-radius: 50%;
    width: 44px;
    height: 44px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 1.5rem;
    box-shadow: 0 2px 8px #4b256320;
    transition: background 0.2s, color 0.2s, border 0.2s;
    cursor: pointer;
  }
  .btn-modo-oscuro:hover {
    background: #4B2563;
    color: #fff;
    border-color: #7c3aed;
  }
</style>