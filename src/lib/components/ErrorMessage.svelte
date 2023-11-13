<script lang="ts">
  import { AlertTriangle } from 'lucide-svelte';

  export let errorData: UploadErrorBody;
</script>

<div
  class="flex items-center gap-6 px-4 py-3 text-red-900 border border-red-100 bg-red-50 rounded-xl"
>
  <AlertTriangle size="30" fill="rgba(127, 29, 29, 0.1)" />
  <div>
    <span class="flex items-center gap-3 text-lg">
      <span><span class="font-bold">Error</span> {errorData.status}</span>
    </span>
    <span class="text-red-800">{errorData.data.error}</span>
  </div>
</div>
