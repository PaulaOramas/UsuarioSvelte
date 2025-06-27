<script>
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';

  let carrito = [];
  let usuario = '';
  let darkMode = false;

  // Cargar carrito y usuario al montar
  onMount(() => {
    carrito = JSON.parse(localStorage.getItem('carrito')) || [];
    usuario = localStorage.getItem('usuario') || '';
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

  function modificarCantidad(index, cambio) {
    carrito[index].Cantidad = (carrito[index].Cantidad || carrito[index].CANTIDAD || 1) + cambio;
    if (carrito[index].Cantidad <= 0) {
      carrito.splice(index, 1);
    }
    localStorage.setItem('carrito', JSON.stringify(carrito));
    carrito = [...carrito];
  }

  function eliminarProducto(index) {
    carrito.splice(index, 1);
    localStorage.setItem('carrito', JSON.stringify(carrito));
    carrito = [...carrito];
  }

  async function pagar() {
    if (!carrito.length) {
      alert('❗ No tienes productos en el carrito.');
      return;
    }

    // Obtener usuario logueado
    const cedulaUsuario = localStorage.getItem("usuario");
    if (!cedulaUsuario) {
      alert('Debes iniciar sesión para pagar.');
      return;
    }

    // Obtener datos del usuario desde la API
    let usuarioData;
    try {
      const res = await fetch(`https://eltragolocorest.runasp.net/api/Usuario?ciRuc=${cedulaUsuario}`);
      if (!res.ok) throw new Error('No se pudo obtener el usuario');
      usuarioData = await res.json();
    } catch (e) {
      alert('Error al obtener datos del usuario.');
      return;
    }

    // Calcular total
    const total = carrito.reduce((sum, p) => sum + p.PROD_PRECIO * (p.Cantidad || p.CANTIDAD || 1), 0);
    const iva = total * 0.12;
    const totalConIva = total + iva;

    // Validar saldo
    if (usuarioData.USU_SALDO < totalConIva) {
      alert('❌ Saldo insuficiente. Tu saldo es: $' + usuarioData.USU_SALDO.toFixed(2));
      return;
    }

    // Preparar datos para la API de factura
    const data = {
      carrito: { Productos: carrito.map(item => ({ idProducto: item.PROD_ID, cantidad: item.Cantidad || item.CANTIDAD || 1 })) },
      direccion: usuarioData.USU_DIRECCION,
      metodoPago: "Saldo",
      cliente: {
        cliCedula: usuarioData.USU_CI_RUC,
        cliNombre: usuarioData.USU_NOMBRE,
        cliApellido: "",
        cliTelefono: usuarioData.USU_TELEFONO
      }
    };

    try {
      const res = await fetch('https://eltragolocorest.runasp.net/api/Factura/insertarFactura', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data)
      });
      if (!res.ok) {
        const errorData = await res.json();
        throw new Error(errorData.Message || 'Error al registrar la compra');
      }
      alert(`✅ ¡Gracias por tu compra!\n\nSubtotal: $${total.toFixed(2)}\nIVA (12%): $${iva.toFixed(2)}\nTotal: $${totalConIva.toFixed(2)}`);
      localStorage.removeItem("carrito");
      carrito = [];
      setTimeout(() => {
        goto('/app/productos/index');
      }, 1000);
    } catch (error) {
      alert("❌ Error al registrar la compra: " + error.message);
    }
  }
</script>

<!-- Bootstrap y CSS -->
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet" />
<link href="https://fonts.googleapis.com/css2?family=Raleway&display=swap" rel="stylesheet" />
<link href="https://fonts.googleapis.com/css2?family=Pacifico&display=swap" rel="stylesheet" />


<style>
  body.dark-mode {
    background: #23172b !important;
    color: #f3e9ff !important;
  }
  .historia-fondo {
    background-color: #4B2563;
    padding: 1rem 1.5rem;
    border-radius: 15px;
    margin-bottom: 1.5rem;
    color: white !important;
    text-align: center;
    box-shadow: 0 4px 24px #4b256340;
  }
  .card-producto {
    display: flex;
    align-items: center;
    gap: 2rem;
    border-radius: 15px;
    background: #fff;
    padding: 1.5rem 2rem;
    box-shadow: 0 0 12px rgba(75,37,99,0.08);
    width: 100%;
    max-width: 900px;
    box-sizing: border-box;
    color: #222;
    margin: auto;
    margin-bottom: 1.5rem;
    transition: background 0.2s, color 0.2s;
  }
  body.dark-mode .card-producto {
    background: #2d1d3a;
    color: #f3e9ff;
  }
  /* Contenedor blanco para la imagen del producto en el carrito */
  .card-producto .img-contenedor {
    background: #fff !important;
    border-radius: 18px;
    display: flex;
    align-items: center;
    justify-content: center;
    width: 200px;
    height: 200px;
    margin: 0 auto;
    box-shadow: 0 2px 8px #4b256320;
    /* Puedes ajustar el tamaño según tu diseño */
  }
  .card-producto img {
    width: 100%;
    height: 100%;
    object-fit: contain;
    background: transparent !important;
    box-shadow: none;
  }
  .info-producto {
    flex: 1;
    display: flex;
    flex-direction: column;
    justify-content: center;
    gap: 0.4rem;
  }
  .info-producto h5 {
    font-size: 1.3rem;
    font-weight: 700;
    margin-bottom: 0.5rem;
    color: #4B2563;
  }
  body.dark-mode .info-producto h5 {
    color: #e0cfff;
  }
  .precio {
    font-size: 1.2rem;
    font-weight: 700;
    color: #7c3aed;
    margin-bottom: 0.5rem;
  }
  .subtotal {
    font-weight: 700;
    font-size: 1.1rem;
    margin-top: 0.5rem;
    color: #4B2563;
  }
  body.dark-mode .subtotal {
    color: #e0cfff;
  }
  .acciones-carrito button {
    font-size: 1.2rem;
    min-width: 38px;
    min-height: 38px;
    border-radius: 50%;
    margin: 0 0.2rem;
    font-weight: 700;
    transition: background 0.2s, color 0.2s;
  }
  .btn-eliminar {
    color: #dc3545;
    border-color: #dc3545;
    margin-top: 0.5rem;
    font-size: 1rem;
    border-radius: 0.7rem;
  }
  .btn-eliminar:hover {
    background-color: #dc3545;
    color: white;
  }
  #opcionesPago {
    font-size: 1.2rem;
    font-weight: 700;
    color: #4B2563;
    margin-top: 2rem;
    text-align: right;
    background: #f3e9ff;
    border-radius: 1rem;
    padding: 1.2rem 2rem;
    box-shadow: 0 2px 8px #4b256320;
  }
  body.dark-mode #opcionesPago {
    background: #2d1d3a;
    color: #e0cfff;
    box-shadow: 0 2px 8px #0004;
  }
  #opcionesPago button,
  #opcionesPago a {
    font-size: 1.15rem;
    padding: 0.5rem 1.5rem;
    border-radius: 0.7rem;
    margin-top: 1rem;
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
  @media (max-width: 576px) {
    .card-producto {
      flex-direction: column;
      max-width: 100%;
      text-align: center;
      padding: 1rem 0.5rem;
      gap: 1rem;
    }
    .card-producto img {
      width: 100% !important;
      max-width: 300px;
      height: auto !important;
      margin: 0 auto;
      display: block;
    }
    #opcionesPago {
      text-align: center;
      padding: 1rem 0.5rem;
    }
  }
</style>


<div class="d-flex justify-content-end align-items-center gap-2 mb-3">
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

<main class="container mt-5 mb-5">
  <section class="historia-fondo text-center p-3 rounded-4 mb-4 shadow-sm btn-outline-light">
    <h2 class="text-white fw-bold mb-0">🛒 Tu Carrito</h2>
  </section>

  {#if carrito.length === 0}
    <div class="alert alert-info text-center">
      🛒 Tu carrito está vacío. ¡Agrega productos desde el catálogo!
    </div>
  {:else}
    {#each carrito as item, index}
      <div class="card-producto">
        <div class="text-center me-md-4 mb-3 mb-md-0">
          <div class="img-contenedor me-md-4 mb-3 mb-md-0">
  <img src={item.PROD_IMG} alt={item.PROD_NOMBRE} />
</div>
        </div>
        <div class="info-producto">
          <h5>{item.PROD_NOMBRE}</h5>
          <p class="precio">${item.PROD_PRECIO.toFixed(2)}</p>
          <div class="acciones-carrito mt-2 mb-2 d-flex justify-content-center align-items-center flex-wrap">
            <button class="btn btn-outline-danger btn-sm me-2" on:click={() => modificarCantidad(index, -1)} disabled={(item.Cantidad || item.CANTIDAD || 1) <= 1}>-</button>
            <span class="mx-2">{item.Cantidad || item.CANTIDAD || 1}</span>
            <button class="btn btn-outline-success btn-sm ms-2" on:click={() => modificarCantidad(index, 1)}>+</button>
          </div>
          <p class="subtotal">Subtotal: ${(item.PROD_PRECIO * (item.Cantidad || item.CANTIDAD || 1)).toFixed(2)}</p>
          <button class="btn btn-outline-danger btn-sm btn-eliminar" on:click={() => eliminarProducto(index)}>🗑 Eliminar</button>
        </div>
      </div>
    {/each}

    <div id="opcionesPago">
      <div><strong>Subtotal:</strong> ${carrito.reduce((sum, p) => sum + p.PROD_PRECIO * (p.Cantidad || p.CANTIDAD || 1), 0).toFixed(2)}</div>
      <div><strong>IVA (12%):</strong> ${(carrito.reduce((sum, p) => sum + p.PROD_PRECIO * (p.Cantidad || p.CANTIDAD || 1), 0) * 0.12).toFixed(2)}</div>
      <div><strong>Total:</strong> ${(carrito.reduce((sum, p) => sum + p.PROD_PRECIO * (p.Cantidad || p.CANTIDAD || 1), 0) * 1.12).toFixed(2)}</div>
      {#if usuario}
        <button class="btn btn-success fw-semibold mt-3" on:click={pagar}>💳 Pagar</button>
      {:else}
        <a href="/app/cuenta/login" class="btn btn-warning fw-semibold mt-3">🔐 Inicia sesión para pagar</a>
      {/if}
    </div>
  {/if}
</main>