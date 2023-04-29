<script lang="ts">
  import Lnmessage from 'lnmessage'
  import { parseNodeAddress } from './utils.js'

  let ln: Lnmessage
  let connectionStatus$: Lnmessage['connectionStatus$']

  $: if (ln) {
    connectionStatus$ = ln.connectionStatus$
  }

  $: {
    if ($connectionStatus$ === 'connected') {
      modalOpen = false
    }
  }

  let address = '03093b030028e642fc3b9a05c8eb549f202958e92143da2e85579b92ef0f49cc7d@localhost:7272'
  let rune = '7OQKm3FL-zIftx1z29E7KFGyVOgAY54ZyeWRtx8e2Q09MQ=='
  let method: string
  let params: string
  let result: string

  let modalOpen = false
  let connecting = false

  async function connect() {
    const { publicKey, ip, port } = parseNodeAddress(address)

    // https://github.com/aaronbarnardsound/lnmessage#initialisation
    ln = new Lnmessage({
      // The public key of the node you would like to connect to
      remoteNodePublicKey: publicKey,
      // WebSocket proxy endpoint to connect through if running in prod
      // wsProxy: 'wss://<WEBSOCKET_PROXY>',
      // The IP address of the node
      ip,
      // The port of the node, defaults to 9735
      port,
      // connect directly to a node without TLS
      wsProtocol: 'ws:',
      logger: {
        info: console.log,
        error: console.error,
        warn: console.warn
      }
    })

    // initiate the connection to the remote node
    connecting = true
    await ln.connect()
    connecting = false
  }

  async function request() {
    let parsedParams: unknown | undefined

    try {
      parsedParams = params ? JSON.parse(params) : undefined

      const requestResult = await ln.commando({
        method,
        params: parsedParams,
        rune
      })

      result = JSON.stringify(requestResult, null, 2)
    } catch (error) {
      const { message } = error as { message: string }
      alert(message)
      return
    }
  }
</script>

<header class="absolute flex justify-between p-6 w-full items-center">
  <h1 class="font-bold text-3xl">ROYGBIV</h1>
  {#if ln}
    <div class="text-sm">
      Browser Id: {`${ln.publicKey.slice(0, 8)}...${ln.publicKey.slice(-8)}`}
    </div>
  {/if}
</header>

<main class="w-screen h-screen flex flex-col items-center justify-center p-4 relative">
  <!-- Modal -->
  {#if modalOpen}
    <div class="w-1/2 max-w-lg border-2 p-6 rounded relative">
      <!-- svelte-ignore a11y-click-events-have-key-events -->
      <div
        class="p-4 cursor-pointer absolute top-2 right-2 right"
        on:click={() => (modalOpen = false)}
      >
        X
      </div>
      <h1 class="text-lg">Connect your Node</h1>
      <!-- Address -->
      <div class="mt-4 w-full text-sm">
        <label class="font-medium mb-1 block" for="address">Address</label>
        <textarea
          id="address"
          class="border w-full p-2 rounded"
          rows="3"
          bind:value={address}
          placeholder="033f4bbfcd67bd0fc858499929a3255d063999ee23f4c5e12b8b1089e132b3e408@localhost:7272"
        />
      </div>
      <!-- Rune -->
      <div class="w-full mt-4 text-sm">
        <label class="font-medium mb-1 block" for="rune">Rune</label>
        <textarea
          id="rune"
          class="border w-full p-2 rounded"
          rows="2"
          bind:value={rune}
          placeholder="O2osJxV-6lGUgAf-0NllduniYbq1Zkn-45trtbx4qAE9MA=="
        />
      </div>
      <!-- Connect Button -->
      <div class="flex items-center justify-between w-full mt-4">
        <button
          on:click={connect}
          disabled={!address}
          class="border border-purple-500 rounded py-1 px-4 disabled:opacity-20 hover:shadow-md active:shadow-none"
          >{$connectionStatus$ === 'connecting' ? '...' : 'Connect'}</button
        >

        {#if connectionStatus$}
          <div class="flex items-center">
            <div class="text-sm">{$connectionStatus$}</div>
            <div
              class:bg-green-500={$connectionStatus$ === 'connected'}
              class:bg-yellow-500={$connectionStatus$ === 'connecting' ||
                $connectionStatus$ === 'waiting_reconnect'}
              class:bg-red-500={$connectionStatus$ === 'disconnected'}
              class="w-3 h-3 rounded-full ml-1 transition-colors"
            />
          </div>
        {/if}
      </div>
    </div>
  {/if}

  {#if !modalOpen && $connectionStatus$ !== 'connected'}
    <div class="">
      <button on:click={() => (modalOpen = !modalOpen)}>OPEN MODAL</button>
    </div>
  {/if}
</main>
