<script lang="ts">
  import { PUBLIC_IMGUR_CLIENT_ID } from '$env/static/public';
  import { ImagePlus, Image, X, Loader2 } from 'lucide-svelte';

  import { formatBytes } from '$lib';

  import Title from '$lib/components/Title.svelte';
  import SuccessMessage from '$lib/components/SuccessMessage.svelte';
  import ErrorMessage from '$lib/components/ErrorMessage.svelte';

  let imageString: string | undefined;
  let fileInput: HTMLInputElement;
  let file: File | undefined;

  let successData: undefined | UploadResponseBody;
  let errorData: undefined | UploadErrorBody;

  let isUploading = false;
  let isDragging = false;

  const acceptedFormats = ['.jpeg', '.jpg', '.png', '.gif', '.webp'];

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

  function onDropFile(
    e: DragEvent & {
      currentTarget: EventTarget & HTMLButtonElement;
    }
  ) {
    if (isUploading) return;
    if (!e.dataTransfer) return;
    if (e.dataTransfer.files.length == 0) return;

    successData = undefined;
    errorData = undefined;
    isDragging = false;

    file = e.dataTransfer.files[0];
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

<svelte:head>
  <title>Imgur Image Uploader</title>
  <meta
    name="description"
    content="Upload your image and directly receive a link for it from Imgur API."
  />
</svelte:head>

<div class="m-auto space-y-10 max-w-lg">
  <Title />

  <div class="flex flex-col gap-6 justify-center items-center w-full">
    {#if imageString}
      <div
        class="flex flex-col gap-10 items-center p-6 w-full h-full rounded-xl border border-dashed border-neutral-500 bg-neutral-100/50"
      >
        <div class="flex flex-col gap-3 items-center w-full">
          <p class="text-2xl">Ready to upload</p>
          <img src={imageString} alt="Uploaded" class="object-cover max-w-md max-h-64 rounded-xl" />
          <div class="flex gap-3 items-center">
            <span class="flex gap-1 items-center">
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
        <div class="flex justify-center w-full">
          {#if isUploading}
            <button
              disabled
              class="flex gap-2 justify-center items-center p-3 w-1/2 text-lg text-white bg-blue-800 rounded-xl opacity-90 transition-colors cursor-wait"
            >
              <Loader2 class="animate-spin" />
              Uploading
            </button>
          {:else}
            <button
              on:click={handleUpload}
              class="flex justify-center items-center p-3 w-1/2 text-lg text-white rounded-xl transition-colors bg-blue-800/90 hover:bg-blue-800"
            >
              Upload
            </button>
          {/if}
        </div>
      </div>
    {:else}
      <button
        class="flex flex-col items-center justify-center px-6 py-12 h-full w-full gap-2 border border-dashed rounded-xl border-neutral-500 bg-neutral-100/50 cursor-pointer data-[is-dragging='true']:bg-blue-100/50 data-[is-dragging='true']:border-blue-800 transition-colors"
        data-is-dragging={isDragging}
        on:dragenter|preventDefault={() => (isDragging = true)}
        on:dragleave|preventDefault={() => (isDragging = false)}
        on:dragover|preventDefault={() => (isDragging = true)}
        on:drop|preventDefault={onDropFile}
        on:click={() => {
          fileInput.click();
        }}
      >
        <ImagePlus size="35" strokeWidth="1.5" />
        <div class="text-center">
          <p>
            <span
              class="font-bold text-blue-800 transition-colors cursor-pointer hover:text-blue-600"
              >Upload a file</span
            > or drag and drop
          </p>
          <p class="text-xs text-neutral-600">JPG, PNG, WEBP or GIF up to 10MB</p>
        </div>
      </button>
    {/if}
  </div>

  {#if successData}
    <SuccessMessage {successData} />
  {/if}
  {#if errorData}
    <ErrorMessage {errorData} />
  {/if}
</div>

<input
  type="file"
  accept={acceptedFormats.join(',')}
  on:change={(e) => onFileSelected(e)}
  bind:this={fileInput}
  class="hidden"
/>
