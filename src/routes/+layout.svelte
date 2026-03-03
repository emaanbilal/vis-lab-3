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
    {url: "/", title: "Home"},
    {url: "/projects", title: "Projects"},
    {url: "/resume", title: "Resume"},
    {url: "/contact", title: "Contact"},
    {url: "https://github.com/emaanbilal", title: "Github"},
];
</script>

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
  <a href={base + p.url}
    class:current={p.url === "/" 
    ? $page.url.pathname === (base + "/") 
    : $page.url.pathname.startsWith(base + p.url)}
   target={p.url.startsWith("http") ? "_blank" : null}>
    {p.title}
  </a>
{/each}
</nav>

<slot />