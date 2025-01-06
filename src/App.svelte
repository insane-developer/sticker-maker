<script lang="ts">
    import Canvas from './lib/Canvas.svelte';
    import FileChooser from './lib//FileChooser.svelte';

    let bitmap: ImageBitmap | void = $state();
</script>

<main>
    {#if bitmap}
        <Canvas bitmap={bitmap}/>
    {:else}
        <FileChooser bind:bitmap={bitmap}/>
    {/if}
</main>

<style>
    main {
        display: flex;
        min-height: 100%;
        width: 100%;
        height: 100%;
    }
</style>
