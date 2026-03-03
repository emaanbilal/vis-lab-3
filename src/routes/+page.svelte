<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import reading from "$lib/reading.json";
  import ReadingItem from "$lib/ReadingItem.svelte";
</script>

<main>
  <div class="intro">
    <div class="intro-main">
      <h1>Emaan Bilal Khan</h1>
      <p>
        I am a Technology and Policy Graduate Student at MIT, where I research
        on AI safety and policy.
      </p>
      <img
        src="images/bg.jpg"
        alt="Photo of Lahore, Pakistan (my hometown)."
      />
    </div>

    <aside class="reading-list">
      <h2>What I'm Reading</h2>
      <div class="reading">
        {#each reading as r}
          <ReadingItem data={r} />
        {/each}
      </div>
    </aside>
  </div>

  <h2>Recent Projects</h2>
  <div class="projects">
    {#each projects.slice(0, 3) as p}
      <Project data={p} />
    {/each}
  </div>
</main>