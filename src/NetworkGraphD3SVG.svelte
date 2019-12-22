<h2>d3 Force Directed Graph in Sveltejs - D3 created svg (zoom)</h2>
<p>Uses D3 created SVG and DOM elements, and D3 mouse handling.</p>

<p>Features: hover, drag nodes, zoom and pan with mouse
or touch screen. Note: on small touch screens you may need to 
increase hit radius to hit a node with a 'fat finger'!</p>
<script>
	import { onMount } from 'svelte';

	import { scaleLinear, scaleOrdinal } from 'd3-scale';
    import { zoom, zoomIdentity } from 'd3-zoom';
	import { schemeCategory10 } from 'd3-scale-chromatic';
	import { select, selectAll } from 'd3-selection';
	import { drag } from 'd3-drag';
	import { forceSimulation, forceLink, forceManyBody, forceCenter } from 'd3-force';

	import {event as currentEvent} from 'd3-selection'  // Needed to get drag working, see: https://github.com/d3/d3/issues/2733
	let d3 = { zoom, zoomIdentity, scaleLinear, scaleOrdinal, schemeCategory10, select, selectAll, drag,  forceSimulation, forceLink, forceManyBody, forceCenter }

	export let graph;

	let width = 500;
	let height = 600;
    const nodeRadius = 5;

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

	let transform = d3.zoomIdentity;
    let simulation, svg;
	onMount(() => {
		svg = d3.select(".chart")
			.append("svg")
			.attr("width", width)
			.attr("height", height)
			// .attr("viewBox", [0, 0, width, height]);

		simulation = d3.forceSimulation(nodes)
			.force("link", d3.forceLink(links).id(d => d.id))
			.force("charge", d3.forceManyBody())
			.force("center", d3.forceCenter(width / 2, height / 2))
			.on('tick', simulationUpdate);

		const link = svg.append("g")
			.attr("stroke", "#999")
			.attr("stroke-opacity", 0.6)
			.selectAll("line")
			.data(links)
			.join("line")
			.attr("stroke-width", d => Math.sqrt(d.value));

		const node = svg.append("g")
			.attr("stroke", "#fff")
			.attr("stroke-width", 1.5)
			.selectAll("circle")
			.data(nodes)
			.join("circle")
			.attr("r", 5)
			.attr("fill", d => colourScale(d.group))
			.call(d3.drag()
				.on("start", dragstarted)
				.on("drag", dragged)
				.on("end", dragended))

		node.append("title")
			.text(d => d.id);

		d3.select(".chart").call(d3.zoom()
			.extent([[0, 0], [width, height]])
			.scaleExtent([1 / 10, 8])
			.on("zoom", zoomed));

		function zoomed() {
			transform = currentEvent.transform;
			simulationUpdate();
		}

		function simulationUpdate () {
			link
				.attr("x1", d => d.source.x)
				.attr("y1", d => d.source.y)
				.attr("x2", d => d.target.x)
				.attr("y2", d => d.target.y)
				.attr("transform", transform);

			node
				.attr("cx", d => d.x)
				.attr("cy", d => d.y)
				.attr("transform", transform);
			}

	});

	function dragsubject() {
        const node = simulation.find(transform.invertX(currentEvent.x), transform.invertY(currentEvent.y), nodeRadius);
        if (node) {
            node.x = transform.applyX(node.x);
            node.y = transform.applyY(node.y);
        }
        return node;
	}

    function dragstarted() {
        if (!currentEvent.active) simulation.alphaTarget(0.3).restart();
        currentEvent.subject.fx = transform.invertX(currentEvent.subject.x);
        currentEvent.subject.fy = transform.invertY(currentEvent.subject.y);
    }

    function dragged() {
        currentEvent.subject.fx = transform.invertX(currentEvent.x);
        currentEvent.subject.fy = transform.invertY(currentEvent.y);
    }

    function dragended() {
        if (!currentEvent.active) simulation.alphaTarget(0);
        currentEvent.subject.fx = null;
        currentEvent.subject.fy = null;
    }

	function resize() {
		({ width, height } = svg.getBoundingClientRect());
	}


</script>

<svelte:window on:resize='{resize}'/>

<div class='chartdiv'></div>

<style>
	svg {
		float: left;
	}

	circle {
		stroke: #fff;
    stroke-width: 1.5;
	}

</style>