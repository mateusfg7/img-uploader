<script lang="ts">
  import { PUBLIC_IMGUR_CLIENT_ID } from '$env/static/public';
  import { ImagePlus, Image, X, Loader2, Check, Copy, AlertTriangle } from 'lucide-svelte';
  import { copy } from 'svelte-copy';

  import { formatBytes } from '$lib';

  import Title from '$lib/components/Title.svelte';

  let imageString: string | undefined;
  let fileInput: HTMLInputElement;
  let file: File | undefined;
  let isUploading = false;
  let successData: undefined | UploadResponseBody;
  let errorData: undefined | UploadErrorBody;
  let showSuccessCopy = false;

  const acceptedFormats = ['.jpeg', '.jpg', '.png', '.gif', '.webp'];

  function showSuccessCopyMessage() {
    showSuccessCopy = true;
    setTimeout(() => {
      showSuccessCopy = false;
    }, 2000);
  }

  function onFileSelected(
    e: Event & {
      currentTarget: EventTarget & HTMLInputElement;
    }
  ) {
    if (!e.currentTarget.files) return;
    if (isUploading) return;

    successData = undefined;
    errorData = undefined;

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

  async function handleUpload() {
    if (!file) {
      console.log('ERR | No file selected');
      return;
    }
    if (file.size > 1e7) {
      console.log('ERR | Bigger file');
      return;
    }
    if (isUploading) {
      console.log('ERR | Already uploading');
      return;
    }

    isUploading = true;

    const body = new FormData();
    body.append('image', file);

    const headers = new Headers();
    headers.append('Authorization', `Client-ID ${PUBLIC_IMGUR_CLIENT_ID}`);

    const requestOptions: RequestInit = {
      method: 'POST',
      redirect: 'follow',
      body,
      headers
    };

    const response: UploadResponseBody | UploadErrorBody = await fetch(
      'https://api.imgur.com/3/image',
      requestOptions
    )
      .then((response) => response.json())
      .catch((error) => console.log('error', error));

    if (response.success) {
      successData = response as UploadResponseBody;
    } else {
      errorData = response as UploadErrorBody;
    }

    isUploading = false;
    file = undefined;
    imageString = undefined;
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
        <img src={imageString} alt="Uploaded" class="object-cover max-w-md max-h-64 rounded-xl" />
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
  {#if successData}
  <div
    class="flex items-center px-4 py-3 text-green-900 border border-green-100 bg-green-50 rounded-xl"
  >
    <div class="flex flex-col flex-1 gap-1">
      <span class="flex items-center gap-3 text-lg">
        <Check size="20" />
        <span>Success</span>
      </span>
      <a href={successData?.data.link} target="_blank" class="text-green-800 hover:text-green-700"
        >{successData?.data.link}</a
      >
    </div>
    {#if showSuccessCopy}
      <span class="px-2 py-3 font-bold">Copied!</span>
    {:else}
      <button
        title="Copy to clipboard"
        use:copy={{ text: String(successData?.data.link), events: 'click' }}
        on:svelte-copy={showSuccessCopyMessage}
        on:svelte-copy:error={(event) => {
          alert(event.detail.message);
        }}
        class="px-5 py-3 hover:text-green-700 active:text-green-900"><Copy /></button
      >
    {/if}
  </div>
  {/if}
  {#if errorData}
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
  {/if}
</div>

<input
  type="file"
  accept={acceptedFormats.join(',')}
  on:change={(e) => onFileSelected(e)}
  bind:this={fileInput}
  class="hidden"
/>

