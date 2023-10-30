<script>
    export let yScale;
    $: ticks = yScale.domain();
    // export let width;

    // $: ticks = yScale.ticks()
</script>

<g class="axis y">
    {#each ticks as tick}
        <g class="tick">
            <text y={yScale(tick)} x="-125">
                {tick}
            </text>
            <!-- <line y1="0" y2="0" x1="0" x2={width} stroke="#d1d1d1"></line> -->
            <!-- <rect></rect> -->
        </g>
    {/each}
</g>