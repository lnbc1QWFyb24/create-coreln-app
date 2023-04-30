<script lang="ts">
    import type Lnmessage from 'lnmessage'
    export let ln: Lnmessage
</script>

<header class="absolute top-0 flex justify-between p-6 w-full items-center">
    <h1 class="font-black text-3xl">ROYGBIV</h1>
    {#if ln}
      <!-- <div class="text-sm">
        Browser Id: {`${ln.publicKey.slice(0, 8)}...${ln.publicKey.slice(-8)}`}
      </div> -->
      <p>
        @TODO - Add button to add a prisim - Add list of members in the prism (name, split) - list
        of members in the container
      </p>
    {/if}
  </header>