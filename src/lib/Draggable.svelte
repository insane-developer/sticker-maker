<script lang="ts">
    import { MAX_STICKER_SIZE, type Coords } from "./constants";

    let {
        zoom = 1,
        cutCallback,
        coords,
    }: {
        zoom: number;
        cutCallback: () => void;
        coords: Coords;
    } = $props();
    let left = $state(coords?.left || 0);
    let top = $state(coords?.top || 0);
    let width = $state(coords?.width || 0);
    let height = $state(coords?.height || 0);

    let isSelecting = $state(false);
    let isDragging = $state(false);
    let areaElem = $state();
    let draggableElem = $state();

    function pointerdown(e: PointerEvent) {
        if (e.target === draggableElem) {
            e.stopPropagation();
            return dragStart();
        }
        if (e.target !== areaElem) {
            return;
        }
        left = e.offsetX/zoom;
        top = e.offsetY/zoom;
        width = height = 0;
        isSelecting = true;
        isDragging = false;
    }
    function selecting(e: PointerEvent) {
        width = e.offsetX/zoom - left;
        height = e.offsetY/zoom - top;
        if (e.shiftKey) {
            width = height = Math.min(width, height);
        }
    }
    function pointerup(e: PointerEvent) {
        isSelecting = false;
        isDragging = false;
    }

    function dragStart() {
        console.log('drag!');
        isDragging = true;
    }
    function dragging(e: PointerEvent) {
        left += e.movementX/zoom;
        top += e.movementY/zoom;
    }

    function cut(e: MouseEvent) {
        e.stopPropagation();
        cutCallback({
            left: left,
            top: top,
            height: height,
            width: width,
        });
        console.log(left, top, width, height);
    }
</script>
<svelte:window
    onpointerdown={pointerdown}
    onpointermove={isSelecting ? selecting : isDragging ? dragging : null}
    onpointerup={pointerup}/>
<div class="draggable-area" bind:this={areaElem}>
    {#if width && height}
        <div class="draggable"
            bind:this={draggableElem}
            class:inactive={isSelecting}
            class:clear={isDragging}
            style:left={`${left*zoom}px`}
            style:width={`${width*zoom}px`}
            style:top={`${top*zoom}px`}
            style:height={`${height*zoom}px`}
        >
            <button class="button" onclick={cut}>cut</button>
            {(width).toFixed(0)}x{(height).toFixed(0)}
            {#if Math.max(width, height) > MAX_STICKER_SIZE}
                {@const factor = MAX_STICKER_SIZE / Math.max(width, height)}
                <span class="downscale-warning">will be downscaled to {(width * factor).toFixed(0)}x{(height * factor).toFixed(0)}</span>
            {/if}
        </div>
    {/if}
</div>
<style>
.draggable-area {
    position: absolute;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
}
.draggable {
    position: absolute;
    border: 1px dashed #afa;
    background: rgba(0,128,0, 0.1);
    color: #afa;
    text-shadow: 0 0 5px rgba(0,0,0,0.5);
    text-align: center;
    overflow: visible;
}

.draggable:hover {
    cursor: grab;
}

.button {
    position: absolute;
    display: block;
    bottom: 40%;
    right: 0;
    left: 0;
    margin: auto;
    width: 40%;
    height: 3em;
    min-width: 3em;
    max-width: 10em;

    border: 1px #afa dashed;
    border-radius: 5px;
    background: rgba(170, 255, 170, 0);

    color: inherit;
    text-shadow: 0 0 5px rgba(0,0,0,0.5);
    cursor: pointer;
    transition: background-color 0.3s;
}

.draggable:hover .button {
    display: block;
}
.button:hover {
    background: rgba(170, 255, 170, 0.1);
}

.button, .draggable.clear .button {
    display: none;
}
.inactive {
    pointer-events: none;
}

.downscale-warning {
    display: block;
    color: rgb(233, 233, 79);
}
</style>
