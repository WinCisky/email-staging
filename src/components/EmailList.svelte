<script lang="ts">
    import Item from './EmailItem.svelte';
    export let items: string[];
    export let onItemSelected: (id: string) => void;

    function handleSelect(id: string) {
        onItemSelected(id);
    }
</script>

{#each items as item}
    <Item id={item} onSelect={handleSelect} />
{/each}
