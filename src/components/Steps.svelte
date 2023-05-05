<script lang="ts">
  import Button from './Button.svelte'
  import Icon from './Icon/Icon.svelte'
  import close from '../icons/close.js'
  import Slide from './Slide.svelte'

  export let finish = (args: any) => {} // no-operation function

  type Slides = typeof slides
  type SlideStep = Slides[number]
  type SlideDirection = 'right' | 'left'

  const slides = ['0', '1', '2'] as const
  let slide: SlideStep = '1'
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
  }

  // Calculate percentages for each member
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

  function addMember() {
    members = [
      ...members,
      {
        name: '',
        destination: '',
        split: 0,
        percentage: 0
      }
    ]
  }

  function deleteMember(index: number) {
    members = members.filter((member) => members.indexOf(member) !== index)
  }
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
      <Button disabled={!label} format="secondary" fullWidth={true} on:click={() => next()}
        >Next</Button
      >
    </div>
  </Slide>
{/if}
<!-- Add prism members  -->
{#if slide === '1'}
  <Slide direction={slideDirection}>
    <h1 class="text-3xl mb-6 text-center">Select Prism Members</h1>
    <div class="flex flex-row gap-6 w-full">
      {#each members as member, i}
        <div class="flex flex-col border rounded p-6 w-96 overflow">
          <!-- Header -->
          <div class="flex justify-between w-full">
            <h3 class="text-2xl">Member {i + 1}</h3>
            {#if members.length > 2}
              <!-- svelte-ignore a11y-click-events-have-key-events -->
              <div class="w-8 cursor-pointer" on:click={() => deleteMember(i)}>
                <Icon icon="Cross" />
              </div>
            {/if}
          </div>
          <!-- Name -->
          <div class="mt-6 w-full flex flex-col">
            <label class="mb-1 block" for="address">Name</label>
            <textarea
              id="address"
              class="border w-full p-2 rounded w-full"
              rows="1"
              cols="200"
              bind:value={member.name}
              placeholder=""
            />
          </div>
          <!-- Destination -->
          <div class="mt-6 w-full text-sm">
            <label class="mb-1 block" for="address">Destination</label>
            <textarea
              id="address"
              class="border w-full p-2 rounded"
              rows="1"
              bind:value={member.destination}
              placeholder="pubkey"
            />
          </div>
          <!-- Split -->
          <div class="mt-6 w-full text-sm">
            <label class="mb-1 block" for="address">Split</label>
            <input
              id="address"
              class="border w-full p-2 rounded"
              type="number"
              bind:value={member.split}
              placeholder="weight"
            />
          </div>
          <!-- Percentage -->
          <div class="mt-6 w-full text-sm">
            <label class="mb-1 block" for="address">Share</label>
            {member.percentage.toFixed(1)}%
          </div>
        </div>
      {/each}
    </div>
    <div class="mt-8 flex w-full justify-between">
      <Button format="secondary" on:click={() => back()}>Back</Button>
      <Button format="primary" on:click={() => addMember()}>+1</Button>
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
    <div class="flex w-full items-end justify-end">
      <Button format="secondary" on:click={() => back()}>Back</Button>
      <Button format="primary" on:click={() => finish({ label, members })}>Finish</Button>
    </div>
  </Slide>
{/if}
