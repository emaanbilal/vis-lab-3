<script>
  import Scrolly from "svelte-scrolly";
  import projects from "$lib/projects.json";

  let scrollyProgress = 0;
  let sorted_projects = projects.sort((a, b) => a.year - b.year);
  let progressPerProject = 100 / sorted_projects.length;

  $: activeProjectIdx = Math.min(
    sorted_projects.length - 1,
    Math.floor(scrollyProgress / progressPerProject)
  );

</script>

<div class="scrolly-wrapper">
  <Scrolly bind:progress={scrollyProgress}>
    <section class="project-narrative">
      {#each projects as p}
        <section class="step">
          <div class="step-content">
            <h3>{p.title}</h3>
            <p>{p.story}</p>
          </div>
        </section>
      {/each}
    </section>

    <svelte:fragment slot="viz">
      <div class="project-narrative-viz project-detail">
        <h3>{sorted_projects[activeProjectIdx].year}</h3>
        <img
          src={sorted_projects[activeProjectIdx].image}
          alt={`Image representing project ${sorted_projects[activeProjectIdx].title}`}
        />
      </div>
    </svelte:fragment>
  </Scrolly>
</div>