<script>
	import { onMount } from 'svelte';
 
  import * as d3 from "d3";
  import { scaleLinear, scaleOrdinal } from 'd3-scale';
  import { select, selectAll } from "d3-selection";
	import { drag } from 'd3-drag';
	import { force } from 'd3-force-graph';

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

	$: xTicks = width > 180 ?
		[0, 4, 8, 12, 16, 20] :
		[0, 10, 20];

	$: yTicks = height > 180 ?
		[0, 2, 4, 6, 8, 10, 12] :
		[0, 4, 8, 12];

	onMount(network);

  const links = graph.links.map(d => Object.create(d));
  const nodes = graph.nodes.map(d => Object.create(d));

	function resize() {
		({ width, height } = svg.getBoundingClientRect());
  }

  function color()
  {
    const scale = scaleOrdinal(); // ??? schemeCategory10
    return d => scale(d.group);
  }

  function dragstarted(d) {
    if (!d3.event.active) simulation.alphaTarget(0.3).restart();
    d.fx = d.x;
    d.fy = d.y;
  }
  
  function dragged(d) {
    d.fx = d3.event.x;
    d.fy = d3.event.y;
  }
  
  function dragended(d) {
    if (!d3.event.active) simulation.alphaTarget(0);
    d.fx = null;
    d.fy = null;
  }
    
    drag()
      .on("start", dragstarted)
      .on("drag", dragged)
      .on("end", dragended);

  const simulation = d3.forceSimulation(nodes)
      .force("link", d3.forceLink(links).id(d => d.id))
      .force("charge", d3.forceManyBody())
      .force("center", d3.forceCenter(width / 2, height / 2));

// let svg
// TODO integrate with sveltejs svg element
  // const svg = d3.select('body').append('svg')
  
  // const svg = d3.create("svg")
      // .attr("viewBox", [0, 0, width, height]);

  const fromGraph = function(d){ return graph.d}

  function network() {
  resize()

  // const link = svg.append("g")
  //     .attr("stroke", "#999")
  //     .attr("stroke-opacity", 0.6)
  //   .selectAll("line")  // ???
  //   .join("line")
  //     .attr("stroke-width", d => Math.sqrt(d.value));

  // const node = svg.append("g")
  //     .attr("stroke", "#fff")
  //     .attr("stroke-width", 1.5)
  //   .selectAll("circle")
  //   .data(nodes)
  //   .join("circle")
  //     .attr("r", 5)
  //     .attr("fill", color)
  //     .call(drag(simulation));

  // node.append("title")
  //     .text(d => d.id);

  // simulation.on("tick", () => {
  //   link
  //       .attr("x1", d => d.source.x)
  //       .attr("y1", d => d.source.y)
  //       .attr("x2", d => d.target.x)
  //       .attr("y2", d => d.target.y);

  //   node
  //       .attr("cx", d => d.x)
  //       .attr("cy", d => d.y);
  // });

  // invalidation.then(() => simulation.stop());

  // return svg.node();
  }
</script>

<svelte:window on:resize='{resize}'/>

<!-- SVG was here -->
<svg bind:this={svg}>
	<g class='axis y-axis'>
		{#each yTicks as tick}
			<g class='tick tick-{tick}' transform='translate(0, {yScale(tick)})'>
				<line x1='{padding.left}' x2='{xScale(22)}'/>
				<text x='{padding.left - 8}' y='+4'>{tick}</text>
			</g>
		{/each}
	</g>

	<g class='axis x-axis'>
		{#each xTicks as tick}
			<g class='tick' transform='translate({xScale(tick)},0)'>
				<line y1='{yScale(0)}' y2='{yScale(13)}'/>
				<text y='{height - padding.bottom + 16}'>{tick}</text>
			</g>
		{/each}
	</g>

		<circle cx='{xScale(5)}' cy='{yScale(5)}' r='5'/>
	{#each nodes as point}
		<circle cx='{xScale(point.x)}' cy='{yScale(point.y)}' r='5'/>
		<circle cx='{xScale(5)}' cy='{yScale(5)}' r='5'/>
	{/each}
</svg>

<style>
	svg {
		width: 50%;
		height: 50%;
		float: left;
	}

	circle {
		fill: orange;
		fill-opacity: 0.6;
		stroke: rgba(0,0,0,0.5);
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
