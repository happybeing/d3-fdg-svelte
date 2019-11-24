<script>
	import NetworkGraph from './NetworkGraph.svelte';
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
	<h2>SvelteJS Demo of d3 Force</h2>

	<NetworkGraph graph={data}/>
	<!-- <Scatterplot points={data.a}/>
	<Scatterplot points={data.b}/>
	<Scatterplot points={data.c}/>
	<Scatterplot points={data.d}/> -->
</div>