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
    <h3>
        <span class="impact" style="background-color:{legendScale(data.impact)}">{data.impact}</span><a href="{data.study}" target="_blank" rel="noreferrer">{data.tooltipDate},{data.country}</a>
    </h3>
        <p><span class="key">Outcome:</span>{data.outcomeSummary}</p>
        <p><span class="key">Impacts:</span>{data.impactSummary}</p>
        <p><span class="key">Vulnerability:</span>{data.vulnerabilitySummary}</p>
        <p><span class="key">Resilience:</span>{data.resilienceSummary}</p>
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
    span.key{
        font-weight:700;
    }
</style>