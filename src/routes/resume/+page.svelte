<svelte:head>
  <title>Resume</title>
</svelte:head>

<nav>
  <a href=".">Home</a>
  <a href="projects">Projects</a>
  <a href="contact">Contact</a>
  <a class="current" href="resume">Resume</a>
  <a href="https://www.github.com/emaanbilal" target="_blank">GitHub</a>
</nav>

<main class="resume">
  <h1>Resume</h1>

  <section>
    <header>
      <p>
        Graduate student in Technology and Policy with interests in AI safety,
        policy, and responsible deployment of machine learning systems.
      </p>
    </header>
  </section>

  <section>
    <header>
      <h2>Education</h2>
    </header>
    <article>
      <header>
        <h3>Massachusetts Institute of Technology</h3>
      </header>
      <p>Master of Science in Technology and Policy</p>
      <p>
        <time datetime="2024-09">September 2024</time> –
        <time datetime="2026-05">May 2026</time>
      </p>
      <ul>
        <li>
          Coursework in machine learning, AI policy, and data visualization.
        </li>
        <li>Research on governance and safety of large-scale AI systems.</li>
      </ul>
    </article>
    <article>
      <header>
        <h3>LUMS, Pakistan</h3>
      </header>
      <p>Bachelor of Science in Computer Science</p>
      <p>
        <time datetime="2021-09">September 2020</time> –
        <time datetime="2024-05">May 2024</time>
      </p>
      <ul>
        <li>Graduated with honors.</li>
      </ul>
    </article>
  </section>

  <section>
    <header>
      <h2>Experience</h2>
    </header>

    <article>
      <header>
        <h3>AI Policy Research Intern – Example Policy Lab</h3>
      </header>
      <p>
        <time datetime="2024-06">June 2024</time> –
        <time datetime="2024-08">August 2024</time>
      </p>
      <ul>
        <li>Analyzed regulatory proposals related to foundation models.</li>
        <li>
          Built small interactive visualizations to communicate technical
          concepts.
        </li>
      </ul>
    </article>

    <article>
      <header>
        <h3>Software Engineer – Sample Tech Company</h3>
      </header>
      <p>
        <time datetime="2023-05">May 2023</time> –
        <time datetime="2024-05">May 2024</time>
      </p>
      <ul>
        <li>Developed web applications using HTML, CSS, and JavaScript.</li>
        <li>
          Collaborated with designers and researchers to improve usability and
          accessibility.
        </li>
      </ul>
    </article>
  </section>

  <section>
    <header>
      <h2>Skills</h2>
    </header>
    <article>
      <h3>Technical</h3>
      <ul>
        <li>Programming: Python, JavaScript</li>
        <li>Data: Pandas, data visualization, basic statistics</li>
        <li>Web: HTML, CSS, basic interaction design</li>
      </ul>
    </article>
    <article>
      <h3>Policy & Communication</h3>
      <ul>
        <li>Technical writing and policy analysis.</li>
        <li>Presenting complex topics to non-technical audiences.</li>
      </ul>
    </article>
  </section>
</main>