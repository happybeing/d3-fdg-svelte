<script>
	import GraphSvelteSVG from './NetworkGraphSvelteSVG.svelte';
	import GraphD3SVG from './NetworkGraphD3SVG.svelte';
	import GraphCanvas from './NetworkGraphCanvas.svelte';
	import GraphCanvasIdContext from './NetworkGraphCanvasIdContext.svelte';

	import data from './data.js';
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

<div class="chart">
	<GraphD3SVG graph={data}/>
</div>