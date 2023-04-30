<script lang="ts">
  import type { Info } from '../types.js'

  export let info: Info
</script>

<header class="absolute top-0 flex justify-between p-6 w-full items-center">
  <h1 class="font-black text-3xl">ROYGBIV</h1>
  {#if info}
    <div class="flex items-center">
      <div
        style="background-color: #{info.color};"
        class="rounded-full w-8 h-8 overflow-hidden mr-2"
      />
      <div class="text-sm font-bold">
        {info.alias}
      </div>
    </div>
  {/if}
</header>
