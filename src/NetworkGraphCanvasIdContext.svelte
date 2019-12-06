<h2>d3 Force Directed Graph in Svelte js - 2 x canvas</h2>
<p>Uses HTML5 <code>&lt;canvas&gt;</code>, one for the 
display an one for hit detection using node colours based 
on node index.</p>
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

    let svg;
    let canvas, idCanvas;
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

    const groupColour = d3.scaleOrdinal(d3.schemeCategory10);

    let transform = d3.zoomIdentity;
    let simulation, context, idContext
    onMount(() => {
        context = canvas.getContext('2d');
        idContext = idCanvas.getContext('2d');
        resize()
        
        simulation = d3.forceSimulation(nodes)
            .force("link", d3.forceLink(links).id(d => d.id))
            .force("charge", d3.forceManyBody())
            .force("center", d3.forceCenter(width / 2, height / 2))
           .on("tick", simulationUpdate);

        // title
        d3.select(context.canvas).on("mousemove", tooltip);
        function tooltip () {
            const d = getNodeFromMouseEvent();
            if (d)
                context.canvas.title = d.id;
            else
                context.canvas.title = '';
        };

        d3.select(canvas)
        .call(d3.drag()
            .container(canvas)
            .subject(dragsubject)
            .on("start", dragstarted)
            .on("drag", dragged)
            .on("end", dragended))
        .call(d3.zoom()
          .scaleExtent([1 / 10, 8])
          .on('zoom', zoomed));    
    });

    function simulationUpdate() {
        nodes = [...nodes]
        links = [...links]
        context.save();
        context.clearRect(0, 0, context.canvas.width, context.canvas.height);
        context.translate(transform.x, transform.y);
        context.scale(transform.k, transform.k);

        idContext.save();
        idContext.clearRect(0, 0, idContext.canvas.width, idContext.canvas.height);
        idContext.translate(transform.x, transform.y);
        idContext.scale(transform.k, transform.k);
        
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
            context.arc(d.x, d.y, 5, 0, 2*Math.PI);
            context.strokeStyle = "#fff";
            context.lineWidth = 1.5;
            context.stroke();
            context.fillStyle = groupColour(d.group);
            context.fill();
            
            idContext.beginPath();
            idContext.arc(d.x, d.y, 5, 0, 2*Math.PI);
            idContext.fillStyle = indexToColor(i);
            idContext.fill();
        });
        context.restore();
        idContext.restore();
    }
 
    function zoomed() {
        transform = currentEvent.transform;
        simulationUpdate();
    }

function dragsubject() {
        const node = getNodeFromMouseEvent();
        if (node) {
            node.x = transform.applyX(node.x);
            node.y = transform.applyY(node.y);
        }
        return node;
    }

    // This method of hit detection is poor on small devices because fat fingers
    // can't hit small targets. Alternatives:
    //  - add a hit radius to this (larger for small touch screens)
    //  - use simulation.find() with a hit radius (larger for small touch screens)
    function getNodeFromMouseEvent() {
        let mouse = d3.mouse(context.canvas);
        const color = idContext.getImageData(mouse[0], mouse[1], 1, 1).data;
        const index = colorToIndex(color);
        const node = nodes[index];
        return node;
    };

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

    function indexToColor(index) {
        const color = "#" + (index + 1).toString(16).padStart(6, "0");
        return color;
    }

    function colorToIndex(color) {
        const index = ((color[0] << 16) + (color[1] << 8) + color[2]) - 1;
        return index;
    }

    function resize() {
        ({ width, height } = canvas);
    }

</script>

<svelte:window on:resize='{resize}'/>

<div class='container'>
    <canvas bind:this={canvas} width='{width}' height='{height}'/>
    <canvas bind:this={idCanvas} width='{width}' height='{height}' hidden='true'/>
</div>

<style>
	canvas {
		float: left;
	}
</style>