<script lang="ts">
  import minus from '../icons/minus'
  import plus from '../icons/plus'
  import type { Member } from '../types'
  import Button from './Button.svelte'
  import Slide from './Slide.svelte'

  export let finish = (args: any) => {} // no-operation function

  type Slides = typeof slides
  type SlideStep = Slides[number]
  type SlideDirection = 'right' | 'left'

  let slides = [0, 1, 2, 3]
  let slide: SlideStep = 0
  let previousSlide: SlideStep = 0
  let label: string = '' // prism name
  let memberCount = 2 // minimum of 2 members in a prism
  let members: Member[] = []

  // Update members & slides based on member count input on first slide
  $: {
    members = new Array(memberCount).fill({
      name: '',
      destination: '',
      split: 1, // Default to all members having same share
      percentage: 0
    })

    const memberSlides = Array.from({ length: memberCount }, (_, index) => index + 1)

    slides = [0, ...memberSlides, memberCount + 1]
  }

  // Calculate percentage values for each member
  $: {
    if (members.length) {
      const allSplits = members.map((member) => member.split).reduce((a, b) => a + b)
      const updatedMembers = members.map((member) => {
        return {
          ...member,
          percentage: member.split ? (member.split / allSplits) * 100 : 0
        }
      })

      members = [...updatedMembers]
    }
  }

  $: slideDirection = (
    slides.indexOf(previousSlide) > slides.indexOf(slide) ? 'right' : 'left'
  ) as SlideDirection

  function isMemberIncomplete(member: Member) {
    if (!member.name || !member.destination || !member.split) {
      return true
    }
    return false
  }

  function back() {
    previousSlide = slides[slides.indexOf(slide) - 2]
    slide = slides[slides.indexOf(slide) - 1]
  }

  function next(to = slides[slides.indexOf(slide) + 1]) {
    previousSlide = slide
    slide = to
  }
  // @TODO Use on summary?
  function deleteMember(index: number) {
    members = members.filter((member) => members.indexOf(member) !== index)
  }
</script>

<!-- Name your prism -->
{#if slide === 0}
  <Slide direction={slideDirection}>
    <div class="max-w-sm">
      <h1 class="text-4xl">Name your prism</h1>
      <input
        id="address"
        class="mt-4 w-full p-2 rounded"
        type="text"
        bind:value={label}
        placeholder="bitcoin-dev-fund"
      />
      <!-- Member Count -->
      <h1 class="mt-8 text-4xl">Member count</h1>
      <div class="flex justify-around mt-4">
        <button
          on:click={() => {
            if (memberCount > 2) {
              memberCount--
            }
          }}
        >
          <div class="w-8">
            {@html minus}
          </div></button
        >
        <p class="text-4xl">{memberCount}</p>
        <button
          on:click={() => {
            if (memberCount < 10) {
              memberCount++
            }
          }}
        >
          <div class="w-8">
            {@html plus}
          </div></button
        >
      </div>

      <div class="mt-8">
        <Button disabled={!label} format="secondary" fullWidth={true} on:click={() => next()}
          >Next</Button
        >
      </div>
    </div>
  </Slide>
{/if}
<!-- Choose prism members  -->
{#each members as member, i}
  {#if slide === i + 1}
    <Slide direction={slideDirection}>
      <div class="max-w-sm">
        <h1 class="text-3xl mb-6">Member {i + 1}</h1>
        <div class="flex flex-col border rounded p-6 w-full overflow">
          <!-- Name, Split & Share (%) -->
          <div class="flex flex-between items-center gap-4">
            <!-- Name -->
            <!-- @TODO input validation for name -->
            <div class="w-full">
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
            <div>
              <label class="mb-1 block" for="split">Split</label>
              <input
                id="split"
                class="border w-full p-2 rounded"
                type="number"
                bind:value={member.split}
              />
            </div>
            <!-- Share -->
            <div class="w-20">
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

        <div class="mt-8 flex w-full justify-between">
          <Button format="secondary" on:click={() => back()}>Back</Button>
          <Button
            disabled={isMemberIncomplete(members[i])}
            format="secondary"
            on:click={() => next()}>Next</Button
          >
        </div>
      </div>
    </Slide>
  {/if}
{/each}
<!-- Summary  -->
{#if slide === slides.length - 1}
  <Slide direction={slideDirection}>
    <div class="max-w-sm">
      <p class="mt-4">
        {label} has {members.length} members. Please review before creating your prism:
      </p>
      <div class="mt-6">
        {#each members as member, i}
          <div class="flex p-1">
            <p class="mr-2">{i + 1})</p>
            <p class="mr-2">{member.percentage.toFixed(2)}% -</p>
            <p>{member.name}</p>
          </div>
        {/each}
      </div>
      <div class="mt-8 flex w-full justify-between">
        <Button format="secondary" on:click={() => back()}>Back</Button>
        <Button format="primary" on:click={() => finish({ label, members })}>Finish</Button>
      </div>
    </div>
  </Slide>
{/if}
