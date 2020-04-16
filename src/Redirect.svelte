<script>
  import { push } from "svelte-spa-router";

  export let redirect = "/";

  push(redirect);
</script>
