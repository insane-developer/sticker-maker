<script lang="ts">
    let {
        files = $bindable(),
        text = 'Select file',
    }: {
        files: FileList,
        text?: string
    } = $props();

    let fileInput: HTMLInputElement | null = $state(null);
    function onclick() {
        fileInput?.click();
    }
</script>
<input type="file" bind:this={fileInput} bind:files={files}/>
<button onclick={onclick}>{text}</button>
<style>
    input {
        display: none;
    }
    button {
        font-size: inherit;
    }
</style>
