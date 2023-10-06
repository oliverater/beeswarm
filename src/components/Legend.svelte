<script>
    export let legendScale;
    export let hoveredLegend;

    $: console.log({hoveredLegend})

//     const colorScale = scaleOrdinal()
//   .domain(region)
//   .range(colorRange)

</script>
    <div class="legend" on:mouseout={() =>{
        hoveredLegend = null;
    }}>
        {#each legendScale.domain() as legendLabel}
            <p
            on:mouseover={() => {
                hoveredLegend = legendLabel;
            }}
            class:unhovered={hoveredLegend && hoveredLegend !== legendLabel}>
                <span style="background-color: {legendScale(legendLabel)}"></span>
                {legendLabel}
            </p>
        {/each}
    </div>
<style>
    .legend{
        display: flex;
        flex-direction: row;
        justify-content: left;
        margin-left: 1.3em;
        margin-top:0.5em;
        flex-wrap: wrap;
        column-gap: 10px;
        row-gap:5px;
    }
    span {
        width: 12px;
        height: 12px;
        display: inline-block;
        border-radius: 50%;
        border: solid 1px #333333;
    }
    p{
        display: flex;
        align-items: center;
        column-gap: 5px;
        transition: opacity  300ms ease;
        cursor: pointer;
    }
    .unhovered{
        opacity: 0.3;
    }
</style>