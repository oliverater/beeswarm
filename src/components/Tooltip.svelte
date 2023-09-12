<script>
    export let data;
    export let legendScale;

    import {fly, fade} from "svelte/transition";
</script>

<div class="tooltip"
    in:fly={{ y:5, duration: 200, delay: 50}}
    out:fade
    style="
    top: {data.y}px;
    left: {data.x +125}px;">
    <h1>
        {data.country}
    </h1>
        {#if data.impact != "Cold spell" || data.impact != "Wildfire" }
        <span class="impact" style="background-color:{legendScale(data.impact)}">
            {data.impact}
        </span>
        {:else}
        <span class="impact" style="background-color:{legendScale(data.impact)} color:black">
            {data.impact}
        </span>
        {/if}
        <span class="outcome">
            {data.outcome}
        </span>
        <span class="link">
            <a href="{data.study}" target="_blank">{data.date}</a>
        </span>
</div> 

<style>
    .tooltip{
        position: absolute;
        background: white;
        box-shadow: 2px 3px 8px rgba(0,0,0,0.15);
        padding: 8px 6px;
        border-radius: 3px;
        display: inline-grid;
    }
    .impact{
        border-radius:3px;
        padding:1.2px;
        width: fit-content;
        color: #f4f4f4;
    }
</style>