<h2>d3 Force Directed Graph in Sveltejs - D3 created svg (zoom)</h2>
<p>Uses D3 created SVG and DOM elements, and D3 mouse handling.</p>

<p>Features: hover, drag nodes, zoom and pan with mouse
or touch screen. Note: on small touch screens you may need to 
increase hit radius to hit a node with a 'fat finger'!</p>
<script>
    import { onMount, onDestroy } from 'svelte';

    import { scaleLinear, scaleOrdinal } from 'd3-scale';
    import { zoom, zoomIdentity } from 'd3-zoom';
    import { schemeCategory10 } from 'd3-scale-chromatic';
    import { select, selectAll } from 'd3-selection';
    import { drag } from 'd3-drag';
    import { forceSimulation, forceLink, forceManyBody, forceCenter } from 'd3-force';

    let d3 = { zoom, zoomIdentity, scaleLinear, scaleOrdinal, schemeCategory10, select, selectAll, drag,  forceSimulation, forceLink, forceManyBody, forceCenter }

    export let graph;

    let width = 500;
    let height = 600;
    const nodeRadius = 5;

    const padding = { top: 20, right: 40, bottom: 40, left: 25 };

    $: links = graph.links.map(d => Object.create(d));
    $: nodes = graph.nodes.map(d => Object.create(d));  

    const colourScale = d3.scaleOrdinal(d3.schemeCategory10);

    let simulation, svg;
    onMount(() => {
        simulation = d3.forceSimulation(nodes)
            .force("link", d3.forceLink(links).id(d => d.id))
            .force("charge", d3.forceManyBody())
            .force("center", d3.forceCenter(width / 2, height / 2))
            .on('tick', simulationUpdate);

        svg = d3.select(".chart")
            .append("svg")
            .attr("width", width)
            .attr("height", height)
  
        const g = svg.append("g");    // Need container when combininig drag and zoom
                                    // See example: https://observablehq.com/@d3/drag-zoom

        const link = g.append("g")
            .attr("stroke", "#999")
            .attr("stroke-opacity", 0.6)
            .selectAll("line")
            .data(links)
            .join("line")
            .attr("stroke-width", d => Math.sqrt(d.value));

        const node = g.append("g")
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

        svg.call(d3.zoom()
                .extent([[0, 0], [width, height]])
                .scaleExtent([1 / 10, 8])
                .on("zoom", zoomed));
            
        function simulationUpdate () {
            link
                .attr("x1", d => d.source.x)
                .attr("y1", d => d.source.y)
                .attr("x2", d => d.target.x)
                .attr("y2", d => d.target.y)

            node
                .attr("cx", d => d.x)
                .attr("cy", d => d.y)
            }
    
        function zoomed(currentEvent) {
            g.attr("transform", currentEvent.transform)
            simulationUpdate();
        }
    });

    onDestroy(() => {
        //d3.select(".chart").remove(svg);
        svg.remove();
    });

    function dragstarted(currentEvent) {
        if (!currentEvent.active) simulation.alphaTarget(0.3).restart();
        currentEvent.subject.fx = currentEvent.x;
        currentEvent.subject.fy = currentEvent.y;
    }

    function dragged(currentEvent) {
        currentEvent.subject.fx = currentEvent.x;
        currentEvent.subject.fy = currentEvent.y;
    }

    function dragended(currentEvent) {
        if (!currentEvent.active) simulation.alphaTarget(0);
        currentEvent.subject.fx = null;
        currentEvent.subject.fy = null;
    }

    function resize() {
        ({ width, height } = svg.node().getBoundingClientRect());
    }


</script>

<svelte:window on:resize='{resize}'/>

<div class='chartdiv'></div>