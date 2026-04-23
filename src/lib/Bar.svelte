<script>
import * as d3 from 'd3';

let width = 400;
let height = 300;
export let data = [];

let margin = { top: 40, right: 150, bottom: 80, left: 60 };
let innerWidth  = width  - margin.left - margin.right;
let innerHeight = height - margin.top  - margin.bottom;
let xAxis, yAxis;

$: xScale = d3.scaleBand()
    .domain(data.map(d => d.label))
    .range([0, innerWidth])
    .padding(0.2);

$: yScale = d3.scaleLinear()
    .domain([0, d3.max(data, d => d.value) || 1])
    .range([innerHeight, 0]);

$: colorScale = d3.scaleOrdinal()
    .domain(data.map(d => d.label))
    .range(d3.quantize(d3.interpolateBlues, data.length));

$: if (xAxis && yAxis) {
    d3.select(xAxis).call(d3.axisBottom(xScale));
    d3.select(yAxis).call(
        d3.axisLeft(yScale)
            .tickFormat(d => Number.isInteger(d) ? d : "")
            .tickValues(d3.range(0, d3.max(data, d => d.value) + 1))
    );
}

$: maxBar = d3.greatest(data, d => d.value);


let selectedIndex = -1;

function toggleBar(index, event) {
    if (!event.key || event.key === "Enter") {
        selectedIndex = index;
    }
}

$: description = `A bar chart showing project counts by year. ${data.map(d => `${d.label}: ${d.value} projects`).join(', ')}.`;
let liveText = "";
    let showChart = true;

    function toggleView() {
        showChart = !showChart;
        liveText = showChart ? "Bar chart view shown." : "Table view shown.";
    }




</script>

<button
  on:click={toggleView}
  aria-pressed={!showChart}
  aria-label="Toggle between bar chart and table view"
  class="toggle-button">
    {showChart ? 'Show Table' : 'Show Chart'}
</button>

{#if showChart}
<div class="container">
<svg viewBox="0 0 {width} {height}"
  role="img"
  aria-labelledby="bar-title bar-desc">
  <title id="bar-title">Projects by Year</title>
  <desc id="bar-desc">{description}</desc>
        <text
            x={margin.left + innerWidth / 2}
            y={margin.top / 2}
            text-anchor="middle"
            class="chart-title">
            Projects per Year
        </text>
        <g transform="translate({margin.left}, {margin.top + innerHeight})"
       bind:this={xAxis} />
    <g transform="translate({margin.left}, {margin.top})"
       bind:this={yAxis} />
    <g transform="translate({margin.left}, {margin.top})">
            {#each data as d, index}
            <rect 
                class="bar"
                stroke="black"
                x={xScale(d.label)}
                y={yScale(d.value)}
                width={xScale.bandwidth()}
                height={innerHeight - yScale(d.value)}
                fill={colorScale(d.label)}
                opacity={selectedIndex === -1 || selectedIndex === index ? 1 : 0.45}
                tabindex="0"
                role="button"
                aria-label={`Select ${d.label}: ${d.value} projects`}
                on:click={(e) => toggleBar(index, e)}
                on:keyup={(e) => toggleBar(index, e)}
            />
            {/each}
            <text
    x={innerWidth / 2}
    y={innerHeight + margin.bottom - 40}
    text-anchor="middle"
    class="axis-label">
    Year
</text>

<text
    x={-(innerHeight / 2)}
    y={-margin.left + 30}
    text-anchor="middle"
    transform="rotate(-90)"
    class="axis-label">
    Number of Projects
</text>
        {#if maxBar}
    <!-- highlight outline around the tallest bar -->
    <rect
        x={xScale(maxBar.label)}
        y={yScale(maxBar.value)}
        width={xScale.bandwidth()}
        height={innerHeight - yScale(maxBar.value)}
        fill="none"
        stroke="currentColor"
        stroke-width="2"
        pointer-events="none"
    />
    <!-- leader line -->
    <line
        x1={xScale(maxBar.label) + xScale.bandwidth()}
        y1={yScale(maxBar.value) + (innerHeight - yScale(maxBar.value)) / 2}
        x2={xScale(maxBar.label) + xScale.bandwidth() + 30}
        y2={yScale(maxBar.value) + (innerHeight - yScale(maxBar.value)) / 2}
        stroke="currentColor"
        stroke-width="1"
    />
    <!-- annotation text at end of leader line -->
    <text
        x={xScale(maxBar.label) + xScale.bandwidth() + 35}
        y={yScale(maxBar.value) + (innerHeight - yScale(maxBar.value)) / 2}
        dominant-baseline="middle"
        class="annotation">
        Year with most projects
    </text>
{/if}


    </g>
</svg>
<p aria-live="polite" class="sr-only">{liveText}</p>
    <ul class="legend">
        {#each data as d}
            <li style="--color: {colorScale(d.label)}">
                <span class="swatch"></span>
                {d.label} <em>({d.value})</em>
            </li>
        {/each}
    </ul>
</div>
{:else}
<table aria-label="Table showing project counts by year" class="data-table">
    <caption>Projects by Year</caption>
    <thead>
        <tr>
          <th id="year-header" scope="col">Year</th>
          <th id="projects-header" scope="col">Projects</th>
        </tr>
      </thead>
    <tbody>
        {#each data as d, i}
          <tr>
            <th id="row-{i}" scope="row">{d.label}</th>
            <td aria-labelledby="row-{i} projects-header">{d.value}</td>
          </tr>
        {/each}
      </tbody>
  </table>
{/if}


<style>
  .container {
    display: flex;
    gap: 2rem;
    align-items: flex-start;
  }

  .legend {
    flex: 1;
    margin: 0;
    padding: 0;
    list-style: none; /* remove default bullets */
    display: flex;
    flex-direction: column;
    gap: 0.6rem;
  }

  .legend li {
    display: flex;
    align-items: center; /* swatch + text vertically aligned */
    gap: 0.6rem;
  }

  .swatch {
    width: 14px;
    height: 14px;
    background-color: var(--color);
    display: inline-block;
    border-radius: 3px;
    box-shadow: 0 0 0 1px color-mix(in oklch, var(--color), canvas 75%);
  }

  svg {
    max-width: 100%;
    height: auto;
    overflow: visible;
  }

  .bar {
    transition: 300ms;
    stroke: black;
    stroke-width: 1px;
    outline: none;
  }
.sr-only {
  position: absolute;
  left: -9999px;
  width: 1px;
  height: 1px;
  overflow: hidden;
}
.bar:focus-visible {
  stroke: white;
  stroke-width: 2px;
  stroke-dasharray: 4; 
  }


svg:hover .bar:not(:hover), .container:focus-within .bar:not(:focus-visible) { opacity: 0.5; }
.bar:focus-visible {
  stroke: white;
  stroke-width: 2px;
  stroke-dasharray: 4;
}



  @media (max-width: 800px) {
    .container {
      flex-direction: column;
      gap: 1.25rem;
    }
  }

  .chart-title {
    font-size: 1em;
    font-weight: bold;
    fill: currentColor;
}
.axis-label {
    font-size: 0.8em;
    fill: currentColor;
}
.annotation {
    font-size: 0.7em;
    fill: black;
    font-style: italic;
}

.data-table {
  margin-top: 1rem;
  margin-bottom: 1rem;
  border-collapse: collapse;
  width: 100%;
  max-width: 30em;
}

.data-table caption {
  font-weight: bold;
  margin-bottom: 0.5em;
  text-align: left;
}

.data-table th,
.data-table td {
  border: 1px solid #ccc;
  padding: 0.5em;
  text-align: left;
}

.data-table th {
  background-color: #f0f0f0;
}



</style>