<script lang="ts">
    let {
        value = $bindable(),
        min,
        max,
        step = '',
        name,
    } = $props();

</script>
<div class="range">
    <input
        type="range"
        class="slider"
        bind:value={value}
        {min}
        {max}
        {step}
        {name}/>
    <input class="number" bind:value={value}/>
</div>
<style>
    .range {
        display: flex;
        flex-wrap: wrap;
    }
    .slider {
        flex: 1 1 auto;
    }
    .number {
        flex: 0 1 25%;
        max-width: 5em;
        min-width: 2em;
    }
</style>
