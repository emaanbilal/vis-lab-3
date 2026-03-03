<svelte:head>
  <title>Contact</title>
</svelte:head>

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