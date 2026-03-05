<script>
import { page } from "$app/stores";
import { base } from "$app/paths";

let colorScheme = "light dark";
let localStorage = globalThis.localStorage ?? {};
if (localStorage.colorScheme) {
  colorScheme = localStorage.colorScheme;
}

$: localStorage.colorScheme = colorScheme;

let root = globalThis.document?.documentElement;
$: root?.style.setProperty("color-scheme", colorScheme);
let pages = [
  { url: "/", title: "Home", external: false },
  { url: "/projects", title: "Projects", external: false },
  { url: "/resume", title: "Resume", external: false },
  { url: "/contact", title: "Contact", external: false },
  { url: "https://github.com/emaanbilal", title: "Github", external: true },
];
</script>

<div class="layout">
  <label class="color-scheme-switch">
    Theme:
    <select bind:value={colorScheme}>
      <option value="light dark">Automatic</option>
      <option value="light">Light</option>
      <option value="dark">Dark</option>
    </select>
  </label>

  <nav>
    {#each pages as p}
      <a
        href={p.external ? p.url : base + p.url}
        class:current={
          !p.external &&
          (p.url === "/"
            ? $page.url.pathname === (base + "/")
            : $page.url.pathname.startsWith(base + p.url))
        }
        target={p.external ? "_blank" : undefined}
        rel={p.external ? "external noopener noreferrer" : undefined}
      >
        {p.title}
      </a>
    {/each}
  </nav>

  <slot />
</div>

<style>
  /* Ensures layout (and nav) get Svelte's scoped class when inspecting */
  .layout :global(nav) {
    /* nav styles live in static/style.css */
  }
</style>