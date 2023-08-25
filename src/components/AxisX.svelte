<script>
    export let xScale;
    export let height;
    export let width;

    $: console.log({ xScale, height, width} )
    $: console.log( xScale.ticks())

    $: ticks = xScale.ticks()
</script>

<g class="axis x">
    {#each ticks as tick}
        <g class="tick" transform="translate({xScale(tick)}, 0)">
            <text x="-20" y={height}>{tick}</text>
    </g>
    {/each}
</g>