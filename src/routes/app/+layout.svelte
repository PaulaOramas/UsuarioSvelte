<script>
  import Header from '../../components/Header.svelte';
  import Footer from '../../components/Footer.svelte';
</script>

<Header />

<slot />  <!-- Aquí se renderiza el contenido de cada página -->

<Footer />

