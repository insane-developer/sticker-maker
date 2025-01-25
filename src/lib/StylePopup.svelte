<script lang="ts">
    import type { Coords } from "./Dragable.svelte";

    let {
        zoom,
        coords,
        bitmap,
        onclose,
    }: {
        zoom: number;
        coords: Coords;
        bitmap: void | ImageBitmap;
        onclose: () => void;
    } = $props();

    let upscale = $state(false);
    let borderWidth = $state(10);
    let borderRadius = $state(10);

    const {
        left,
        top,
        width,
        height,
    } = coords;
    let ctx: CanvasRenderingContext2D|null = $state(null);
    // let scaleFactor = $state(1);
    let canvas: HTMLCanvasElement;
    let patternCanvas = document.createElement('canvas');
    let fill: CanvasPattern | null = $state(null);
    const patternContext = patternCanvas.getContext("2d");
    if (!patternContext) {
        throw new Error('Cannot get canvas context');
    }

    $effect(() => {
		ctx = canvas.getContext('2d');
    });

    $effect(() => {
        if (!bitmap) {
            return;
        }
        // let factor = 500 / Math.max(width, height);
        // console.log(factor);
        // scaleFactor = upscale ? factor : Math.min(1, factor);
        patternCanvas.width = width;
        patternCanvas.height = height;

        patternContext?.drawImage(bitmap, left, top, width, height, 0, 0, width, height);
        fill = ctx?.createPattern(patternCanvas, "no-repeat") || null;
    });


    $effect(() => {
        if (!bitmap) {
            return;
        }
        console.log('got', bitmap, coords);
        canvas.width = width;
        canvas.height = height;
        if (!ctx || !fill) {
            return;
        }

        ctx.beginPath();
        ctx.fillStyle = fill;
        ctx.lineWidth = borderWidth;
        ctx.strokeStyle = '#ffff';
        ctx.roundRect(borderWidth / 2, borderWidth / 2, width - borderWidth, height - borderWidth, borderRadius);
        ctx.stroke();
        ctx.fill();
        ctx.closePath();
    });

    function save() {

    }
    function close() {
        console.log('close');
        onclose();
    }
</script>
<div
    class="style-popup"
    style:--width={`${width*zoom}px`}
    style:top={`${top*zoom}px`}
    style:left={`${left*zoom}px`}
    >
    <button class="close" aria-label="close" onclick={close}>x</button>
    <canvas class="canvas" bind:this={canvas}>
    </canvas>
    <div class="controls">
        <div>
            <label for="upscale"><input type="checkbox" bind:checked={upscale} name="upscale"/>upscale</label>
        </div>
        <div>
            <label class="over" for="border">border width</label>
            <input type="range" bind:value={borderWidth} min="0" max="{Math.min(width,height)/4}" name="border"/><input bind:value={borderWidth}/>
        </div>
        <div>
            <label class="over" for="corners">border radius</label>
            <input type="range" bind:value={borderRadius} min="0" max="{Math.min(width,height)/2 - borderWidth/2}" name="corners"/><input bind:value={borderRadius}/>
        </div>
        <div>
            <button onclick={save}>save</button>
        </div>
    </div>
</div>
<style>
    .style-popup {
        position: absolute;
        background: rgba(255,255,255,0.7);
        border-radius: 2em;
        box-shadow: 0 0 2em rgba(0,0,0,0.3);
        min-width: 10em;
        padding: 2em;
        margin: -2em;
        width: var(--width);
    }
    .close {
        position: absolute;
        right: .5em;
        top: .5em;
        padding: 0.5em;
        border: none;
        background: none;
    }
    .canvas {
        width: 100%;
        max-width: var(--width);
    }
    input[type="checkbox"] {
        margin-left: 0;
    }
    label.over {
        display: block;
    }
</style>
