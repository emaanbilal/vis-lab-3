<script>
  export let data = {};
</script>

<article class="reading-item">
  <img class="reading-cover" src="{data.image}" alt="" />
  <div class="reading-details">
    <h3 class="reading-title">{data.title}</h3>
    <p class="reading-author">{data.author}</p>
  </div>
</article>