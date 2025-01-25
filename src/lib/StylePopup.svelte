<script lang="ts">
    import { MAX_STICKER_SIZE } from "./constants";
    import type { Coords } from "./constants";

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

    let borderWidth = $state(10);
    let borderRadius = $state(10);

    const {
        left,
        top,
        width,
        height,
    } = coords;
    let ctx: CanvasRenderingContext2D|null = $state(null);
    let scaleFactor = $state(1);
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
        let factor = MAX_STICKER_SIZE / Math.max(width, height);
        scaleFactor = Math.min(1, factor);
        const scaledWidth = scaleFactor * width;
        const scaledHeight = scaleFactor * height;
        patternCanvas.width = scaledWidth;
        patternCanvas.height = scaledHeight;

        patternContext?.drawImage(bitmap, left, top, width, height, 0, 0, scaledWidth, scaledHeight);
        fill = ctx?.createPattern(patternCanvas, "no-repeat") || null;
    });


    $effect(() => {
        if (!bitmap) {
            return;
        }

        canvas.width = scaleFactor * width;
        canvas.height = scaleFactor * height;
        if (!ctx || !fill) {
            return;
        }
        const scaledBorderWidth = borderWidth / scaleFactor | 0;
        const scaledBorderRadius = borderRadius / scaleFactor;

        ctx.beginPath();
        ctx.fillStyle = fill;
        ctx.lineWidth = scaledBorderWidth;
        ctx.strokeStyle = '#fff';

        ctx.roundRect(scaledBorderWidth / 2, scaledBorderWidth / 2, scaleFactor * width - scaledBorderWidth, scaleFactor * height - scaledBorderWidth, scaledBorderRadius);
        ctx.stroke();
        ctx.fill();
        ctx.closePath();
    });

    function save() {
        const img = canvas.toDataURL('image/png');
        const a = document.createElement('a');
        a.target='_blank';
        a.download = `sticker-${new Date().toISOString()}.png`;
        a.href = img;
        a.click();
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
    <button class="close" aria-label="close" onclick={close}></button>
    <canvas class="canvas" class:upscale={Math.max(width, height) < MAX_STICKER_SIZE} bind:this={canvas}>
    </canvas>
    <div class="controls">
        <div>
            <label class="over" for="border">border width</label>
            <input type="range" bind:value={borderWidth} min="0" max="{scaleFactor*Math.min(width,height)/4}" name="border"/><input bind:value={borderWidth}/>
        </div>
        <div>
            <label class="over" for="corners">border radius</label>
            <input type="range" bind:value={borderRadius} min="0" max="{scaleFactor*Math.min(width,height)/2 - borderWidth/2}" name="corners"/><input bind:value={borderRadius}/>
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
        right: 1em;
        top: 1em;
        padding: 0.5em;
        border: none;
        background: url(../assets/cross.svg) 50% 50%/contain no-repeat;
    }

    .canvas {
        width: 100%;
        max-width: var(--width);
    }

    label.over {
        display: block;
    }
    button {
        margin-top: 0.5em;
    }
</style>
