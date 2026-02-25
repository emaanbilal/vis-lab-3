<svelte:head>
  <title>Contact</title>
</svelte:head>

<nav>
  <a href=".">Home</a>
  <a href="projects">Projects</a>
  <a class="current" href="contact">Contact</a>
  <a href="resume">Resume</a>
  <a href="https://github.com/emaanbilal" target="_blank">GitHub</a>
</nav>

<main>
  <h1>Contact</h1>
  <form action="mailto:emaan@mit.edu" method="GET">
    <label
      >Email:
      <input type="email" name="email" value="emaanbilal@gmail.com" />
    </label>
    <label
      >Subject:
      <input name="subject" value="Contact from website" />
    </label>
    <label
      >Message:
      <textarea name="body">Testing</textarea>
    </label>
    <button>Send</button>
  </form>
</main>