<script> //test
  import data from "$data/data.js";
  //console.log(data);
  import {forceSimulation, forceX, forceY, forceCollide} from "d3-force";
  import { fade } from "svelte/transition";
  // import simulation from "d3-force/src/simulation";

  import AxisX from "$components/AxisX.svelte";
  import AxisY from "$components/AxisY.svelte";
  import Legend from "$components/Legend.svelte";
  import Tooltip from "$components/Tooltip.svelte";


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

  let tooltip;
  let visible;
</script>

<h1>Extreme weather attribution study tracker</h1>
<Legend {legendScale}></Legend>
<div class="chart-container" bind:clientWidth={width}>
<svg {width} {height}>
  <g class="inner-chart" transform="translate({margin.left}, {margin.top})">
    <AxisX xScale={xScale} height={innerHeight} width={innerWidth}></AxisX>
    <AxisY {yScale}></AxisY>
    {#each nodes as node}
    <!-- svelte-ignore a11y-no-noninteractive-tabindex -->
    <circle
      cx={node.x}
      cy={node.y}
      r={RADIUS}
      fill={legendScale(node.impact)}
      stroke="black"
      on:click={() => {
        tooltip = node;
      }}
      on:focus={() => {
        tooltip = node;
      }}
      tabindex=0
      />
    {/each}
  </g>
</svg>
  {#if tooltip}
    <!-- <Tooltip data={tooltip} {legendScale}></Tooltip> -->
    <div class="tooltip">
      <div class="close-tooltip" on:click={() => {
        tooltip = null;
      }}>X</div>
      <h3>
          <span class="impact" style="background-color:{legendScale(data.impact)}">{data.impact}</span><a href="{data.study}" target="_blank" rel="noreferrer">{data.tooltipDate},{data.country}</a>
      </h3>
          <p><span class="key">Outcome:</span>{data.outcomeSummary}</p>
          <p><span class="key">Impacts:</span>{data.impactSummary}</p>
          <p><span class="key">Vulnerability:</span>{data.vulnerabilitySummary}</p>
          <p><span class="key">Resilience:</span>{data.resilienceSummary}</p>
  </div>
  {/if}
</div>

<style>
  /* :global(.tick text, .axis-title)   */
  h1{
    font-size:1.6em;
    font-weight: bold;
    color: #333333;
    margin-left:0.8em;
  }
  circle{
    cursor:pointer;
  }
  circle:hover{
    filter: brightness(115%);
  }
  circle:focus{
    outline:none;
  }
  .tooltip{
        position: absolute;
        background: rgba(255, 255, 255, 0.95);
        box-shadow: 2px 3px 8px rgba(0,0,0,0.15);
        padding: 2.5em;
        border-radius: 3px;
        display: inline-grid;
        width: auto;
        top:5%;
        left:20%;
        right:10%;
        bottom:10%;
    }
    .close-tooltip{
        position: absolute;
        right: 0;
        margin-right: 1.2em;
        margin-top: 1.2em;
        font-size: 1.2em;
        font-weight: 700;
    }
    .close-tooltip:hover{
        filter: brightness(15%);
        cursor: pointer;
    }
    .impact{
        border-radius:3px;
        padding:1.2px;
        width: fit-content;
        color: #f4f4f4;
    }
    span.key{
        font-weight:700;
    }
</style>
