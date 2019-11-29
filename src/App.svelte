<script>
	import NetworkGraph from './NetworkGraph.svelte';
	import NetworkGraphCanvas from './NetworkGraphCanvas.svelte';
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
	<h2>d3 Force Directed Graph - canvas</h2>

	<!-- <NetworkGraph graph={data}/> -->
	<NetworkGraphCanvas graph={data}/>
</div>