<script lang="ts">
  import Button from './Button.svelte'
  import Slide from './Slide.svelte'

  export let finish = (args: any) => {} // no-operation function

  type Slides = typeof slides
  type SlideStep = Slides[number]
  type SlideDirection = 'right' | 'left'

  const slides = ['0', '1', '2'] as const
  let slide: SlideStep = '0'
  let previousSlide: SlideStep = '0'
  let incomplete = true

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

  // prism name
  let label: string
  // Default form to two members to start
  let members = [
    {
      name: '',
      destination: '',
      split: 0,
      percentage: 0
    },
    {
      name: '',
      destination: '',
      split: 0,
      percentage: 0
    }
  ]

  $: {
    incomplete = members.some((member) => !member.name || !member.destination || !member.split)
    console.log('incomplete = ', incomplete)
  }

  // Calaculate percentages for each member
  $: {
    if (members) {
      const allSplits = members.map((member) => member.split).reduce((a, b) => a + b)
      const updatedMembers = members.map((member) => {
        return {
          ...member,
          percentage: member.split ? (member.split / allSplits) * 100 : 0
        }
      })

      members = updatedMembers
    }
  }

  // @TODO
  function addMember() {}
</script>

<!-- Name your prism -->
{#if slide === '0'}
  <Slide direction={slideDirection}>
    <div>
      <h1 class="text-4xl">Name your prism</h1>
      <div class="mt-6">
        <label class="font-medium" for="address">Name</label>
        <textarea
          id="address"
          class="mt-2 w-full p-2 rounded"
          rows="1"
          bind:value={label}
          placeholder="bitcoin-dev-fund"
        />
      </div>
    </div>
    <div class="mt-8">
      <Button format="secondary" fullWidth={true} on:click={() => next()}>Next</Button>
    </div>
  </Slide>
{/if}
<!-- Add prism members  -->
{#if slide === '1'}
  <Slide direction={slideDirection}>
    <div class="flex flex-row gap-4 w-full">
      {#each members as member}
        <div class="flex flex-wrap border border basis-1/2 p-6">
          <div class="mt-4 w-full text-sm flex flex-col gap-4">
            <h3>Member</h3>
            <label class="font-medium mb-1 block" for="address">Name</label>
            <textarea
              id="address"
              class="border w-full p-2 rounded w-full"
              rows="1"
              cols="200"
              bind:value={member.name}
              placeholder=""
            />
          </div>
          <div class="mt-4 w-full text-sm">
            <label class="font-medium mb-1 block" for="address">Destination</label>
            <textarea
              id="address"
              class="border w-full p-2 rounded"
              rows="1"
              bind:value={member.destination}
              placeholder="pubkey"
            />
          </div>
          <div class="mt-4 w-full text-sm">
            <label class="font-medium mb-1 block" for="address">Split</label>
            <input
              id="address"
              class="border w-full p-2 rounded"
              type="number"
              bind:value={member.split}
              placeholder="weight"
            />
          </div>
          <div class="mt-4 w-full text-sm">
            <label class="font-medium mb-1 block" for="address">Share</label>
            {member.percentage.toFixed(1)}%
          </div>
        </div>
      {/each}
    </div>
    <div class="mt-8 flex w-full justify-between">
      <Button format="secondary" on:click={() => back()}>Back</Button>
      <Button disabled={incomplete} format="secondary" on:click={() => next()}>Next</Button>
    </div>
  </Slide>
{/if}
<!-- Summary  -->
{#if slide === '2'}
  <Slide direction={slideDirection}>
    <div>
      <h1 class="text-4xl">Summary</h1>
      <!-- <div class="mt-6">
        <label class="font-medium" for="address">Name</label>
        <textarea
          id="address"
          class="mt-2 w-full p-2 rounded"
          rows="1"
          bind:value={label}
          placeholder="bitcoin-dev-fund"
        />
      </div> -->
    </div>
    <div class="flex gap-2 w-full items-end justify-end">
      <Button format="secondary" on:click={() => back()}>Back</Button>
      <Button format="primary" on:click={() => finish({ label, members })}>Finish</Button>
    </div>
  </Slide>
{/if}
