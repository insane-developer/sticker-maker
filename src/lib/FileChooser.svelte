<script lang="ts">
    let {
        bitmap = $bindable()
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
</script>
<div class="file-chooser"
    class:in-drag={inDrag}
    ondrop={onDrop}
    ondragover={onDragover}>
    <input type="file" bind:files={files} />
</div>
<style>
    .file-chooser {
        display: flex;
        justify-content: center;
        align-items: center;
        width: 100%;
        border: 3px dashed gray;
        box-sizing: border-box;
        margin: 2em;
    }

    .in-drag {
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
