# D3 Force Directed Graph in Svelte

This project contains Svelte JS versions of the [D3 Force Directed Graph example](https://observablehq.com/@d3/force-directed-graph).

## Svelte Implementations
Each Svelte version is implemented as a file/component which can be tested
by modifying `src/App.svelte` to select the one you wish to try. Or you can
view the code and a live example using the 'REPL' links below.

1. **NetworkGraphSvelteSVG.svelte** - uses Svelte `SVG` elements ([REPL](https://svelte.dev/repl/b74876c524734666ba44cb3fa5b48e90) or [REPL](https://svelte.dev/repl/01a5774b53e9416584428c025668407b?version=3.15.0) without d3-zoom`*`)
2. **NetworkGraphD3SVG.svelte** - uses D3 `SVG` elements ([REPL](https://svelte.dev/repl/a405bd0a2ace463683b6100c94d3e968?version=3.15.0))
3. **NetworkGraphCanvas.svelte** - uses `canvas` with D3 hit detection ([REPL](https://svelte.dev/repl/8b5800c9552440fbb51848504a3c46c9) or [REPL](https://svelte.dev/repl/498b9556c3254c56a2f6c7cfc206bfb1?version=3.16.0) without d3-zoom)
4. **NetworkGraphCanvasIdContext.svelte** - uses `canvas` with a second context for hit detection ([REPL](https://svelte.dev/repl/5cc46975e2c94f4d877d81b6dfbaa142) or [REPL](https://svelte.dev/repl/2b1b461355204525989af7b9b191ef49?version=3.16.0) without d3-zoom)

    `*` In (1) without zoom, I've added some axes but that's the only visual difference.

### Performance
It will be interesting to see how the alternative hit detection methods of
(3) and (4) compare. The former uses D3 `simulation.find()` while (4) 
uses a second `idContext` (drawing all the nodes a second time and using 
the colour of any hit to determine which node was hit).

Work to compare performance of the above can be found in [d3-fdg-svelte-perf](https://github.com/theWebalyst/d3-fdg-svelte-perf)
### Touch Screens
All should work with laptop, mobile and tablet touch screens (tested with Chrome and Firefox on Ubuntu and Android) but
note:
> It can be hard to hit the nodes with a fat finger on a small screen, so if you
> want to test or support mobile devices, you may have to enlarge the hit radius 
> when the display is a small screen.

## Get started
The conversions are based on the default [sveltejs/template](https://github.com/sveltejs/template), so refer to that for more information. 
Note though, I have modified it to use `yarn` rather than `npm`, so the 
essential commands are given below.

> *Note that you will need to have [Node.js](https://nodejs.org) and `yarn` installed.*

Get the code...
```bash
git clone https://github.com/theWebalyst/d3-fdg-svelte
```

Install the dependencies...

```bash
cd d3-fdg-svelte
yarn
```

...then start [Rollup](https://rollupjs.org):

```bash
yarn dev
```

Navigate to [localhost:5000](http://localhost:5000). You should see the app running.

## More information 
For more information about the template used to create this app, see the README at https://github.com/sveltejs/template.
