<script lang="ts">
    export let format:string = 'primary'
</script>

<button class={"border rounded p-[2px] border-0" + " " + format} on:click>
    <span class="bg-black p-2 block rounded">
        <p class="">
            <slot />
        </p>
    </span>
</button>

<style lang="css">
    .primary, .primary p {
        @apply bg-gradient-to-br from-red-500 via-green-500 to-violet-500;
    }

    .primary p {
        @apply bg-clip-text text-transparent;
    }


    .secondary {
        @apply bg-white;
    }
    
</style>