<script>
	import NetworkGraphSvelteSVG from './NetworkGraphSvelteSVG.svelte';
	import NetworkGraphD3SVG from './NetworkGraphD3SVG.svelte';
	import NetworkGraphCanvas from './NetworkGraphCanvas.svelte';
	import NetworkGraphCanvasIdContext from './NetworkGraphCanvasIdContext.svelte';

	import data from './data.js';

	const graphList = [
		{ name: 'NetworkGraphSvelteSVG', component: NetworkGraphSvelteSVG },
		{ name: 'NetworkGraphD3SVG', component: NetworkGraphD3SVG },
		{ name: 'NetworkGraphCanvas', component: NetworkGraphCanvas },
		{ name: 'NetworkGraphCanvasIdContext', component: NetworkGraphCanvasIdContext },
	];

	let selectedGraph = graphList[2];

</script>

<style>
	.chart {
		width: 100%;
		max-width: 640px;
		height: calc(100% - 4em);
		min-height: 280px;
		max-height: 480px;
		margin: 0 auto;
	}
</style>

<div class="menu">
	{#each graphList as graph}
		<button on:click={() => { selectedGraph = graph; }}>{graph.name}</button>
	{/each}
</div>

<div class="chart">
	<svelte:component this={selectedGraph.component} graph={data} />
</div>
