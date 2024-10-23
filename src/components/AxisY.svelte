<script>
    export let yScale;
    $: ticks = yScale.domain();
    // export let width;

    // $: ticks = yScale.ticks()
</script>

<g class="axis y">
    {#each ticks as tick}
    <!-- <rect width="100" height="20" style="fill:white;" /> -->
        <g class="tick">
            <text y={yScale(tick)} x="-10">
                {tick}
            </text>
            <!-- <line y1="0" y2="0" x1="0" x2={width} stroke="#d1d1d1"></line> -->
            <!-- <rect></rect> -->
        </g>
    {/each}
</g>

<style>
    .tick{
        text-anchor:end
    }
</style>