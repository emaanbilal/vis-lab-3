<script>
  import rewind from "@mapbox/geojson-rewind";
  import { base } from "$app/paths";
  import { onMount } from "svelte";
  import * as d3 from "d3";

  /** Deterministic dummy eviction counts (cases) per neighborhood name */
  function evictionCases(name) {
    let h = 2166136261;
    for (let i = 0; i < name.length; i++) {
      h ^= name.charCodeAt(i);
      h = Math.imul(h, 16777619);
    }
    return 12 + (h >>> 0) % 214;
  }

  const vbW = 920;
  const vbH = 820;

  /** Offshore / non-residential polygons that blow up the map scale */
  const EXCLUDED_NAMES = new Set(["harbor islands"]);

  function dropExcludedNeighborhoods(collection) {
    if (!collection?.features) return collection;
    return {
      type: "FeatureCollection",
      features: collection.features.filter((f) => {
        const n = String(f.properties?.name ?? "")
          .trim()
          .toLowerCase();
        return !EXCLUDED_NAMES.has(n);
      }),
    };
  }

  let geojson = null;
  let loadError = null;
  let projection;
  let pathGen;
  let colorScale;
  let domainMax = 1;
  let hoveredId = null;
  let selectedFeature = null;

  $: selectedName = selectedFeature?.properties?.name ?? null;
  $: selectedCases = selectedName ? evictionCases(selectedName) : null;

  $: if (
    selectedFeature &&
    EXCLUDED_NAMES.has(
      String(selectedFeature.properties?.name ?? "").trim().toLowerCase(),
    )
  ) {
    selectedFeature = null;
  }

  onMount(async () => {
    try {
      const res = await fetch(`${base}/boston_neighborhoods.geojson`);
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      const raw = await res.json();
      // Source file has clockwise outers; D3 treats those as "the whole world minus
      // this hole", so fitExtent zooms to the globe and Boston looks like a flat smear.
      geojson = dropExcludedNeighborhoods(rewind(raw, true));
    } catch (e) {
      loadError = e?.message ?? String(e);
    }
  });

  $: if (geojson?.features?.length) {
    projection = d3.geoMercator().fitExtent(
      [
        [24, 24],
        [vbW - 24, vbH - 24],
      ],
      geojson,
    );
    pathGen = d3.geoPath(projection);
    const values = geojson.features.map((f) =>
      evictionCases(f.properties.name),
    );
    domainMax = d3.max(values) ?? 1;
    colorScale = d3
      .scaleSequential(d3.interpolateRgbBasis(["#1a3a52", "#3d7a8c", "#c45c4a", "#f4c14d"]))
      .domain([0, domainMax]);
  }

  function selectFeature(f) {
    selectedFeature = f;
  }

  function keyForFeature(f) {
    return f.properties?.OBJECTID ?? f.properties?.name;
  }
</script>

<svelte:head>
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="anonymous" />
  <link
    href="https://fonts.googleapis.com/css2?family=DM+Sans:ital,opsz,wght@0,9..40,400;0,9..40,600;0,9..40,700;1,9..40,400&family=Fraunces:ital,opsz,wght@0,9..144,500;0,9..144,700;1,9..144,500&display=swap"
    rel="stylesheet"
  />
</svelte:head>

<div class="shell">
  <aside class="panel panel-left" aria-live="polite">
    <p class="eyebrow">Greater Boston · neighborhood scale</p>
    <h1 class="title">Eviction filings</h1>
    <p class="lede">
      Census-style neighborhood boundaries (not ZIPs)—each polygon is a named place with
      elected local governance. Dummy case counts power the choropleth for now.
    </p>

    <div class="selection">
      {#if selectedName}
        <p class="label">Selected area</p>
        <p class="place-name">{selectedName}</p>
        <p class="stat">
          <span class="stat-label">Dummy filings (12 mo.)</span>
          <span class="stat-value">{selectedCases}</span>
        </p>
      {:else}
        <p class="hint">Click a neighborhood on the map to name it here.</p>
      {/if}
    </div>

    <div class="legend-mini">
      <span class="legend-track" aria-hidden="true"></span>
      <div class="legend-labels">
        <span>Lower</span>
        <span>Higher</span>
      </div>
    </div>
  </aside>

  <section class="panel panel-map" aria-label="Boston neighborhoods map">
    {#if loadError}
      <p class="error">Could not load map data: {loadError}</p>
    {:else if !geojson}
      <p class="loading">Drawing boundaries…</p>
    {:else}
      <div class="map-frame">
        <svg
          class="map-svg"
          viewBox="0 0 {vbW} {vbH}"
          preserveAspectRatio="xMidYMid meet"
          role="img"
          aria-label="Choropleth of Boston neighborhoods by dummy eviction filings"
        >
          <rect class="ocean" width={vbW} height={vbH} rx="4" />

          {#each geojson.features as f (keyForFeature(f))}
            {@const id = keyForFeature(f)}
            {@const cases = evictionCases(f.properties.name)}
            {@const isSel = selectedFeature && keyForFeature(selectedFeature) === id}
            <path
              class="tract"
              class:selected={isSel}
              class:hovered={hoveredId === id}
              d={pathGen(f)}
              fill={colorScale(cases)}
              stroke="rgba(12, 18, 28, 0.55)"
              stroke-width={isSel ? 2.2 : 1}
              tabindex="0"
              role="button"
              aria-pressed={isSel ? "true" : "false"}
              aria-label="{f.properties.name}, {cases} dummy filings"
              on:click={() => selectFeature(f)}
              on:keydown={(e) => {
                if (e.key === "Enter" || e.key === " ") {
                  e.preventDefault();
                  selectFeature(f);
                }
              }}
              on:mouseenter={() => (hoveredId = id)}
              on:mouseleave={() => (hoveredId = null)}
            />
          {/each}

          <text x={vbW / 2} y={36} class="map-title" text-anchor="middle">
            Neighborhood boundaries
          </text>
          <text x={vbW / 2} y={56} class="map-sub" text-anchor="middle">
            Dummy eviction-weighted fill · click to inspect
          </text>
        </svg>
      </div>

      <div class="legend-bar" aria-hidden="true">
        <span>Filings (dummy)</span>
        <div class="bar-wrap">
          <div class="bar"></div>
        </div>
        <div class="ticks">
          <span>0</span>
          <span>{Math.round(domainMax * 0.5)}</span>
          <span>{domainMax}+</span>
        </div>
      </div>
    {/if}
  </section>
</div>

<style>
  .shell {
    display: grid;
    grid-template-columns: minmax(280px, 1fr) minmax(360px, 1.35fr);
    gap: clamp(1rem, 3vw, 2.5rem);
    align-items: stretch;
    min-height: min(88vh, 920px);
    font-family: "DM Sans", system-ui, sans-serif;
    color: #e8e4dc;
    --ink: #0c1018;
    --panel: #121826;
    --accent: #e8b54a;
  }

  .panel {
    border-radius: 14px;
    padding: clamp(1.25rem, 3vw, 2rem);
    box-sizing: border-box;
  }

  .panel-left {
    background:
      radial-gradient(120% 80% at 10% 0%, rgba(232, 181, 74, 0.12), transparent 55%),
      linear-gradient(165deg, #141a26 0%, #0c1018 48%, #101824 100%);
    border: 1px solid rgba(255, 255, 255, 0.06);
    box-shadow:
      0 24px 48px rgba(0, 0, 0, 0.35),
      inset 0 1px 0 rgba(255, 255, 255, 0.04);
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
  }

  .eyebrow {
    font-size: 0.72rem;
    letter-spacing: 0.14em;
    text-transform: uppercase;
    color: rgba(232, 228, 220, 0.55);
    margin: 0 0 0.75rem;
  }

  .title {
    font-family: "Fraunces", Georgia, serif;
    font-weight: 700;
    font-size: clamp(1.75rem, 4vw, 2.35rem);
    line-height: 1.1;
    margin: 0 0 0.75rem;
    color: #f5f1e8;
  }

  .lede {
    margin: 0 0 1.75rem;
    font-size: 0.95rem;
    line-height: 1.55;
    color: rgba(232, 228, 220, 0.78);
    max-width: 38ch;
  }

  .selection {
    flex: 1;
    min-height: 12rem;
    padding: 1.25rem 1.35rem;
    border-radius: 10px;
    background: rgba(0, 0, 0, 0.22);
    border: 1px solid rgba(255, 255, 255, 0.06);
  }

  .label {
    font-size: 0.7rem;
    letter-spacing: 0.12em;
    text-transform: uppercase;
    color: rgba(232, 181, 74, 0.85);
    margin: 0 0 0.35rem;
  }

  .place-name {
    font-family: "Fraunces", Georgia, serif;
    font-size: clamp(1.85rem, 4.5vw, 2.75rem);
    font-weight: 500;
    margin: 0 0 1rem;
    line-height: 1.05;
    color: #fff;
    text-wrap: balance;
  }

  .stat {
    display: flex;
    flex-direction: column;
    gap: 0.25rem;
  }

  .stat-label {
    font-size: 0.85rem;
    color: rgba(232, 228, 220, 0.6);
  }

  .stat-value {
    font-size: 1.65rem;
    font-weight: 700;
    font-variant-numeric: tabular-nums;
    color: var(--accent);
  }

  .hint {
    margin: 0;
    font-size: 1.05rem;
    line-height: 1.45;
    color: rgba(232, 228, 220, 0.55);
    font-style: italic;
  }

  .legend-mini {
    margin-top: 1.5rem;
  }

  .legend-track {
    display: block;
    height: 6px;
    border-radius: 3px;
    background: linear-gradient(90deg, #1a3a52, #3d7a8c, #c45c4a, #f4c14d);
  }

  .legend-labels {
    display: flex;
    justify-content: space-between;
    font-size: 0.72rem;
    margin-top: 0.35rem;
    color: rgba(232, 228, 220, 0.45);
  }

  .panel-map {
    background:
      radial-gradient(ellipse 90% 60% at 70% 20%, rgba(61, 122, 140, 0.15), transparent),
      linear-gradient(145deg, #1a2230 0%, #0e131c 100%);
    border: 1px solid rgba(255, 255, 255, 0.07);
    box-shadow: 0 28px 56px rgba(0, 0, 0, 0.4);
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }

  .map-frame {
    position: relative;
    flex: 1;
    min-height: 320px;
    border-radius: 10px;
    overflow: hidden;
    border: 1px solid rgba(255, 255, 255, 0.05);
  }

  .map-svg {
    width: 100%;
    height: 100%;
    display: block;
  }

  .ocean {
    fill: #0a1624;
  }

  .tract {
    cursor: pointer;
    transition:
      filter 0.2s ease,
      opacity 0.2s ease;
  }

  .tract:hover,
  .tract.hovered {
    filter: brightness(1.12);
  }

  .tract:focus {
    outline: 2px solid var(--accent);
    outline-offset: 0;
  }

  .tract.selected {
    filter: drop-shadow(0 0 8px rgba(232, 181, 74, 0.45));
  }

  .map-title {
    fill: rgba(232, 228, 220, 0.9);
    font-family: "Fraunces", Georgia, serif;
    font-size: 18px;
    font-weight: 600;
    pointer-events: none;
  }

  .map-sub {
    fill: rgba(232, 228, 220, 0.45);
    font-family: "DM Sans", sans-serif;
    font-size: 11px;
    pointer-events: none;
  }

  .legend-bar {
    font-size: 0.78rem;
    color: rgba(232, 228, 220, 0.55);
  }

  .legend-bar .bar-wrap {
    margin-top: 0.35rem;
    height: 10px;
    border-radius: 5px;
    overflow: hidden;
    border: 1px solid rgba(255, 255, 255, 0.08);
  }

  .legend-bar .bar {
    height: 100%;
    background: linear-gradient(90deg, #1a3a52, #3d7a8c, #c45c4a, #f4c14d);
  }

  .ticks {
    display: flex;
    justify-content: space-between;
    margin-top: 0.25rem;
    font-variant-numeric: tabular-nums;
  }

  .loading,
  .error {
    margin: 0;
    padding: 2rem;
    color: rgba(232, 228, 220, 0.7);
  }

  .error {
    color: #f4a09a;
  }

  @media (max-width: 900px) {
    .shell {
      grid-template-columns: 1fr;
      min-height: unset;
    }

    .panel-map {
      order: -1;
    }
  }
</style>
