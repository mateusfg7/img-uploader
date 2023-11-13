<script lang="ts">
  import { AlertTriangle } from 'lucide-svelte';

  export let errorData: UploadErrorBody;
</script>

<div
  class="flex gap-6 items-center px-4 py-3 text-red-900 bg-red-50 rounded-xl border border-red-100"
>
  <AlertTriangle size="30" fill="rgba(127, 29, 29, 0.1)" />
  <div>
    <span class="flex gap-3 items-center text-lg">
      <span><span class="font-bold">Error</span> {errorData.status}</span>
    </span>
    <span class="text-red-800">{errorData.data.error}</span>
  </div>
</div>
