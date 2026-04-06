<script>
import { base } from '$app/paths';
import { onMount, tick } from 'svelte';
import * as d3 from 'd3';
import { computePosition, autoPlacement, offset } from '@floating-ui/dom';
import BarHorizontal from '$lib/BarHorizontal.svelte';
import LineChart from '$lib/LineChart.svelte';

let locData = [];
let commits = [];
let clickedCommits = [];
let svg;
let brushG;
let hoveredIndex = -1;
let hoveredCommit = {};
$: hoveredCommit = commits[hoveredIndex] ?? hoveredCommit ?? {};

let commitTooltip;
let tooltipPosition = { x: 0, y: 0 };

async function dotInteraction(index, evt) {
	if (evt.type === 'click') {
		const commit = commits[index];
		if (!clickedCommits.includes(commit)) {
			clickedCommits = [...clickedCommits, commit];
		} else {
			clickedCommits = clickedCommits.filter((c) => c !== commit);
		}
		return;
	}
	if (evt.type === 'mouseenter') {
		hoveredIndex = index;
		await tick();
		const hoveredDot = evt.target;
		if (commitTooltip) {
			tooltipPosition = await computePosition(hoveredDot, commitTooltip, {
				strategy: 'fixed',
				middleware: [offset(5), autoPlacement()]
			});
		}
	} else if (evt.type === 'mouseleave') {
		hoveredIndex = -1;
	}
}

let width = 720;
let height = 320;
let margin = { top: 24, right: 20, bottom: 40, left: 48 };

$: usableArea = (() => {
	const ua = {
		top: margin.top,
		right: width - margin.right,
		bottom: height - margin.bottom,
		left: margin.left
	};
	return {
		...ua,
		width: ua.right - ua.left,
		height: ua.bottom - ua.top
	};
})();

let xAxis;
let yAxis;
let yAxisGridlines;
let brush = d3.brush();
let linesByDate = [];

$: {
	const rolled = d3.rollups(
		locData,
		v => v.length,
		d => d3.timeDay.floor(d.datetime)
	).map(([date, count]) => ({ date, count }));

	const [minDate, maxDate] = d3.extent(rolled, d => d.date);
	const allDays = d3.timeDays(minDate, d3.timeDay.offset(maxDate, 1));

	linesByDate = allDays.map(date => ({
		date,
		count: rolled.find(d => d.date.getTime() === date.getTime())?.count ?? 0
	}));
}



$: [minDate, maxDate] =
	commits.length > 0
		? d3.extent(commits, (d) => d.datetime)
		: [new Date(), new Date()];
$: maxDatePlusOne = (() => {
	const d = new Date(maxDate);
	d.setDate(d.getDate() + 1);
	return d;
})();

$: xScale = d3
	.scaleTime()
	.domain([minDate, maxDatePlusOne])
	.range([usableArea.left, usableArea.right])
	.nice();

$: yScale = d3
	.scaleLinear()
	.domain([24, 0])
	.range([usableArea.bottom, usableArea.top]);

$: lineExtent =
	commits.length > 0 ? d3.extent(commits, (d) => d.totalLines) : [1, 1];
$: [lineMin, lineMaxRaw] = lineExtent;
$: lineMax = lineMin === lineMaxRaw ? lineMin + 1 : lineMaxRaw;
$: rScale = d3.scaleSqrt().domain([lineMin, lineMax]).range([5, 30]);

$: {
	if (xAxis && yAxis && yAxisGridlines) {
		d3.select(yAxisGridlines).call(
			d3.axisLeft(yScale).tickFormat("").tickSize(-usableArea.width)
		);
		d3.select(xAxis).call(d3.axisBottom(xScale));
		d3.select(yAxis).call(
			d3.axisLeft(yScale).tickFormat((d) => String(d % 24).padStart(2, "0") + ":00")
		);
	}
}

onMount(async () => {
locData = await d3.csv(`${base}/loc.csv`, row => ({
	...row,
	line: Number(row.line),
	depth: Number(row.depth),
	length: Number(row.length),
	date: new Date(row.date + "T00:00" + row.timezone),
	datetime: new Date(row.datetime)
}));
commits = d3.groups(locData, d => d.commit).map(([commit, lines]) => {
	let first = lines[0];
	let {author, date, time, timezone, datetime} = first;
	let ret = {
		id: commit,
		url: "https://github.com/vis-society/lab-7/commit/" + commit,
		author, date, time, timezone, datetime,
		hourFrac: datetime.getHours() + datetime.getMinutes() / 60,
		totalLines: lines.length,
		lines: lines
	};

	return ret;
});
commits = d3.sort(commits, (d) => -d.totalLines);

});

// Each row in loc.csv represents one line of code (`type` = language).
$: allLanguageLabels = Array.from(new Set(locData.map((d) => d.type)));
$: selectedLines =
	selectedCommits.length > 0 ? selectedCommits.flatMap((c) => c.lines) : locData;
$: languageCounts = d3.rollup(
	selectedLines,
	(v) => v.length,
	(d) => d.type
);
$: barData = allLanguageLabels.map((label) => ({
	label,
	value: languageCounts.get(label) ?? 0
}));
$: barTitle =
	selectedCommits.length > 0
		? 'Lines by language (selected commits)'
		: 'Total lines of code by language (website)';

$: brushSelection = null;

function brushed (evt) {
	brushSelection = evt.selection;
}

function isCommitBrushed (commit) {
	if (!brushSelection) {
		return false;
	}
	let min = {x: brushSelection[0][0], y: brushSelection[0][1]};
	let max = {x: brushSelection[1][0], y: brushSelection[1][1]};
	let x = xScale(commit.datetime);
	let y = yScale(commit.hourFrac);
	return x >= min.x && x <= max.x && y >= min.y && y <= max.y;
}
$: brushedCommits = brushSelection ? commits.filter(isCommitBrushed) : [];

$: selectedCommits = Array.from(new Set([...clickedCommits, ...brushedCommits]));



$: {
	if (brushG) {
		brush = brush.extent([
			[usableArea.left, usableArea.top],
			[usableArea.right, usableArea.bottom]
		]);
		brush.on('start brush end', brushed);
		d3.select(brushG).call(brush);
	}
	if (svg) d3.select(svg).selectAll('.dots').raise();
}

</script>

<h1>Meta</h1>
<p>This is a page about the meta-level of my project.</p>

<section class="commit-scatter">
	<h2>Commits over time</h2>
	<svg
		bind:this={svg}
		class="scatter-svg"
		viewBox="0 0 {width} {height}"
		role="img"
		aria-label="Scatter plot of commits by date and time of day"
	>
		<g
			class="gridlines"
			transform="translate({usableArea.left}, 0)"
			bind:this={yAxisGridlines}
		/>
		<g transform="translate(0, {usableArea.bottom})" bind:this={xAxis} />
		<g transform="translate({usableArea.left}, 0)" bind:this={yAxis} />
		<g class="brush" bind:this={brushG} />
		<g class="dots">
			{#each commits as commit, index (commit.id)}
				<circle
					class:selected={selectedCommits.includes(commit)}
					cx={xScale(commit.datetime)}
					cy={yScale(commit.hourFrac)}
					r={rScale(commit.totalLines)}
					fill="steelblue"
					fill-opacity="0.65"
					data-index={index}
					on:click={(evt) => dotInteraction(index, evt)}
					on:mouseenter={(evt) => dotInteraction(index, evt)}
					on:mouseleave={(evt) => dotInteraction(index, evt)}
				/>
			{/each}
		</g>
	</svg>

	<dl
		class="info tooltip"
		bind:this={commitTooltip}
		hidden={hoveredIndex === -1}
		style:left="{tooltipPosition.x}px"
		style:top="{tooltipPosition.y}px"
	>
		<dt>Commit</dt>
		<dd>
			<a href={hoveredCommit.url} target="_blank" rel="noreferrer">{hoveredCommit.id}</a>
		</dd>

		<dt>Date</dt>
		<dd>
			{hoveredCommit.datetime?.toLocaleString('en', { dateStyle: 'full' })}
		</dd>

		<dt>Time</dt>
		<dd>
			{hoveredCommit.datetime?.toLocaleString('en', { timeStyle: 'short' })}
		</dd>

		<dt>Author</dt>
		<dd>{hoveredCommit.author}</dd>

		<dt>Lines edited</dt>
		<dd>{hoveredCommit.totalLines}</dd>
	</dl>
	<LineChart data={linesByDate} />
</section>

<section class="meta-bar">
	<h2>Languages</h2>
	<BarHorizontal data={barData} title={barTitle} />
</section>

<style>
	.commit-scatter {
		margin-top: 2.5rem;
		max-width: 1100px;
		margin-inline: auto;
	}

	.commit-scatter h2 {
		margin: 0 0 0.35rem;
		font-size: 1.25rem;
	}

	.scatter-note {
		margin: 0 0 1rem;
		color: color-mix(in oklch, canvasText 72%, canvas);
		font-size: 0.95rem;
	}

	.scatter-svg {
		width: 100%;
		max-width: 720px;
		height: auto;
		overflow: visible;
		display: block;
	}

	.gridlines {
		stroke-opacity: 0.2;
	}

	.scatter-svg circle {
		transition: 200ms;
	}

	.scatter-svg circle:hover:not(.selected) {
		fill: darkgreen;
	}

	.scatter-svg circle.selected {
		fill: var(--color-accent, oklch(65% 50% 0));
	}

	dl.info {
		display: grid;
		grid-template-columns: auto 1fr;
		gap: 0.35rem 1rem;
		margin: 0;
		padding: 0.75rem 1rem;
		font-size: 0.9rem;
		transition-duration: 500ms;
		transition-property: opacity, visibility;
	}

	dl.info dt {
		margin: 0;
		font-weight: 600;
		color: color-mix(in oklch, canvasText 55%, canvas);
	}

	dl.info dd {
		margin: 0;
	}

	dl.info[hidden]:not(:hover):not(:focus-within) {
		opacity: 0;
		visibility: hidden;
	}

	dl.tooltip {
		position: fixed;
		z-index: 20;
		max-width: min(22rem, calc(100vw - 2rem));
		pointer-events: auto;
		background-color: oklch(100% 0 0 / 88%);
		border-radius: 8px;
		box-shadow: 0 4px 24px oklch(0% 0 0 / 12%);
		backdrop-filter: blur(8px);
	}

	.meta-bar {
		margin-top: 2.5rem;
		max-width: 1100px;
		margin-inline: auto;
	}

	.meta-bar h2 {
		margin: 0 0 0.75rem;
		font-size: 1.25rem;
	}
</style>
