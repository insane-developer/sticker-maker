<script lang="ts">
    import Dragable, { type Coords } from './Dragable.svelte';
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
            console.log('!bitmap');
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
        console.log('before cut', dimensions);
        croppedCoords = dimensions;
        showStylePopup = true;
    }

    function changeZoom(e: WheelEvent) {
        if (!e.ctrlKey) {
            return;
        }
        e.preventDefault();
        const delta = e.deltaY;
        console.log();
        zoom *= Math.exp(-delta/1000);
        console.log(delta, zoom);
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
            <Dragable
                {zoom}
                coords={croppedCoords}
                cutCallback={cut}/>
        {/if}
        <!-- {#if croppedPart}
            <StylePopup
                top={cropTop}
                left={cropLeft}
                width={cropWidth}
                height={cropHeight}
                coords={croppedCoords}
                bind:image={croppedPart}
            />
        {:else}
            <Dragable
                zoom={zoom}
                bind:left={cropLeft}
                bind:top={cropTop}
                bind:width={cropWidth}
                bind:height={cropHeight}
                cutCallback={cut}/>
        {/if} -->
    </div>
    <span class="zoom">{zoom.toPrecision(2)}</span>
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

    .select {
        position: absolute;
        border: 1px dashed #fff;
    }
    .zoom {
        position: absolute;
        right: 1em;
        bottom: 1em;
    }
</style>
