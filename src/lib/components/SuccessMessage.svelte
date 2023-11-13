<script lang="ts">
  import { Check, Copy } from 'lucide-svelte';
  import { copy } from 'svelte-copy';

  export let successData: UploadResponseBody;

  let showSuccessCopy = false;

  function showSuccessCopyMessage() {
    showSuccessCopy = true;
    setTimeout(() => {
      showSuccessCopy = false;
    }, 2000);
  }
</script>

<div
  class="flex items-center px-4 py-3 text-green-900 border border-green-100 bg-green-50 rounded-xl"
>
  <div class="flex flex-col flex-1 gap-1">
    <span class="flex items-center gap-3 text-lg">
      <Check size="20" />
      <span>Success</span>
    </span>
    <a href={successData.data.link} target="_blank" class="text-green-800 hover:text-green-700"
      >{successData.data.link}</a
    >
  </div>
  {#if showSuccessCopy}
    <span class="px-2 py-3 font-bold">Copied!</span>
  {:else}
    <button
      title="Copy to clipboard"
      use:copy={{ text: String(successData.data.link), events: 'click' }}
      on:svelte-copy={showSuccessCopyMessage}
      on:svelte-copy:error={(event) => {
        alert(event.detail.message);
      }}
      class="px-5 py-3 hover:text-green-700 active:text-green-900"><Copy /></button
    >
  {/if}
</div>
