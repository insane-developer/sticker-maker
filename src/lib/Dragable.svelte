<script lang="ts">

    let {
        zoom = 1,
        left=$bindable(0),
        top=$bindable(0),
        width=$bindable(0),
        height=$bindable(0),
        cutCallback
    } = $props();

    let isSelecting = $state(false);
    let isDragging = $state(false);
    let areaElem = $state();
    let draggableElem = $state();

    function startSelecing(e: PointerEvent) {
        if (e.target !== areaElem) {
            return;
        }
        left = e.offsetX;
        top = e.offsetY;
        width = height = 0;
        isSelecting = true;
        isDragging = false;
    }
    function selecting(e: PointerEvent) {
        width = e.offsetX - left;
        height = e.offsetY - top;
        width = height = Math.min(width, height);
    }
    function endSelection(e: PointerEvent) {
        isSelecting = false;
    }

    function dragStart(e: PointerEvent) {
        if (e.target !== draggableElem) {
            return;
        }
        console.log('drag!');
        e.stopPropagation();
        isDragging = true;
    }
    function dragging(e: PointerEvent) {
        left += e.movementX;
        top += e.movementY;
    }
    function dragEnd(e: PointerEvent) {
        console.log('drag!');
        isDragging = false;
        e.stopPropagation();
    }

    function cut(e: MouseEvent) {
        e.stopPropagation();
        cutCallback({
            left,
            top,
            height,
            width
        });
        console.log(left, top, width, height);
    }
</script>
<div class="dragable-area"
    bind:this={areaElem}
    onpointerdown={startSelecing}
    onpointermove={isSelecting ? selecting : null}
    onpointerup={endSelection}
    >
    {#if width && height}
        <div class="dragable"
            bind:this={draggableElem}
            class:inactive={isSelecting}
            class:clear={isDragging}
            style:left={`${left}px`}
            style:width={`${width}px`}
            style:top={`${top}px`}
            style:height={`${height}px`}
            onpointerdown={dragStart}
            onpointermove={isDragging ? dragging : null}
            onpointerup={dragEnd}
        >
            <button class="button" onclick={cut}>cut</button>
            {(width*zoom).toFixed(0)}x{(height*zoom).toFixed(0)}
        </div>
    {/if}
</div>
<style>
.dragable-area {
    position: absolute;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    overflow: hidden;
}
.dragable {
    position: absolute;
    border: 1px dashed #afa;
    background: rgba(0,128,0, 0.1);
    color: #afa;
    text-align: center;
    overflow: visible;
}
.dragable:hover .button {
    display: block;
    position:absolute;
    bottom: 100%;
    right: 0;
}
.button, .dragable.clear .button {
    display: none;
}
.inactive {
    pointer-events: none;
}
</style>
