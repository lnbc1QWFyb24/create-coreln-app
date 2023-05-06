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

  const slides = ['0', '1', '2', '3'] as const
  let slide: SlideStep = '0'
  let previousSlide: SlideStep = '0'
  let incompleteMember = true
  let label: string = 'bitcoin-devs' // prism name
  let memberCount = 2 // minimum of 2 members in a prism
  let members: Member[] = []

  // Update members array based on member count input on first slide
  // @TODO update the slides var
  $: {
    members = new Array(memberCount).fill({
      name: 'Alice',
      destination: 'dsafasdf',
      split: 1,
      percentage: 0
    })
  }

  // Calculate percentage for each member
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

  // Prevent moving to next slide when member details incomplete
  $: {
    incompleteMember = members.some(
      (member) => !member.name || !member.destination || !member.split
    )
  }

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
  // @TODO do we need this?
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
  // @TODO Use on summary?
  function deleteMember(index: number) {
    members = members.filter((member) => members.indexOf(member) !== index)
  }
</script>

<!-- Name your prism -->
{#if slide === '0'}
  <Slide direction={slideDirection}>
    <div>
      <h1 class="text-4xl">Name your prism</h1>
      <div class="mt-4">
        <input
          id="address"
          class="mt-2 w-full p-2 rounded"
          type="text"
          bind:value={label}
          placeholder="bitcoin-dev-fund"
        />
      </div>
      <!-- Member Count -->
      <h1 class="mt-8 text-4xl">Member count</h1>
      <div class="flex justify-around mt-8">
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
    </div>
    <div class="mt-8">
      <Button disabled={!label} format="secondary" fullWidth={true} on:click={() => next()}
        >Next</Button
      >
    </div>
  </Slide>
{/if}
<!-- Choose prism members  -->
{#each members as member, i}
  {#if slide === (i + 1).toString()}
    <Slide direction={slideDirection}>
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
        <Button disabled={incompleteMember} format="secondary" on:click={() => next()}>Next</Button>
      </div>
    </Slide>
  {/if}
{/each}
<!-- Summary  -->
{#if slide === (members.length + 1).toString()}
  <Slide direction={slideDirection}>
    <div>
      <h1 class="text-4xl">Summary</h1>
      <p>{label} has {members.length} members. Please review before submission:</p>
    </div>
    <div class="flex w-full items-end justify-end">
      <Button format="secondary" on:click={() => back()}>Back</Button>
      <Button format="primary" on:click={() => finish({ label, members })}>Finish</Button>
    </div>
  </Slide>
{/if}
