<script>
import { base } from '$app/paths';
import { onMount } from 'svelte';
import * as d3 from 'd3';
import BarHorizontal from '$lib/BarHorizontal.svelte';

let locData = [];
let wrangled = [];

onMount(async () => {
    locData = await d3.csv(`${base}/loc.csv`, row => ({
        ...row,
        line: Number(row.line),
        length: Number(row.length),
        depth: Number(row.depth)
    }));
});

// Each row in loc.csv represents one line of code.
// Roll up total lines by language (in this dataset, `type` holds the language).
$: wrangled = d3
    .rollups(locData, v => v.length, d => d.type)
    .map(([type, count]) => ({ label: type, value: count }));
</script>

<h1>Meta</h1>
<p>This is a page about the meta-level of my project.</p>
<section>
  <BarHorizontal data={wrangled} />
</section>
