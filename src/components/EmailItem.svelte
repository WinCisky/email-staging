<script lang="ts">
    export let id: string;
    export let onSelect: (id: string) => void;

    function handleClick() {
        onSelect(id);
    }
</script>

<div on:click={handleClick}>
    Item {id}
</div>
