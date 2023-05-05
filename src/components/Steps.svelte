<script lang="ts">
  import Button from './Button.svelte'
  import Icon from './Icon/Icon.svelte'
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
    <h1 class="text-3xl mb-6 text-center">Choose Prism Members</h1>
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
          <!-- Name, Split & Share (%) -->
          <!-- @TODO - Update functionality to allow scrolling of prism members on x axis -->
          <div class="flex flex-between items-center gap-4">
            <!-- Name -->
            <!-- @TODO input validation for name -->
            <div class="mt-6 w-full">
              <label class="mb-1 block" for="name">Name</label>
              <input
                id="name"
                class="border w-full p-2 rounded"
                type="text"
                bind:value={member.name}
                placeholder="Alice"
              />
            </div>
            <!-- Split -->
            <div class="mt-6">
              <label class="mb-1 block" for="split">Split</label>
              <input
                id="split"
                class="border w-full p-2 rounded"
                type="number"
                bind:value={member.split}
                placeholder="weight"
              />
            </div>
            <!-- Share -->
            <div class="mt-6 w-20">
              <label class="mb-1 block" for="percentage">Share</label>
              <p class="p-2 pl-0">{member.percentage.toFixed(1)}%</p>
            </div>
          </div>
          <!-- Destination -->
          <!-- TODO - input validation for pubkey -->
          <div class="mt-6">
            <label class="mb-1 block" for="destination">Destination</label>
            <textarea
              id="destination"
              class="border w-full p-2 rounded"
              rows="1"
              bind:value={member.destination}
              placeholder="pubkey"
            />
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
