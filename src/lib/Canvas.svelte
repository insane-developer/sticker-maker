<script lang="ts">
    import Dragable from './Dragable.svelte';
    let canvas: HTMLCanvasElement;
    let root: HTMLDivElement;
    let { bitmap }: {bitmap: void | ImageBitmap} = $props();

    let width = $state(0);
    let height = $state(0);
    let canvasWidth = $state(0);
    let zoom = $state(calcInitialZoom());

    let cropLeft = $state(0);
    let cropTop = $state(0);
    let cropWidth = $state(0);
    let cropHeight = $state(0);

    let ctx: CanvasRenderingContext2D | null = $state(null);
    $effect(() => {
		ctx = canvas.getContext('2d');
    });

    $effect(() => {
        console.log('resize', width, height);
        //zoom = canvasWidth / width;
        console.log(canvasWidth, width, zoom);
    });

    $effect(() => {
        if (!bitmap) {
            console.log('!bitmap');
            return;
        }
        canvas.width = width = bitmap.width;
        canvas.height = height = bitmap.height;
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

    function cut(dimensions) {
        const {
            left: sx,
            top: sy,
            width: sw,
            height: sh
        } = dimensions;
        const image = ctx?.getImageData(sx, sy, sw, sh);

        image
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
    style:--zoom={zoom}>
    <div class="scalable">
        <canvas class="canvas" bind:this={canvas}
            bind:offsetWidth={canvasWidth}
        ></canvas>
        <Dragable
            zoom={zoom}
            bind:left={cropLeft}
            bind:top={cropTop}
            bind:width={cropWidth}
            bind:height={cropHeight}
            cutCallback={cut}/>
    </div>
</div>
<style>
    .root {
        overflow: auto;
        background: #999;
        width: 100%;
        justify-content: center;
        box-sizing: border-box;
        --zoom: 1;
    }

    .scalable {
        position: relative;
        width: calc(var(--zoom) * min(100vh, 100vw));
        margin: 50vh 50vw;
    }

    .canvas {
        width: 100%;
        height: 100%;
    }

    .select {
        position: absolute;
        border: 1px dashed #fff;
    }
</style>
