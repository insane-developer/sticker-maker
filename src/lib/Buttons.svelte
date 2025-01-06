<script lang="ts">

    let {
        bitmap = $bindable(),
        cut
    } = $props();

    let files = $state<FileList>();

    $effect(() => {
        const file = files?.item(0);
        if (!file) {
            return;
        }
        console.log('file', file);

        window.createImageBitmap(file)
            .then(data => {
                bitmap = data;
            });
    });
</script>
<input type="file" bind:files={files} />
<button class="cut" onclick={cut}>cut!</button>
