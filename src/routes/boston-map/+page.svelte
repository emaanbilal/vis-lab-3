<script>
  import BostonEvictionMap from "$lib/BostonEvictionMap.svelte";
</script>

<main class="page">
  <BostonEvictionMap />
</main>

<style>
  .page {
    max-width: 120rem;
    margin-inline: auto;
    padding: 1rem clamp(0.75rem, 2vw, 1.5rem) 3rem;
  }
</style>
