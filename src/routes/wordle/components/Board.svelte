<script lang="ts">
  import { createEventDispatcher } from 'svelte';
  import { get } from 'svelte/store';
  import { browser } from '$app/environment';
  import type { LetterState, StopState } from './types';
  import { flip } from './flip';
  import {
    addChar,
    clearChars,
    inputState,
    letterState,
    removeChar,
    setState,
  } from '../+page.svelte';

  export let word: string;
  export let totalAttempts = word.length + 1;

  const dispatcher = createEventDispatcher<{'stop': StopState}>();

  let currentAttempt = 0;
  let guesses: string[][] = Array(totalAttempts).fill([]);

  $: currentAttempt >= totalAttempts && console.log('you los');

  function checkWord(guess: string) {
    const array: LetterState[] = Array(guess.length);

    if (guess === word) {
      console.log('gg');
      return array.fill('correct');
    }

    let wordCheck = word;
    for (let i = 0; i < guess.length; i++) {
      const letter = guess[i]!;
      let result: LetterState = 'present';

      if (!wordCheck.includes(letter)) result = 'absent';
      if (wordCheck.indexOf(letter) === i) result = 'correct';
      wordCheck = wordCheck.replace(letter, '#');

      array[i] = result;
      if (get(letterState)[letter] !== 'correct') setState(letter, result);
    }

    return array;
  }

  const keyboardListener = ({ key, ctrlKey }: KeyboardEvent) => {
    if (/^[a-zA-Z]$/.test(key) && $inputState.length < word.length) return addChar(key);

    if (key === 'Enter') {
      if ($inputState.length !== word.length) return;
      guesses[currentAttempt++] = $inputState;
      clearChars();
    }

    if (key === 'Backspace') {
      if (ctrlKey) clearChars();
      return removeChar()
    }
  }

  if (browser) document.addEventListener('keydown', keyboardListener);

  function stop(state: StopState) {
    document.removeEventListener('keydown', keyboardListener);
    dispatcher('stop', state);
  }

  type LetterParams = {
    letter: string,
    type: LetterState | 'active' | 'inactive',
    position: number,
  };
</script>

{#snippet letter({ letter, type, position }: LetterParams)}
  <button
    class="letter {type}"
    in:flip={{ duration: 300, i: position }}
    on:click={() => {
      if (!letter || $inputState.length >= word.length) return;

      addChar(letter);
    }}
  >
    {letter}
  </button>
{/snippet}

<div class="board" style="--attempts: {totalAttempts}; --letters: {word.length};">
  {#each guesses as guess, line}
    <div class="attempt">
      {#if currentAttempt > line}
        {@const result = checkWord(guess.join(''))}
        {#each guess as character, i}
          {@render letter({ letter: character, type: result[i]!, position: i })}
        {/each}
      {:else if currentAttempt < line}
        {#each { length: word.length } as _, i}
          {@render letter({ letter: '', type: 'inactive', position: i })}
        {/each}
      {:else}
        {#each { length: word.length } as _, i}
          {@render letter({ letter: $inputState[i] ?? '', type: 'active', position: i })}
        {/each}
      {/if}
    </div>
  {/each}
</div>

<style lang="scss">
  .board {
    --_letter-size: 4rem;
    --_gap: 0.25rem;

    display: grid;
    grid-template-rows: repeat(var(--attempts), var(--_letter-size));
    gap: var(--_gap);
  }

  .attempt {
    display: grid;
    grid-template-columns: repeat(var(--letters), var(--_letter-size));
    gap: inherit;
  }

  .letter {
    display: grid;
    place-items: center;
    height: var(--_letter-size);
    aspect-ratio: 1;
    border-radius: 0.125em;
    border-style: solid;
    border-width: 0.125em;

    font-size: calc(var(--_letter-size) - 1.5em);
    font-weight: bold;
    text-transform: uppercase;
    line-height: 0;

    color: var(--_color-fg, hsl(0, 0%, 100%));
    background-color: var(--_color-bg, transparent);
    border-color: var(--_color-border, var(--_color-bg));

    &.active {
      --_color-border: hsl(0, 0%, 62%);
      --_color-fg: var(--_color-border);

      :global(.dark) & {
        --_color-fg: hsl(0, 0%, 80%);
      }
    }
    &.inactive {
      --_color-border: var(--color-inactive);
    }

    &.correct {
      --_color-bg: var(--color-correct);
    }
    &.present {
      --_color-bg: var(--color-present);
    }
    &.absent {
      --_color-bg: var(--color-absent);
    }
  }
</style>