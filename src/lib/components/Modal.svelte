<script lang="ts">
  import { onMount } from 'svelte';
  import { browser } from '$app/environment';

  export let content: ConstructorOfATypedSvelteComponent | string;
  export let open = true;
  export let wrapper: HTMLElement | undefined = browser ? document.body : undefined;

  let dialog: HTMLDialogElement;

  $: open ? dialog?.showModal?.() : dialog?.close?.();

  onMount(() => {
    if (open) dialog.showModal();

    if (!wrapper) return;

    dialog.remove();
    wrapper.appendChild(dialog);
  });
</script>

<dialog bind:this={dialog}>
  <button
    class="close"
    on:click={() => open = false}
    on:submit={() => open = false}
  >
    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 384 512"><path d="M378.4 71.4c8.5-10.1 7.2-25.3-2.9-33.8s-25.3-7.2-33.8 2.9L192 218.7 42.4 40.6C33.9 30.4 18.7 29.1 8.6 37.6S-2.9 61.3 5.6 71.4L160.7 256 5.6 440.6c-8.5 10.2-7.2 25.3 2.9 33.8s25.3 7.2 33.8-2.9L192 293.3 341.6 471.4c8.5 10.1 23.7 11.5 33.8 2.9s11.5-23.7 2.9-33.8L223.3 256l155-184.6z"/></svg>
  </button>
  {#if typeof content === 'string'}
    {content}
  {:else}
    <svelte:component this={content} />
  {/if}
</dialog>

<style>
  .close {
    all: unset;
  }
</style>
