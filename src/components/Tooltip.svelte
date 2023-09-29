<script>
    export let data;
    export let legendScale;

    import {fly, fade} from "svelte/transition";
    let visible = true;
    function toggleVissible() {
        visible = !visible
    }
</script>

{#if visible}
<div class="tooltip">
    <div class="close-tooltip" on:click={toggleVissible}>X</div>
    <h3>
        <span class="impact" style="background-color:{legendScale(data.impact)}">{data.impact}</span><a href="{data.study}" target="_blank" rel="noreferrer">{data.tooltipDate},{data.country}</a>
    </h3>
        <p><span class="key">Outcome:</span>{data.outcomeSummary}</p>
        <p><span class="key">Impacts:</span>{data.impactSummary}</p>
        <p><span class="key">Vulnerability:</span>{data.vulnerabilitySummary}</p>
        <p><span class="key">Resilience:</span>{data.resilienceSummary}</p>
</div>
{/if}

<style>
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