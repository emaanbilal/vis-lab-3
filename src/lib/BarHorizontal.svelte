<script>
  import * as d3 from "d3";
  export let data = [];

  let width = 460;
  let height = 320;
  let margin = { top: 56, right: 20, bottom: 60, left: 105 };

  let innerWidth = width - margin.left - margin.right;
  let innerHeight = height - margin.top - margin.bottom;

  let xAxis;
  let yAxis;

  let xScale;
  let yScale;
  let colorScale;
  let maxBar = null;

  $: {
    let labels = data.map((d) => d.label);

    xScale = d3
      .scaleLinear()
      .domain([0, d3.max(data, (d) => d.value) || 1])
      .range([0, innerWidth]);

    yScale = d3
      .scaleBand()
      .domain(labels)
      .range([0, innerHeight])
      .padding(0.2);

    colorScale = d3.scaleOrdinal(d3.schemeTableau10).domain(labels);
    maxBar = d3.greatest(data, (d) => d.value);
  }

  $: if (xAxis && yAxis) {
    d3.select(xAxis).call(
      d3
        .axisBottom(xScale)
        .ticks(Math.min(6, Math.max(2, data.length)))
        .tickSizeOuter(0)
    );

    d3.select(yAxis).call(d3.axisLeft(yScale).tickSizeOuter(0));
  }
</script>

<div class="container">
  <svg viewBox="0 0 {width} {height}">
    <text
      x={margin.left + innerWidth / 2}
      y={margin.top / 2}
      text-anchor="middle"
      class="chart-title"
    >
      Total Lines of Code by Language
    </text>

    <g transform="translate({margin.left}, {margin.top})">
      <g bind:this={xAxis} transform={`translate(0, ${innerHeight})`} />
      <g bind:this={yAxis} />

      {#each data as d (d.label)}
        <g>
          <rect
            x={0}
            y={yScale(d.label)}
            width={xScale(d.value)}
            height={yScale.bandwidth()}
            fill={colorScale(d.label)}
          />

          {#if maxBar && d.label === maxBar.label}
            <!-- Highlight the max language bar -->
            <rect
              x={0}
              y={yScale(d.label)}
              width={xScale(d.value)}
              height={yScale.bandwidth()}
              fill="none"
              stroke="currentColor"
              stroke-width="2"
            />
          {/if}
        </g>
      {/each}

      {#if maxBar}
        <!-- Annotation -->
        <line
          x1={xScale(maxBar.value)}
          y1={yScale(maxBar.label) + yScale.bandwidth() / 2}
          x2={xScale(maxBar.value) + 28}
          y2={yScale(maxBar.label) + yScale.bandwidth() / 2}
          stroke="currentColor"
          stroke-width="1"
        />
        <text
          x={xScale(maxBar.value) + 34}
          y={yScale(maxBar.label) + yScale.bandwidth() / 2}
          dominant-baseline="middle"
          class="annotation"
        >
          Most LOC: {maxBar.label}
        </text>
      {/if}

      <!-- Axis labels -->
      <text x={innerWidth / 2} y={innerHeight + 46} text-anchor="middle" class="axis-label">
        Lines of code
      </text>
<text
    x={-(innerHeight / 2)}
    y={-margin.left + 30}
    text-anchor="middle"
    transform="rotate(-90)"
    class="axis-label">
    Language
</text>
    </g>
  </svg>

  <ul class="legend">
    {#each data as d (d.label)}
      <li style="--color: {colorScale(d.label)}">
        <span class="swatch"></span>
        {d.label} <em>({d.value})</em>
      </li>
    {/each}
  </ul>
</div>

<style>
  .container {
    display: flex;
    gap: 1.25rem;
    align-items: flex-start;
    max-width: 1100px;
    width: 100%;
    margin-inline: auto;
    justify-content: center;
  }

  .legend {
    flex: 1 1 260px;
    max-width: 280px;
    margin: 0;
    padding: 0;
    list-style: none;
    display: flex;
    flex-direction: column;
    gap: 0.6rem;
  }

  .legend li {
    display: flex;
    align-items: center;
    gap: 0.6rem;
  }

  .swatch {
    width: 14px;
    height: 14px;
    background-color: var(--color);
    display: inline-block;
    border-radius: 3px;
    box-shadow: 0 0 0 1px
      color-mix(in oklch, var(--color), canvas 75%);
    flex-shrink: 0;
  }

  svg {
    max-width: 100%;
    height: auto;
    overflow: visible;
    flex: 1 1 520px;
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
    font-size: 0.75em;
    fill: black;
    font-style: italic;
  }

  @media (max-width: 900px) {
    .container {
      flex-direction: column;
      gap: 1.25rem;
    }
  }
</style>

