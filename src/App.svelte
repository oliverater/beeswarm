<script> //test
  import data from "$data/data.js";
  console.log(data);
  import {forceSimulation, forceX, forceY, forceCollide} from "d3-force";
  // import simulation from "d3-force/src/simulation";

  const RADIUS = 5
  $: simulation = forceSimulation(data)
  .force("x", forceX().x(d => xScale(d.year)).strength(0.8))
  .force("y", forceY().y(d => yScale(d.event)).strength(0.2))
  .force("collide", forceCollide().radius(RADIUS))

  $: console.log( simulation.nodes( ) )

  let width = 40,
      height = 400;

  const margin = {
    top: 0,
    right: 0,
    bottom: 20,
    left: 0 
  };

  $: innerWidth = width - margin.left - margin.right;
  let innerHeight = height - margin.top - margin.bottom;

  // $: console.log({innerWidth})
  import {scaleLinear, scaleBand} from "d3-scale"; 

  $: xScale = scaleLinear()
  .domain([2015, 2023]) //INPUT
  .range([0, innerWidth]) //OUTPUT

  let yScale = scaleBand()
  .domain(data.map(d => d.event) )
  .range([innerHeight, 0])

  $: nodes = simulation.nodes();
</script>

<div class="chart-container" bind:clientWidth={width}>
<svg {width} {height}>
  <g class="inner-chart" transform="translate({margin.left}, {margin.top})">
    {#each nodes as node}
    <circle
      cx={node.x}
      cy={node.y}
      r={RADIUS}
      fill='#ff00ff'
      />
    {/each}
  </g>
</svg>
</div>

<style>
  
</style>
