<script>    
    export let data = {};
</script>

<article>
    <h2>{data.title}</h2>
    <img src="{data.image}" alt="">
    <a href="{data.link}">View Project</a>
</article>

<style>
.projects {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(15em, 1fr));
  gap: 2em;
}

.projects article {
  display: grid;
  grid-template-rows: subgrid;
  grid-row: span 3;
}

.projects article h2 {
  margin: 0;
}

</style>