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
    
    let simulation, context, idContext
    onMount(() => {
        context = canvas.getContext('2d');
        idContext = idCanvas.getContext('2d');
        resize()
        
        simulation = d3.forceSimulation(nodes)
            .force("link", d3.forceLink(links).id(d => d.id))
            .force("charge", d3.forceManyBody())
            .force("center", d3.forceCenter(width / 2, height / 2))
            .on('tick', function ticked() {
                simulation.tick()   //???
                // context.clearRect(0, 0, width, height);
                nodes = [...nodes]
                links = [...links]
            });

        // title
        d3.select(context.canvas).on("mousemove", () => {

            const d = getNodeFromMouseEvent(currentEvent);
            if (d) {
            context.canvas.title = d.id;
            } else {
            context.canvas.title = "";
            }
        });
  
        d3.select(context.canvas)
            .call(
            dragFunction(simulation)
                .container(context.canvas)
                .subject(() => {
                const d = getNodeFromMouseEvent(currentEvent.sourceEvent);
                return d || { x: currentEvent.x, y: currentEvent.y };
                })
            );
  
        simulation.on("tick", () => {
            context.clearRect(0, 0, context.canvas.width, context.canvas.height);
            idContext.clearRect(0, 0, idContext.canvas.width, idContext.canvas.height);
            
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
        });

    });

    function getNodeFromMouseEvent(event) {
        let mouse = getMousePos(canvas, event);
        const color = idContext.getImageData(mouse.x, mouse.y, 1, 1).data;
        const index = colorToIndex(color);
        const node = nodes[index];
        return node;
    };
    
    function getMousePos(canvas, event) {
        var rect = canvas.getBoundingClientRect();
        return {
            x: event.clientX - rect.left,
            y: event.clientY - rect.top
        };
    }

    function resize() {
        // ({ width, height } = canvas);
        // console.log('resize()', width, height)
    }

    function indexToColor(index) {
        const color = "#" + (index + 250).toString(16).padStart(6, "0");
        return color;
    }

    function colorToIndex(color) {
        const index = ((color[0] << 16) + (color[1] << 8) + color[2]) - 250;
        return index;
    }

    function dragFunction(simulation) {
    
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
        
        return d3.drag()
            .on("start", dragstarted)
            .on("drag", dragged)
            .on("end", dragended);
    }

</script>

<svelte:window on:resize='{resize}'/>

<div class='container'>
    <canvas bind:this={canvas} width='600' height='500'/>
    <canvas bind:this={idCanvas} width='600' height='500' hidden='true'/>
</div>

<style>
	canvas {
		/* width: 80%; */
		/* height: 80%; */
		float: left;
	}
</style>