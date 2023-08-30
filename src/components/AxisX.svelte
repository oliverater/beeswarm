<script>
    export let xScale;
    export let height;
    export let width;

    // $: console.log({ xScale, height, width} )
    // $: console.log( xScale.ticks())

    $: ticks = xScale.ticks()
</script>

<g class="axis x">
    {#each ticks as tick}
        <g class="tick" transform="translate({xScale(tick)}, 0)">
            <text x="-20" y={height+20}>{tick}</text>
            <line x1="0" x2="0" y1="0" y2={height} stroke="#d1d1d1"></line>
    </g>
    {/each}
    <text class="axis-title" x={width/2} y={height+30} text-anchor="middle" dominant-baseline="hanging" dy="5">Years</text>
</g>