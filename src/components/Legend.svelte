<script>
    export let legendScale;
    console.log({legendScale})

//     const colorScale = scaleOrdinal()
//   .domain(region)
//   .range(colorRange)

</script>
    <div class="legend">
        {#each legendScale.domain() as impact}
            <p>
                <span style="background-color: {legendScale(impact)}"></span>
                {impact}
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
    }
</style>