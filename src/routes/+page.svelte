<script lang="ts">
  import Lnmessage from 'lnmessage'
  import { parseNodeAddress } from '../utils.js'
  import Header from '../components/Header.svelte'
  import Slide from '../components/Slide.svelte'
  import { goto } from '$app/navigation'
  import type { Info, Prism } from '../types.js'
  import { fade } from 'svelte/transition'

  let ln: Lnmessage
  let connectionStatus$: Lnmessage['connectionStatus$']

  $: if (ln) {
    connectionStatus$ = ln.connectionStatus$
  }

  $: {
    if ($connectionStatus$ === 'connected') {
      modalOpen = null
    }
  }

  // let address = ''
  // let rune = ''
  let address = '0211339e6b9db0e7e14d19bcd612d06ba26f793e8d049ebe9a99a3966668f4d81f@localhost:7272'
  let rune = '0Ew8MyncAizPDi6rs8fp4vJCl7mQYNA8fVew2lwTFnE9NQ=='
  let bolt12 = ''
  let info: Info

  let modalOpen: 'connect' | 'qr' | null = null
  let connecting = false

  type Slides = typeof slides
  type SlideStep = Slides[number]
  type SlideDirection = 'right' | 'left'

  const slides = ['0', '1', '2', 'summary'] as const
  let slide: SlideStep = '0'
  let previousSlide: SlideStep = '0'

  $: slideDirection = (
    slides.indexOf(previousSlide) > slides.indexOf(slide) ? 'right' : 'left'
  ) as SlideDirection

  function back() {
    previousSlide = slides[slides.indexOf(slide) - 2]
    slide = slides[slides.indexOf(slide) - 1]
  }

  function next(to = slides[slides.indexOf(slide) + 1]) {
    previousSlide = slide
    slide = to
  }

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

    const infoResult = await request('getinfo')
    info = infoResult as Info
  }

  async function request(method: string, params?: unknown): Promise<unknown> {
    try {
      const result = await ln.commando({
        method,
        params,
        rune
      })

      return result
    } catch (error) {
      const { message } = error as { message: string }
      console.log(message)
    }
  }

  async function createPrism(prism: Prism) {
    try {
      const result = await request('createprism', prism)
      bolt12 = (result as { bolt12: string }).bolt12
    } catch (error) {
      console.log(error)
    }
  }
</script>

<svelte:head>
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" />
  <link
    href="https://fonts.googleapis.com/css2?family=Inter:wght@400;700;900&display=swap"
    rel="stylesheet"
  />
</svelte:head>
<main class="w-screen h-screen flex flex-col items-center justify-center relative">
  <Header {info} />

  <!-- Button to open connect modal -->
  {#if $connectionStatus$ !== 'connected' && !modalOpen}
    <div class="">
      <button class="px-4 py-2 border rounded" on:click={() => (modalOpen = 'connect')}
        >Connect</button
      >
    </div>
  {/if}

  <!-- Prism Steps -->
  {#if $connectionStatus$ === 'connected'}
    <div class="border max-w-lg w-full p-10">
      {#if slide === '0'}
        <Slide direction={slideDirection}>
          Create Prisom
          <button class="border p-2" on:click={() => next()}>Next</button>
        </Slide>
      {/if}
      {#if slide === '1'}
        <Slide direction={slideDirection}>
          <button class="border p-2" on:click={() => back()}>Back</button>
          Add Members
          <button class="border p-2" on:click={() => next()}>Next</button>
        </Slide>
      {/if}
      {#if slide === '2'}
        <Slide direction={slideDirection}
          ><button class="border p-2" on:click={() => back()}>Back</button>Finish</Slide
        >
      {/if}
    </div>
  {/if}
</main>

<!-- MODALS -->
{#if modalOpen === 'connect'}
  <div
    transition:fade
    class="w-full h-full top-0 absolute backdrop-blur-sm bg-black/30 flex flex-col items-center justify-center z-10"
  >
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <button class="p-4 cursor-pointer absolute top-4 right-4" on:click={() => (modalOpen = null)}>
      X
    </button>
    <div class="w-1/2 max-w-lg border-2 p-6 rounded relative">
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
  </div>
{/if}
