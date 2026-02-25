<svelte:head>
  <title>Projects</title>
</svelte:head>

<script>
  import projects from "$lib/projects.json";
  import Project from "$lib/Project.svelte";
</script>

<!DOCTYPE html>
<html lang="en">
    <link rel="stylesheet" href="../style.css">
<head>
  <nav>
      <a href=".">Home</a>
      <a class="current" href="projects">Projects</a>
      <a href="contact">Contact</a>
      <a href="resume">Resume</a>
      <a href="https://github.com/emaanbilal" target="_blank">GitHub</a>
  </nav>
</head>
<body>
    <h1> {projects.length} Projects</h1>
    <div class="projects">
        {#each projects as p}
        <Project data={p} />
        {/each}
       
    </div>
</body>
</html>