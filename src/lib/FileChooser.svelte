<script lang="ts">
    let {
        bitmap = $bindable(),
        hidden,
    } = $props();

    let files = $state<FileList>();
    let inDrag = $state(false);

    $effect(() => {
        const file = files?.item(0);
        if (!file) {
            return;
        }
        bindFile(file);
    });

    function bindFile(file: File) {
        console.log('file', file);

        window.createImageBitmap(file)
            .then(data => {
                bitmap = data;
            });
    }
    function onDragover(e: DragEvent) {
        e.preventDefault();
        inDrag = true;
    }
    function onDrop(e: DragEvent) {
        console.log('drop');

        inDrag = false;
        e.preventDefault();
        if (e.dataTransfer !== null) {
            bindFile(e.dataTransfer.files[0]);
        }
    }
    function onDragEnd() {
        console.log('dragend');
        inDrag = false;
    }
</script>
<svelte:window
    ondrop={onDrop}
    ondragover={onDragover}
    onpointerout={onDragEnd}
    ondragend={onDragEnd}/>
<div class="file-chooser" class:in-drag={inDrag} class:hidden={hidden}>
    <div class="border">
        <div class="text">
            Drop the image here
            <span class:hidden={hidden}>or select with <input type="file" bind:files={files} title="select"/></span>
        </div>
    </div>
</div>
<style>
    .file-chooser {
        position: fixed;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        padding: 2em;
        background: #fff;
    }
    .hidden {
        visibility: hidden;
    }

    .in-drag {
        visibility: visible;
        z-index: 1;
    }

    .border {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        border: 3px dashed gray;
        box-sizing: border-box;
        height: 100%;
    }
    .text {
        width: 15em;
        text-align: left;
    }
    .in-drag .border {
        animation: blink 1s infinite;
    }

    @keyframes blink{
        from {
            border-color: gray;
        }
        50% {
            border-color: rgb(233, 180, 81);
            background: #fdf1ba;
        }
        to {
            border-color: gray;
        }
    }

    input {
        border: none;
    }
</style>
