<script>
  import Router, { location } from "svelte-spa-router";
  import Home from "./Home.svelte";
  import About from "./About.svelte";
  import Redirect from "./Redirect.svelte";

  const routes = {
    "/": Home,
    "/about": About,
    "*": Redirect
  };
</script>

<div class="container mt-4">
  <div class="text-center mb-4">
    <h2>Tugas Kecil Strategi Algoritma 4</h2>
    <h2>Pattern Matching</h2>
    <h5>Oleh Moch. Nafkhan Alzamzami (13518132)</h5>
  </div>
  <Router {routes} />
  <div class="mt-5 mb-5">
    <hr />
    <a href={`#/${$location === '/' ? 'about' : ''}`}>
      {$location === '/' ? 'Perihal' : '< Kembali ke halaman utama'}
    </a>
  </div>
</div>
