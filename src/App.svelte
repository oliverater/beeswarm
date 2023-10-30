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

  const region = data.map(d => d.region);
  const impact = data.map(d => d.impact);

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
  .domain([2015.0, 2023.99]) //INPUT
  .range([0, innerWidth]) //OUTPUT

const colorRange = [
    "#999999", //drought
    "#0b4572", //extreme rainfall
    "#c7432b", // heatwave
    "#2f8fce", // Storm, extreme rainfall
    "#efc530", // Wildfire
    "#c6e7fa" // Cold spell
  ]

let yAxisLabel = region; // Initial y-axis label
let legendLabel = impact; // Initial legend label

  $: colorScale = scaleOrdinal()
  .domain(legendLabel)
  .range(colorRange)


  $: legendScale = scaleOrdinal()
  .domain(legendLabel)
  .range(colorRange)


  $: yScale = scaleBand()
  .domain(yAxisLabel)
  .range([0, innerHeight]);

  const simulation = forceSimulation(data)
  $: {
  simulation.force("x", forceX().x(d => xScale(d.xAxis)).strength(0.75))
  .force("y", forceY()
  .y((d) => (groupbyContinent ? yScale(d.region) : yScale(d.impact)))
  .strength(0.3))
  .force("collide", forceCollide().radius(RADIUS))
  .alpha(0.5)
  .alphaDecay(0.0005)
  .restart()
  }

  $: console.log( simulation.nodes( ) )

  let nodes = []; // An empty array which will be populated once the simulation ticks
  $: simulation.on("tick", () => {
    nodes = simulation.nodes(); // Update the nodes array
});

  let tooltip;

  let impactRate = {
    "Impacts worsened by climate change" : "^",
    "More severe or likely" : "^",
    "No evidence of change" : "–",
    "Less severe or likely" : "v"
  }

  let hoveredLegend;
  let groupbyContinent = true;

//update chart

  const toggleLabels = () => {
    yAxisLabel = yAxisLabel === impact ? region : impact;
    legendLabel = legendLabel === region ? impact : region;
    groupbyContinent = !groupbyContinent;
  };

let checked = false;
</script>

<h1>Extreme weather attribution study tracker</h1>
<Legend {legendScale} bind:hoveredLegend></Legend>
<div id="toggle-container">
    <p class="{checked ? '': 'bold'}">Continent</p>
      <label class="switch">
      <input type="checkbox" bind:checked on:click={toggleLabels}/>
      <span class="slider" />
    </label>
    <p class="{checked ? 'bold': ''}">Impact</p>
</div>
<div class="chart-container" bind:clientWidth={width}>
<!-- svelte-ignore a11y-click-events-have-key-events -->
<svg {width} {height} on:click={() => {
  tooltip = null;
}}>
  <g class="inner-chart" transform="translate({margin.left}, {margin.top})">
    <AxisX xScale={xScale} height={innerHeight} width={innerWidth}></AxisX>
    <rect width="115" height={innerHeight+20} x={-125} y={-20} style="fill:white;" />
    <rect width="100" height={innerHeight+20} x={innerWidth} y={-20} style="fill:white;" />
    <AxisY {yScale} {width}></AxisY>
    $: {#each nodes as node}
    <!-- svelte-ignore a11y-no-noninteractive-tabindex -->
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <circle
      cx={node.x}
      cy={node.y}
      r={RADIUS}
      fill={groupbyContinent ? legendScale(node.impact) : legendScale(node.region)}
      stroke={tooltip || hoveredLegend ? tooltip === node || hoveredLegend === node.impact || hoveredLegend === node.region ? "black" : "transparent" : "black"}
      opacity={tooltip || hoveredLegend ? tooltip === node || hoveredLegend === node.impact || hoveredLegend === node.region ? 1 : 0.3 : 1 }
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
    <div class="tooltip">
      <!-- svelte-ignore a11y-click-events-have-key-events -->
      <div class="close-tooltip" on:click={() => {
        tooltip = null;
      }}>X</div>
      <h3>
          <span class="impact" style="background-color:{groupbyContinent ? legendScale(tooltip.impact) : legendScale(tooltip.region)}"></span> {groupbyContinent ? [(tooltip.impact)+ " | " +(tooltip.region)] : [(tooltip.region)+ " | " +(tooltip.impact)]}
      </h3>
      <p><a class="metadata" href="{tooltip.study}" target="_blank" rel="noreferrer">{tooltip.tooltipDate}: {tooltip.country}</a></p>
          <p class="main-outcome">{impactRate[tooltip.outcome]?impactRate[tooltip.outcome]:tooltip.outcome}{tooltip.outcome}</p>
          <p class="summary"><span class="key">Outcome: </span>{tooltip.outcomeSummary}</p>
          <p class="summary"><span class="key">Impacts: </span>{tooltip.impactSummary}</p>
          <p class="summary"><span class="key">Vulnerability: </span>{tooltip.vulnerabilitySummary}</p>
          <p class="summary"><span class="key">Resilience: </span>{tooltip.resilienceSummary}</p>
  </div>
  {/if}
</div>

<style>
  .bold{
    font-weight: 700;
  }
  h1,h3{
    font-size:1.6em;
    font-weight: bold;
    color: #333333;
  }
  h1{
    margin-left:0.8em;
  }
  p.summary{
    line-height: 1.2;
    font-size:1.15em;
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
        background: #fffaf0;
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
        width: 12px;
        height: 12px;
        display: inline-block;
        border-radius: 50%;
        border: solid 1px #333333;
    }
    span.key{
        font-weight:700;
    }

    a.metadata{
      text-decoration: underline;
      color:#333333;
      font-weight: 700;
    }

    /* toggle */
    #toggle-container{
      display:flex; align-items:center;
      flex-direction: row;
      justify-content: left;
      margin-left: 1.3em;
      margin-top: 0.5em;
      flex-wrap: wrap;
      column-gap: 10px;
      row-gap: 5px;
    }

    .switch {
      position: relative;
      display: inline-block;
      width: 35px;
      height: 20px;
    }
  
    .switch input {
      opacity: 0;
      width: 0;
      height: 0;
    }
  
    .slider {
      position: absolute;
      cursor: pointer;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      background-color: #2f8fce;
      -webkit-transition: 0.4s;
      transition: 0.4s;
      border-radius: 20px;
    }
  
    .slider:before {
      position: absolute;
      content: "";
      height: 14px;
      width: 14px;
      left: 3px;
      bottom: 3px;
      background-color: white;
      -webkit-transition: 0.2s;
      transition: 0.2s;
      border-radius: 50%;
    }
  
    input:checked + .slider {
      background-color: #57b0eb;
    }
  
    input:checked + .slider {
      box-shadow: 0 0 1px #57b0eb;
    }
  
    input:checked + .slider:before {
      -webkit-transform: translateX(16px);
      -ms-transform: translateX(16px);
      transform: translateX(16px);
    }

    /* Cant remove initial lines from X Axis */
    :global(.mid:first-child){
        display: none !important;
    }
</style>
