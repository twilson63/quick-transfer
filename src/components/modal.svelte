<script>
  import { createEventDispatcher } from "svelte";

  export let id = window.crypto.randomUUID();
  export let open = false;
  export let cancel = true;
  export let bgColor = "";
  export let border = "";

  const dispatch = createEventDispatcher();
</script>

<input type="checkbox" {id} bind:checked={open} class="modal-toggle" />
<div class="modal">
  <div class="modal-box  {bgColor} {border}">
    {#if cancel}
      <button
        on:click={() => (open = false)}
        class="btn btn-sm btn-circle absolute right-2 top-2">✕</button
      >
    {/if}
    <slot />
  </div>
</div>
