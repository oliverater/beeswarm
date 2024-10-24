<script> //test
  import data from "$data/data.js";
  import upIcon from "$image/upChevron.png";
  import noIcon from "$image/noChevron.png";
  import downIcon from "$image/downChevron.png";
  import {forceSimulation, forceX, forceY, forceCollide} from "d3-force";
  import { fade } from "svelte/transition";
  // import simulation from "d3-force/src/simulation";

  import AxisX from "$components/AxisX.svelte";
  import AxisY from "$components/AxisY.svelte";
  import Legend from "$components/Legend.svelte";
  import Tooltip from "$components/Tooltip.svelte";


  let Desktop = 10;
  let Mobile = 6

  import { mean, rollups } from "d3-array";

  const region = data.map(d => d.region);
  const impact = data.map(d => d.event);

  let width = 400,
      height = 500;

  const margin = {
    top: 50,
    right: 50,
    bottom: 50,
    left: 165 
  };

  // let innerWidth = 0;

  $: innerWidth = width - margin.left - margin.right;
  let innerHeight = height - margin.top - margin.bottom;

  // $: console.log({innerWidth})
  import {scaleLinear, scaleBand, scaleOrdinal} from "d3-scale"; 

  $: xScale = scaleLinear()
  .domain([2015.0, 2024.99]) //INPUT
  .range([0, innerWidth])


const colorRange = [
"#dc006e", // Heat 
"#005582", // Rainfall
"#ffb94b", // Drought
"#00966e", // Storm
"#f04600", // Wildfire
"#00d7e6", // Cold spell
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

  simulation.force("x", forceX().x(d => xScale(d.year)).strength(0.3))
  .force("y", forceY()
  .y((d) => (groupbyContinent ? yScale(d.region) : yScale(d.event)))
  .strength(0.35))
  .force("collide", forceCollide().radius(innerWidth < 550 ? Mobile : Desktop))
  .alpha(0.15)
  .alphaDecay(0.00001)
  .restart()
  }

  // $: console.log( simulation.nodes( ) )

  let nodes = []; // An empty array which will be populated once the simulation ticks
  $: simulation.on("tick", () => {
    nodes = simulation.nodes(); // Update the nodes array
});

  let tooltip;

  let impactRate = {
    "Impacts worsened by climate change" : upIcon,
    "More severe or likely" : upIcon,
    "No evidence of change" : noIcon,
    "Inconclusive" : noIcon,
    "Less severe or likely" : downIcon, 
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

<!-- <svelte:window bind:innerWidth /> -->
<h1>Extreme weather attribution study tracker</h1>
<Legend {legendScale} bind:hoveredLegend></Legend>
<div id="toggle-container">
    <p class="{checked ? '': 'bold'}">Region</p>
      <label class="switch">
      <input type="checkbox" bind:checked on:click={toggleLabels}/>
      <span class="slider" />
    </label>
    <p class="{checked ? 'bold': ''}">Event type</p>
</div>
<div class="chart-container" bind:clientWidth={width}>
<!-- svelte-ignore a11y-click-events-have-key-events -->
<svg {width} {height} on:click={() => {
  tooltip = null;
}}>
  <g class="inner-chart" transform="translate({margin.left}, {margin.top})">
    <AxisX xScale={xScale} height={innerHeight} width={innerWidth}></AxisX>

    <AxisY {yScale} {width}></AxisY>
    $: {#each nodes as node}
    <!-- svelte-ignore a11y-no-noninteractive-tabindex -->
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    {#if innerWidth < 550}
    <circle
      cx={node.x}
      cy={node.y}
      r={Mobile}
      fill={groupbyContinent ? legendScale(node.event) : legendScale(node.region)}
      stroke={tooltip || hoveredLegend ? tooltip === node || hoveredLegend === node.event || hoveredLegend === node.region ? "black" : "transparent" : "black"}
      opacity={tooltip || hoveredLegend ? tooltip === node || hoveredLegend === node.event || hoveredLegend === node.region ? 1 : 0.3 : 1 }
      on:click={() => {
        tooltip = node;
      }}
      on:focus={() => {
        tooltip = node;
      }}
      tabindex=0
      />
      {:else}
      <circle
      cx={node.x}
      cy={node.y}
      r={Desktop}
      fill={groupbyContinent ? legendScale(node.event) : legendScale(node.region)}
      stroke={tooltip || hoveredLegend ? tooltip === node || hoveredLegend === node.event || hoveredLegend === node.region ? "black" : "transparent" : "black"}
      opacity={tooltip || hoveredLegend ? tooltip === node || hoveredLegend === node.event || hoveredLegend === node.region ? 1 : 0.3 : 1 }
      on:click={() => {
        tooltip = node;
      }}
      on:focus={() => {
        tooltip = node;
      }}
      tabindex=0
      />
      {/if}
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
          <span class="impact" style="background-color:{groupbyContinent ? legendScale(tooltip.event) : legendScale(tooltip.region)}"></span> {groupbyContinent ? [(tooltip.event)+ " | " +(tooltip.region)] : [(tooltip.region)+ " | " +(tooltip.event)]}
      </h3>
      <p class="summary"><span class="key">{tooltip.tooltipDate}: {tooltip.country}</span></p>
          <p class="summary">
            <span class="key" style="display:block;margin-block-end:0.5em">Role of climate change:</span>
            <img class="impact-outcome" src={impactRate[tooltip.outcome]}/> {tooltip.outcome}
          </p>
          <p class="summary"><span class="key">Outcome: </span>{tooltip.outcomeSummary}</p>
          <p class="summary"><span class="key">Impacts: </span>{tooltip.impactsSummary}</p>
          <hr>
          <p class="summary"><span class="key"><a href="{tooltip.study}" target="_blank" rel="noreferrer">Link to study</a></span></p>

          {#if tooltip.peerReviewed != ''}
          <p class="summary"><span class="key">Peer review: <a href="{tooltip.peerReviewed}" target="_blank" rel="noreferrer">{tooltip.peerReviewed}</a></span></p>
          {/if}
          <!-- <p class="summary"><span class="key">Vulnerability: </span>{tooltip.vulnerabilitySummary}</p>
          <p class="summary"><span class="key">Resilience: </span>{tooltip.resilienceSummary}</p> -->
  </div>
  {/if}

</div>

<style>
  .bold{
    font-weight: 700;
  }
  h1,h3{
    font-size:2em;
    font-weight: bold;
    color: #333333;
    font-family: 'Spectral', serif;
  }
  h1{
    margin-left:0.5em;
  }
  p.summary{
    line-height: 1.2;
    font-size:1.15em;
  }
  circle{
    cursor:pointer;
  }
  circle:hover{
    filter: brightness(145%);
  }
  circle:focus{
    outline:none;
  }
  .tooltip{
        position: absolute;
        background: #f5f5f5;
        box-shadow: 2px 3px 8px rgba(0,0,0,0.15);
        padding: 2.5em;
        border-radius: 3px;
        display: flex;
        flex-direction: column;
        width: auto;
        top:5%;
        left:5%;
        right:5%;
        /* bottom:10%; */
    }
    .tooltip > h3, .tooltip > p{
      margin-block-end: 1em;
    }
    .tooltip > p a{
      color: #dc006e;
    }
    .tooltip > hr{
      border-bottom:1px solid #333333;
      width: 100%;
      margin-block-end: 2em;
    }
    span.section{
      font-size:0.8em;
      line-height: normal;
      display: block;
      font-weight: 700;
      margin-block-end: 0.25em;
    }
    .close-tooltip{
        position: absolute;
        right: 0;
        margin-right: 1.2em;
        margin-top:-1em;
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
    img.impact-outcome{
      max-width:25px;
      height: auto;
    }
    span.key{
        font-weight:700;
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
    @media (max-width:776px){
    :global(.x-axis){
        font-size: 12px;
    }
    .tooltip > h3, .tooltip > p{
      margin-block-end: 0.75em;
    }
    .tooltip > p a{
      font-size:14px;
      line-break: anywhere;
    }
  }
  @media (max-width:700px){
    :global(text .x-axis){
      /* display: none; */
      fill: #57b0eb;
    }
  }
</style>
