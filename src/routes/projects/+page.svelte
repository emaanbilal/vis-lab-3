<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
  import ProjectNarrative from "$lib/ProjectNarrative.svelte";
  import Bar from "$lib/Bar.svelte";
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  let years = projects.map(proj => proj.year)
  let range = Math.max(...years) - Math.min(...years);

  let rawData = [];
  $: barData = d3.rollups(projects, v => v.length, d => d.year)
    .map(([year, count]) => ({ label: String(year), value: count }));



let wrangled = [];
let total = 0;
onMount(async () => {
    rawData = await d3.json('/lab6_example.json');
    total = d3.sum(rawData, d => d.lines);
    wrangled = d3.rollups(
        rawData,
        v => d3.sum(v, d => d.lines)/total,
        d => d.language
    );
});

$: barData = d3.rollups(projects, v => v.length, d => d.year)
    .map(([year, count]) => ({ label: String(year), value: count }));

</script>

<svelte:head>
  <title>Projects</title>
</svelte:head>

<main>
  <h1>{projects.length} Projects over {range} Years</h1>
            <p>Scroll down to see my a timeline of my projects and how they've contributed to my professional and personal life</p>

    <ProjectNarrative />

    <p class="outro">Thanks for scrolling through my project story! Feel free to explore all of the projects at your leisure below.</p>
  <div class="projects">
    {#each projects as p}
      <Project data={p} />
    {/each}
  </div>
  <section>
    <Bar data={barData} />
</section>
</main>