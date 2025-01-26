<script lang="ts">
    import type { Coords } from './constants';
    import Draggable from './Draggable.svelte';
    import StylePopup from './StylePopup.svelte';
    let canvas: HTMLCanvasElement;
    let root: HTMLDivElement;
    let { bitmap }: {bitmap: void | ImageBitmap} = $props();

    let width = $state(0);
    let zoom = $state(calcInitialZoom());

    let croppedCoords: Coords = $state({
        left: 0,
        top: 0,
        width: 0,
        height: 0,
    });
    let showStylePopup = $state(false);

    let ctx: CanvasRenderingContext2D | null = $state(null);
    $effect(() => {
		ctx = canvas.getContext('2d');
    });

    $effect(() => {
        if (!bitmap) {
            return;
        }
        canvas.width = width = bitmap.width;
        canvas.height = bitmap.height;
        zoom = Math.min(window.innerHeight, window.innerWidth) / width;
        ctx?.drawImage(bitmap, 0, 0);
        root.scrollTo((root.scrollWidth - window.innerWidth) / 2, (root.scrollHeight - window.innerHeight) / 2);
    });

    function calcInitialZoom() {
        if (!bitmap) {
            return 0.9;
        }

        const aspect = bitmap.width / bitmap.height;
        return 0.9 * Math.min(1, aspect);
    }

    function cut(dimensions: Coords) {
        croppedCoords = dimensions;
        showStylePopup = true;
    }

    function changeZoom(e: WheelEvent) {
        if (!e.ctrlKey) {
            return;
        }
        e.preventDefault();
        const delta = e.deltaY;

        zoom *= Math.exp(-delta/1000);
    }
</script>
<div
    class="root"
    onwheel={changeZoom}
    bind:this={root}
    style:--zoom={zoom}
    style:--width={`${width}px`}>
    <div class="scalable">
        <canvas class="canvas" bind:this={canvas}></canvas>
        {#if showStylePopup}
            <StylePopup
                onclose={() => showStylePopup = false}
                coords={croppedCoords}
                {zoom}
                {bitmap}
            />
        {:else}
            <Draggable
                {zoom}
                coords={croppedCoords}
                cutCallback={cut}/>
        {/if}
    </div>
    <button class="zoom" onclick={() => {zoom = 1}} title="Reset to 100%">{(100*zoom).toFixed(0)}%</button>
</div>
<style>
    .root {
        overflow: auto;
        width: 100%;
        justify-content: center;
        box-sizing: border-box;
        --zoom: 1;
    }

    .scalable {
        position: relative;
        width: calc(var(--zoom) * var(--width));
        margin: 50vh 50vw;
    }

    .canvas {
        width: 100%;
        height: 100%;
        box-shadow: 0 0 3em rgba(0,0,0,0.5), inset 0 0 3em rgba(0,0,0,0.5);
    }

    .zoom {
        position: absolute;
        right: 1em;
        bottom: 1em;
        background: rgba(194, 194, 194, 0.5);
        display: block;
        border: none;
        border-radius: 0.5em;
        padding: 0.2em 0.5em;
        font-size: inherit;
    }
</style>
