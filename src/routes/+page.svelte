<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import reading from "$lib/reading.json";
  import ReadingItem from "$lib/ReadingItem.svelte";
  import { onMount } from "svelte";

  let githubData = null; // This will eventually hold our Github stats
  let loading = true; // This will be true *until* the fetch's promise resolves to a value
  let error = null; // If the API call resulted in an error, it will go into this variable

  onMount(async () => {
    console.log("Page has been mounted!")
    try {
      let response = await fetch("https://api.github.com/users/emaanbilal");
      console.log(response);
      githubData = await response.json();
      console.log(githubData);
    } catch (err) {
      error = err;
    }
    loading = false;
  });

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

  {#if loading}
    <p>Loading...</p>
  {:else if error}
    <p>Something went wrong: {error.message}</p>
  {:else}
    <section class="github-stats">
      <h2>My GitHub Stats</h2>
      <div class="github-stats-grid">
        <div class="github-stat">
          <span class="github-stat-label">Followers</span>
          <span class="github-stat-value">{githubData.followers}</span>
        </div>
        <div class="github-stat">
          <span class="github-stat-label">Following</span>
          <span class="github-stat-value">{githubData.following}</span>
        </div>
        <div class="github-stat">
          <span class="github-stat-label">Public Repos</span>
          <span class="github-stat-value">{githubData.public_repos}</span>
        </div>
      </div>
    </section>
  {/if}



  <h2>Recent Projects</h2>
  <div class="projects">
    {#each projects.slice(0, 3) as p}
      <Project data={p} />
    {/each}
  </div>
</main>