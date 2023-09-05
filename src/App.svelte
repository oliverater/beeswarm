<script> //test
  import data from "$data/data.js";
  //console.log(data);
  import {forceSimulation, forceX, forceY, forceCollide} from "d3-force";
  // import simulation from "d3-force/src/simulation";

  import AxisX from "$components/AxisX.svelte";
  import AxisY from "$components/AxisY.svelte";
  import Legend from "$components/Legend.svelte";

  const RADIUS = 8
  const simulation = forceSimulation(data)
  $: {
  simulation.force("x", forceX().x(d => xScale(d.year)).strength(0.5))
  .force("y", forceY().y(d => yScale(d.region)).strength(0.4))
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
      height = 600;

  const margin = {
    top: 50,
    right: 50,
    bottom: 50,
    left: 150 
  };

  $: innerWidth = width - margin.left - margin.right;
  let innerHeight = height - margin.top - margin.bottom;

  // $: console.log({innerWidth})
  import {scaleLinear, scaleBand, scaleOrdinal} from "d3-scale"; 

  $: xScale = scaleLinear()
  .domain([2015, 2023]) //INPUT
  .range([0, innerWidth]) //OUTPUT

  // import { mean, rollups } from "d3-array";

  // const region = rollups(
  //   data,
  //   (v) => mean(v, (d) => d.year),
  //   (d) => d.region
  //   )
  //   .sort((a,b) => a[1] - b[1])
  //   .map((d) => d[0]);

const region = data.map(d => d.region);
const impact = data.map(d => d.impact);

  // console.log({events})

  const colorRange = [
    "#999999", //drought
    "#0b4572", //extreme rainfall
    "#c7432b", // heatwave
    "#2f8fce", // Storm, extreme rainfall
    "#efc530", // Wildfire
    "#c6e7fa" // Cold spell
  ]

  const colorScale = scaleOrdinal()
  .domain(region)
  .range(colorRange)


  const legendScale = scaleOrdinal()
  .domain(impact)
  .range(colorRange)


  let yScale = scaleBand()
  .domain(region)
  .range([0, innerHeight])

</script>

<h1>Extreme weather attribution study tracker</h1>
<Legend {legendScale}></Legend>
<div class="chart-container" bind:clientWidth={width}>
<svg {width} {height}>
  <g class="inner-chart" transform="translate({margin.left}, {margin.top})">
    <AxisX xScale={xScale} height={innerHeight} width={innerWidth}></AxisX>
    <AxisY {yScale}></AxisY>
    {#each nodes as node}
    <circle
      cx={node.x}
      cy={node.y}
      r={RADIUS}
      fill={colorScale(node.impact)}
      stroke="black"
      on:mouseover={() => {
        console.log(node)
      }}
      />
    {/each}
  </g>
</svg>
</div>

<style>
  /* :global(.tick text, .axis-title)   */
  h1{
    font-size:1.6em;
    font-weight: bold;
    color: #333333;
    margin-left:0.8em;
  }
</style>
