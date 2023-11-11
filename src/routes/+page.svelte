<script lang="ts">
  import { ImagePlus, Image, X, Loader2 } from 'lucide-svelte';
  import { formatBytes } from '$lib';

  import Title from '$lib/components/Title.svelte';

  let imageString: string | undefined = 'https://github.com/mateusfg7.png';
  // let imageString: string | undefined;
  let fileInput: HTMLInputElement;
  let file: File | undefined;
  let isUploading = false;

  const acceptedFormats = ['.jpeg', '.jpg', '.png', '.gif', '.webp'];

  function onFileSelected(
    e: Event & {
      currentTarget: EventTarget & HTMLInputElement;
    }
  ) {
    if (!e.currentTarget.files) return;

    file = e.currentTarget.files[0];
    let reader = new FileReader();
    reader.readAsDataURL(file);
    reader.onload = (e) => {
      imageString = e.target?.result as string;
    };
  }

  const removeImage = () => {
    imageString = undefined;
    file = undefined;
  };

  function handleUpload() {
    isUploading = true;
  }
</script>

<div class="max-w-lg m-auto space-y-10">
  <Title />
  <div
    class="flex flex-col items-center justify-center w-full gap-6 p-6 border border-dashed rounded-xl border-neutral-500 bg-neutral-100/50"
  >
    {#if imageString}
      <div class="flex flex-col items-center gap-3">
        <p class="text-2xl">Ready to upload</p>
        <img src={imageString} alt="Uploaded" class="max-w-md object-cover max-h-64 rounded-xl" />
        <div class="flex items-center gap-3">
          <span class="flex items-center gap-1">
            <Image size="20" />
            <span>
              {file?.name} ({formatBytes(Number(file?.size))})
            </span>
          </span>
          <button on:click={removeImage} class="transition-colors hover:text-red-500"
            ><X size="20" /></button
          >
        </div>
      </div>
      {#if isUploading}
        <button
          disabled
          class="flex items-center justify-center w-1/2 gap-2 p-3 text-lg text-white transition-colors bg-blue-800 cursor-wait rounded-xl opacity-90"
        >
          <Loader2 class="animate-spin" />
          Uploading
        </button>
      {:else}
        <button
          on:click={handleUpload}
          class="flex items-center justify-center w-1/2 p-3 text-lg text-white transition-colors bg-blue-800/90 rounded-xl hover:bg-blue-800"
        >
          Upload
        </button>
      {/if}
    {:else}
      <div class="flex flex-col items-center justify-center gap-2">
        <ImagePlus size="35" strokeWidth="1.5" />
        <div class="text-center">
          <p>
            <button
              class="font-bold text-blue-800 transition-colors cursor-pointer hover:text-blue-600"
              on:click={() => {
                fileInput.click();
              }}>Upload a file</button
            > or drag and drop
          </p>
          <p class="text-xs text-neutral-600">JPG, PNG, WEBP or GIF up to 10MB</p>
        </div>
      </div>
    {/if}
  </div>
</div>

<input
  type="file"
  accept={acceptedFormats.join(',')}
  on:change={(e) => onFileSelected(e)}
  bind:this={fileInput}
  class="hidden"
/>
