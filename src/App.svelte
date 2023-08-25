<script> //test
  import data from "$data/data.js";
  //console.log(data);
  import {forceSimulation, forceX, forceY, forceCollide} from "d3-force";
  // import simulation from "d3-force/src/simulation";

  import AxisX from "$components/AxisX.svelte";

  const RADIUS = 8
  const simulation = forceSimulation(data)
  $: {
  simulation.force("x", forceX().x(d => xScale(d.year)).strength(0.5))
  .force("y", forceY().y(d => yScale(d.event)).strength(0.4))
  .force("collide", forceCollide().radius(RADIUS))
  .alpha(0.3)
  .alphaDecay(0.0005)
  .restart()
  }

  $: console.log( simulation.nodes( ) )

  let nodes = []; // An empty array which will be populated once the simulation ticks
  simulation.on("tick", () => {
    nodes = simulation.nodes(); // Update the nodes array
});

  let width = 400,
      height = 400;

  const margin = {
    top: 20,
    right: 30,
    bottom: 20,
    left: 30 
  };

  $: innerWidth = width - margin.left - margin.right;
  let innerHeight = height - margin.top - margin.bottom;

  // $: console.log({innerWidth})
  import {scaleLinear, scaleBand, scaleOrdinal} from "d3-scale"; 

  $: xScale = scaleLinear()
  .domain([2015, 2023]) //INPUT
  .range([0, innerWidth]) //OUTPUT

  import { mean, rollups } from "d3-array";

  const events = rollups(
    data,
    (v) => mean(v, (d) => d.year),
    (d) => d.event
    )
    .sort((a,b) => a[1] - b[1])
    .map((d) => d[0]);

  console.log({events})

  const colorRange = [
    "#0b4572",
    "#2f8fce",
    "#c6e7fa",
    "#c7432b",
    "#efc530",
    "#999999"
  ]

  const colorScale = scaleOrdinal()
  .domain(events)
  .range(colorRange)

  let yScale = scaleBand()
  .domain(events)
  .range([innerHeight, 0])

</script>

<div class="chart-container" bind:clientWidth={width}>
<svg {width} {height}>
  <g class="inner-chart" transform="translate({margin.left}, {margin.top})">
    <AxisX xScale={xScale} height={innerHeight} width={innerWidth}></AxisX>
    {#each nodes as node}
    <circle
      cx={node.x}
      cy={node.y}
      r={RADIUS}
      fill={colorScale(node.event)}
      stroke="black"
      />
    {/each}
  </g>
</svg>
</div>

<style>
  
</style>
