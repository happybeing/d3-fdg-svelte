<h2>d3 Force Directed Graph in Svelte js - canvas with d3 hit detection</h2>
<p>Uses HTML5 <code>&lt;canvas&gt;</code> for the display and d3 'find()' 
for hit detection.
<p>Features: hover, drag nodes, zoom and pan with mouse
or touch screen. Note: on small touch screens you may need to 
increase hit radius to hit a node with a 'fat finger'!</p>

<script>
    import { onMount } from 'svelte';

    import { scaleLinear, scaleOrdinal } from 'd3-scale';
    import { zoom, zoomIdentity } from 'd3-zoom';
    import { schemeCategory10 } from 'd3-scale-chromatic';
    import { select, selectAll, mouse } from 'd3-selection';
    import { drag } from 'd3-drag';
    import { forceSimulation, forceLink, forceManyBody, forceCenter } from 'd3-force';

    import {event as currentEvent} from 'd3-selection'  // Needed to get drag working, see: https://github.com/d3/d3/issues/2733
    let d3 = { zoom, zoomIdentity, scaleLinear, scaleOrdinal, schemeCategory10, select, selectAll, mouse, drag,  forceSimulation, forceLink, forceManyBody, forceCenter }

    export let graph;

    let canvas;
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

    $: xTicks = width > 180 ?
        [0, 4, 8, 12, 16, 20] :
        [0, 10, 20];

    $: yTicks = height > 180 ?
        [0, 2, 4, 6, 8, 10, 12] :
        [0, 4, 8, 12];

    $: d3yScale = scaleLinear()
        .domain([0, height])
        .range([height, 0]);

    $: links = graph.links.map(d => Object.create(d));
    $: nodes = graph.nodes.map(d => Object.create(d));  

    const groupColour = d3.scaleOrdinal(d3.schemeCategory10);

    let transform = d3.zoomIdentity;
    let simulation, context
    onMount(() => {
        context = canvas.getContext('2d');
        resize()
        
        simulation = d3.forceSimulation(nodes)
            .force("link", d3.forceLink(links).id(d => d.id))
            .force("charge", d3.forceManyBody())
            .force("center", d3.forceCenter(width / 2, height / 2))
            .on("tick", simulationUpdate);

        // title
        d3.select(context.canvas)
            .on("mousemove", () => {
            const mouse = d3.mouse(context.canvas);
            const d = simulation.find(transform.invertX(mouse[0]), transform.invertY(mouse[1]), nodeRadius);
            
            if (d) 
                context.canvas.title = d.id;
            else
                context.canvas.title = '';
        });

        d3.select(canvas)
        .call(d3.drag()
            .container(canvas)
            .subject(dragsubject)
            .on("start", dragstarted)
            .on("drag", dragged)
            .on("end", dragended))
        .call(d3.zoom()
          .scaleExtent([1 / 10, 8])
          .on('zoom', zoomed));    });

    function simulationUpdate () {
        context.save();
        context.clearRect(0, 0, context.canvas.width, context.canvas.height);
        context.translate(transform.x, transform.y);
        context.scale(transform.k, transform.k);

        links.forEach(d => {
            context.beginPath();
            context.moveTo(d.source.x, d.source.y);
            context.lineTo(d.target.x, d.target.y);
            context.globalAlpha = 0.6;
            context.strokeStyle = "#999";
            context.lineWidth = Math.sqrt(d.value);
            context.stroke();
            context.globalAlpha = 1;
        });
        
        nodes.forEach((d, i) => {
            context.beginPath();
            context.arc(d.x, d.y, nodeRadius, 0, 2*Math.PI);
            context.strokeStyle = "#fff";
            context.lineWidth = 1.5;
            context.stroke();
            context.fillStyle = groupColour(d.group);
            context.fill();
        });
        context.restore();
    }

    function zoomed() {
        transform = currentEvent.transform;
        simulationUpdate();
    }

    // Use the d3-force simulation to locate the node
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
        ({ width, height } = canvas);
    }

</script>

<svelte:window on:resize='{resize}'/>

<div class='container'>
    <canvas bind:this={canvas} width='{width}' height='{height}'/>
</div>

    <style>
    canvas {
        float: left;
    }
</style>