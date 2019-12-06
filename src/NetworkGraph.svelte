<h2>d3 Force Directed Graph in Sveltejs - svg</h2>
<p>Uses SVG for the display and DOM elements for mouse handling.</p>
<script>
	import { onMount } from 'svelte';

	import { scaleLinear, scaleOrdinal } from 'd3-scale';
	import { schemeCategory10 } from 'd3-scale-chromatic';
	import { select, selectAll } from 'd3-selection';
	import { drag } from 'd3-drag';
	import { forceSimulation, forceLink, forceManyBody, forceCenter } from 'd3-force';

	import {event as currentEvent} from 'd3-selection'  // Needed to get drag working, see: https://github.com/d3/d3/issues/2733
	let d3 = { scaleLinear, scaleOrdinal, schemeCategory10, select, selectAll, drag,  forceSimulation, forceLink, forceManyBody, forceCenter }

	export let graph;

	let svg;
	let width = 500;
	let height = 600;

	const padding = { top: 20, right: 40, bottom: 40, left: 25 };

	$: xScale = scaleLinear()
		.domain([0, 20])
		.range([padding.left, width - padding.right]);

	$: yScale = scaleLinear()
		.domain([0, 12])
		.range([height - padding.bottom, padding.top]);

	$: d3yScale = scaleLinear()
		.domain([0, height])
		.range([height, 0]);

	$: links = graph.links.map(d => Object.create(d));
	$: nodes = graph.nodes.map(d => Object.create(d));  

	const colourScale = d3.scaleOrdinal(d3.schemeCategory10);

	onMount(() => {

	simulation = d3.forceSimulation(nodes)
		.force("link", d3.forceLink(links).id(d => d.id))
		.force("charge", d3.forceManyBody())
		.force("center", d3.forceCenter(width / 2, height / 2))
		.on('tick', function ticked() {		
					simulation.tick()
					nodes = [...nodes]
			links = [...links]
				});
	d3.select(svg)
		.call(d3.drag()
			.container(svg)
			.subject(dragsubject)
			.on("start", dragstarted)
			.on("drag", dragged)
			.on("end", dragended));
	});

	let simulation
	function dragstarted() {
		if (!currentEvent.active) simulation.alphaTarget(0.3).restart();
		currentEvent.subject.fx = currentEvent.subject.x;
		currentEvent.subject.fy = currentEvent.subject.y;
	}

	function dragged() {
		currentEvent.subject.fx = currentEvent.x;
		currentEvent.subject.fy = currentEvent.y;
	}

	function dragended() {
		if (!currentEvent.active) simulation.alphaTarget(0);
		currentEvent.subject.fx = null;
		currentEvent.subject.fy = null;
	}

	function dragsubject() {
		return simulation.find(currentEvent.x, currentEvent.y);
	}

	function resize() {
		({ width, height } = svg.getBoundingClientRect());
	}


</script>

<svelte:window on:resize='{resize}'/>

<!-- SVG was here -->
<svg bind:this={svg} width='{width}' height='{height}'>  
	{#each links as link}
    <g stroke='#999' stroke-opacity='0.6'>
      <line x1='{link.source.x}' y1='{d3yScale(link.source.y)}' 
            x2='{link.target.x}' y2='{d3yScale(link.target.y)}'
            transform='translate(0 {height}) scale(1 -1)'>
            <title>{link.source.id}</title>
      </line>
    </g>
	{/each}

	{#each nodes as point}
    <circle class='node' r='5' fill='{colourScale(point.group)}' cx='{point.x}' cy='{d3yScale(point.y)}'
     transform='translate(0 {height}) scale(1 -1)'>
    <title>{point.id}</title></circle>
	{/each}

</svg>

<style>
	svg {
		float: left;
	}

	circle {
		stroke: #fff;
    stroke-width: 1.5;
	}

	.tick line {
		stroke: #ddd;
		stroke-dasharray: 2;
	}

	text {
		font-size: 12px;
		fill: #999;
	}

	.x-axis text {
		text-anchor: middle;
	}

	.y-axis text {
		text-anchor: end;
	}
</style>