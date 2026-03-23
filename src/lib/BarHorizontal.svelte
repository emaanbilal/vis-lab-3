<script>
  import * as d3 from "d3";

  export let data = [];
  export let title = "";

  let width = 400;
  let height = 240;
  let margin = { top: 40, right: 14, bottom: 52, left: 96 };

  $: innerWidth = width - margin.left - margin.right;
  $: innerHeight = height - margin.top - margin.bottom;

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

  $: maxTickCount = Math.min(d3.max(data, (d) => d.value) || 1, 10);

  $: if (xAxis && yAxis) {
    d3.select(xAxis).call(
      d3
        .axisBottom(xScale)
        .ticks(maxTickCount)
        .tickSizeOuter(0)
    );

    d3.select(yAxis).call(d3.axisLeft(yScale).tickSizeOuter(0));
  }
</script>

<div class="container">
  <svg viewBox="0 0 {width} {height}">
    <text
      x={margin.left + innerWidth / 2}
      y={margin.top / 2 + 4}
      text-anchor="middle"
      class="chart-title"
    >
      {title}
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

      {#if maxBar && maxBar.value > 0}
        <text
          x={xScale(maxBar.value) + 6}
          y={yScale(maxBar.label) + yScale.bandwidth() / 2}
          dominant-baseline="middle"
          text-anchor="start"
          class="annotation"
        >
          Most LOC: {maxBar.label}
        </text>
      {/if}

      <text
        x={innerWidth / 2}
        y={innerHeight + 36}
        text-anchor="middle"
        class="axis-label"
      >
        <tspan x={innerWidth / 2} dy="0">Lines of</tspan>
        <tspan x={innerWidth / 2} dy="1.1em">code</tspan>
      </text>
      <text
        x={-(innerHeight / 2)}
        y={-margin.left + 22}
        text-anchor="middle"
        transform="rotate(-90)"
        class="axis-label"
      >
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
    gap: 1rem;
    align-items: flex-start;
    max-width: 1100px;
    width: 100%;
    margin-inline: auto;
    justify-content: center;
  }

  .legend {
    flex: 1 1 200px;
    max-width: 220px;
    margin: 0;
    padding: 0;
    list-style: none;
    display: flex;
    flex-direction: column;
    gap: 0.45rem;
    font-size: 0.85rem;
  }

  .legend li {
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }

  .swatch {
    width: 12px;
    height: 12px;
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
    flex: 1 1 400px;
  }

  .chart-title {
    font-size: 0.78rem;
    font-weight: bold;
    fill: currentColor;
  }

  .axis-label {
    font-size: 0.68rem;
    fill: currentColor;
  }

  .annotation {
    font-size: 0.65rem;
    fill: currentColor;
    font-style: italic;
  }

  @media (max-width: 900px) {
    .container {
      flex-direction: column;
      gap: 1rem;
    }
  }
</style>
