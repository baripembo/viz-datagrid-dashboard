<script>
	import * as d3 from 'd3';
	import { onMount } from 'svelte';

	export let data = [];
	export let title = null;
	export let valueFormat = d3.format('.2s');
	export let width = 100;
	export let barHeight = 16;
	export let barPadding = 8;
	export let nameWidth = 100;
	export let labelWidth = 50;
	export let textPadding = 10;

	let chartWidth, height, xScale, yScale;

	$: rankingData = data.sort((a, b) => b.value - a.value).slice(0,10);
	$: console.log(rankingData)

	const xAccessor = (d) => d.value;
	const yAccessor = (d) => d.name;

	$: rankingData && init()

	function init() {
		chartWidth = width - nameWidth - labelWidth;
		height = rankingData.length * (barHeight+barPadding);

		xScale = d3.scaleLinear()
	  	.domain([0, d3.max(rankingData, xAccessor)])
			.range([0, chartWidth]);

	  yScale = d3.scaleBand()
			.domain(rankingData.map(yAccessor))
	  	.range([0, height]);
	}
</script>


{#if title}
	<h3>{title}</h3>
{/if}

<div class='bar-container'>
	<svg viewBox='0 0 {width} {height}' preserveAspectRatio='none' {width} {height}>
		<g>
			{#each rankingData as d, i}
				<text class='name' x={nameWidth - textPadding} y={yScale(d.name)}>{d.name}</text>
				<rect 
					class='bar'
					x={nameWidth}
					y={yScale(d.name)}
					height={barHeight}
					width={xScale(d.value)}
				/>
				<text class='label' x={nameWidth + xScale(d.value) + textPadding} y={yScale(d.name)}>{valueFormat(d.value)}</text>
			{/each}
		</g>
	</svg>
</div>

<style lang='scss'>
	.bar-container {
		fill: #007CE0;
	}
	.axis path {
		color: #007CE0;
	}
	.label,
	.name {
		alignment-baseline: hanging;
	}
	.name {
		text-anchor: end;
	}
</style>
