<script>
  import '../scripts/registerServiceWorker';
  import { onMount, onDestroy } from 'svelte';
  import axios from 'axios';
  import 'primeflex/primeflex.css';
  import { SvelteToast } from '@zerodevx/svelte-toast';
  import { toast } from '@zerodevx/svelte-toast';

  const options = {
    duration: 4000,
    initial: 0,
    next: 0,
    pausable: true,
  };

  let userID;
  let processed = false;
  let uploaded = false;
  let fileFormat = [];
  let uploadedFiles = [];
  let convertingLoader = false;
  let uniqueFileFormats = [];
  let finalFileFormats = [];
  let fileTypeSelection = false;
  let downloadingLoader = false;

  const checkFile = (file) => {
    if (file.name.includes('.')) {
      if (
        !file.type.includes('application/') &&
        !file.type.includes('image/') &&
        !file.type.includes('video/') &&
        !file.type.includes('audio/') &&
        !file.type.includes('font/') &&
        !file.name.includes('.dSYM')
      ) {
        if (file.size <= 5000000) {
          uploadedFiles.push(file);
          fileFormat.push(file.name.split('.').pop());
          uploadedFiles = uploadedFiles;
        } else {
          toast.push(
            `${file.name}: File needs to be under 5MB (was ${
              file.size / 1000000
            }MB)`
          );
        }
      } else {
        toast.push(`${file.name}: Only code/text files can be uploaded.`);
      }
    } else {
      toast.push(`${file.name}: File extension is needed.`);
    }
  };

  onMount(async () => {
    await axios
      .get('https://code2doc2022.herokuapp.com/')
      .then((res) => {
        userID = res.data.userId;
      })
      .catch((e) => {
        toast.push('Could not create user sessions.');
      });
    function handleFiles(e) {
      const fileList = this.files;
      for (let i = 0; i < fileList.length; i++) {
        checkFile(fileList[i]);
      }
      e.target.value = '';
    }
    const uploadButtons = document.querySelectorAll('.upload');
    uploadButtons.forEach((button) => {
      button.addEventListener('change', handleFiles, false);
    });
  });

  const deleteFolder = async () => {
    await axios.delete(
      'https://code2doc2022.herokuapp.com/upload/clearFolder',
      {
        headers: {
          'User-Name': userID,
        },
      }
    );
  };

  window.addEventListener('beforeunload', (e) => {
    deleteFolder();
  });

  const checkTypes = () => {
    uniqueFileFormats = [...new Set(fileFormat)];
    if (uniqueFileFormats.length === 1) {
      finalFileFormats = uniqueFileFormats;
      convert();
    } else {
      // TODO
      fileTypeSelection = true;
    }
  };

  const convert = async () => {
    fileTypeSelection = false;
    convertingLoader = true;
    let formatString = '';
    formatString = finalFileFormats.join();
    let fileData = new FormData();
    uploadedFiles.forEach((file) => {
      fileData.append('files', file);
    });
    try {
      const uploadRes = await axios.post(
        'https://code2doc2022.herokuapp.com/upload/uploadFiles',
        fileData,
        {
          headers: {
            'User-Name': userID,
            'File-Format': formatString,
          },
        }
      );
      uploaded = true;
      const procRes = await axios.get(
        'https://code2doc2022.herokuapp.com/process/',
        {
          headers: {
            'User-Name': userID,
          },
        }
      );
      processed = true;
      convertingLoader = false;
    } catch (error) {
      convertingLoader = false;
      toast.push(error);
    }
  };

  const download = async () => {
    downloadingLoader = true;
    await axios
      .get('https://code2doc2022.herokuapp.com/download/', {
        headers: {
          'User-Name': userID,
        },
        responseType: 'blob',
      })
      .then((res) => {
        downloadingLoader = false;
        window.open(URL.createObjectURL(res.data));
      })
      .catch((e) => {
        downloadingLoader = false;
        toast.push(e);
      });
  };

  function dropHandler(ev) {
    ev.preventDefault();
    var dropzone = ev.currentTarget;
    dropzone.classList.remove('hovered');
    if (ev.dataTransfer.items) {
      for (var i = 0; i < ev.dataTransfer.items.length; i++) {
        if (ev.dataTransfer.items[i].kind === 'file') {
          var file = ev.dataTransfer.items[i].getAsFile();
          checkFile(file);
        }
      }
    } else {
      for (var i = 0; i < ev.dataTransfer.files.length; i++) {
        checkFile(ev.dataTransfer.files[i]);
      }
    }
  }

  function dragOverHandler(ev) {
    ev.preventDefault();
    var dropzone = ev.currentTarget;
    dropzone.classList.add('hovered');
  }

  function dragOutHandler(ev) {
    ev.preventDefault();
    var dropzone = ev.currentTarget;
    dropzone.classList.remove('hovered');
  }

  const deleteUploaded = (id) => {
    uploadedFiles.splice(id, 1);
    uploadedFiles = uploadedFiles;
    fileFormat.splice(id, 1);
    fileFormat = fileFormat;
  };

  $: if (uploadedFiles.length) {
    uploaded = true;
  } else {
    uploaded = false;
  }

  const selectAllFormats = () => {
    if (document.getElementsByName('fileFormatsAll')[0].checked) {
      finalFileFormats = uniqueFileFormats;
    } else {
      finalFileFormats = [];
    }
  };

  const toggleSelectAllFormats = () => {
    if (finalFileFormats === uniqueFileFormats) {
      document.getElementsByName('fileFormatsAll')[0].checked = true;
    } else {
      document.getElementsByName('fileFormatsAll')[0].checked = false;
    }
  };
</script>

<main class="h-full">
  <div class="wrap flex flex-column">
    <SvelteToast {options} />
  </div>
  {#if convertingLoader}
    <div class="loading-blackout flex z-3 h-full w-full fixed">
      <div class="max-w-10 loading-div m-auto p-6">
        {#if !uploaded}
          <p>Uploading files...</p>
        {:else if !processed}
          <p>Processing files...</p>
        {/if}
      </div>
    </div>
  {/if}
  {#if downloadingLoader}
    <div class="loading-blackout flex z-3 h-full w-full fixed">
      <div class="max-w-10 loading-div m-auto p-6">
        <p>Downloading...</p>
      </div>
    </div>
  {/if}
  {#if fileTypeSelection}
    <div
      class="loading-blackout flex z-4 h-full w-full fixed"
      on:click={() => (fileTypeSelection = false)}
    >
      <div
        class="max-w-10 loading-div m-auto sm:mx-auto mx-5 p-6"
        on:click={(e) => e.stopPropagation()}
      >
        <p class="mb-4 text-center">Which filetypes do you want to convert?</p>
        {#each uniqueFileFormats as fileFormat}
          <label class="mt-1">
            <input
              class="chk"
              type="checkbox"
              bind:group={finalFileFormats}
              value={fileFormat}
              name="fileFormats"
              on:change={toggleSelectAllFormats}
            />
            {fileFormat}
          </label>
        {/each}
        <label class="mt-4">
          <input
            class="chk"
            type="checkbox"
            name="fileFormatsAll"
            on:change={selectAllFormats}
          />
          Select All
        </label>
        <button
          class="sm:mt-7 mt-3 button button-inverse mx-auto {!finalFileFormats.length
            ? 'button-disabled'
            : null}"
          disabled={!finalFileFormats.length}
          on:click={convert}>Convert</button
        >
      </div>
    </div>
  {/if}
  <header class="px-5 py-4 flex flex-row align-items-center gap-2">
    <img src="./logo.svg" class="w-min h-3rem" alt="logo" />
    <h2>code2doc</h2>
  </header>
  {#if !processed}
    <div class="flex h-full flex-column content px text-center">
      <h1 class="title lg:text-8xl md:text-7xl text-6xl sm:mb-3 mb-4 font-bold">
        code2doc
      </h1>
      <p class="sm:mb-6 mb-5 xl:w-3 lg:w-6 tagline">
        A quick solution to get all your code converted to a document.
      </p>
      <div
        style="min-height: 5rem;"
        id="drop-zone"
        on:drop={dropHandler}
        on:dragover={dragOverHandler}
        on:dragleave={dragOutHandler}
        class="z-1 relative p-5 sm:w-10 w-9 flex-grow-1 mt-1"
      >
        <div
          style="transform: translate(-50%,-50%); --webkit-transform: translate(-50%,-50%"
          class="absolute left-50 top-50 overflow-auto h-full flex flex-row flex-wrap row-gap-4 w-full"
        >
          {#if !uploaded}
            <div class="m-auto">
              <label class="inline-block button" for="uploadmain"
                >Choose files</label
              >
              <input type="file" id="uploadmain" class="upload" multiple />
              <p class="mt-2">or drop your files here</p>
            </div>
          {:else}
            {#each uploadedFiles as file, i}
              <div class="xl:w-2 lg:w-3 md:w-4 sm:w-6 w-12 relative pt-4">
                <div style="top: 10px; left: 10px" class="absolute">
                  <span
                    class="inline-block button button-circle-sm button-circle text-center flex align-items-center justify-content-center"
                    on:click={() => deleteUploaded(i)}
                  >
                    <p>x</p>
                  </span>
                </div>
                <p>{i + 1}</p>
                <p class="px-2">{fileFormat[i]}</p>
                <div class="file-name px-2 mt-2">
                  {#if file.name.length > 16}
                    {file.name.slice(0, 12)}{file.name.length > 12
                      ? '...'
                      : null}
                  {:else}
                    {file.name}
                  {/if}
                </div>
              </div>
            {/each}
          {/if}
        </div>
        <div
          style="top: -25px; right: -25px"
          class="z-2 absolute {!uploaded ? 'hidden' : null}"
        >
          <label
            class="inline-block button button-circle text-center flex"
            for="upload"
          >
            <img class="m-auto" src="./plus.svg" alt="plus" /></label
          >
          <input type="file" id="upload" class="upload" multiple />
        </div>
      </div>
      <button
        class="sm:mt-7 mt-6 button button-inverse {!uploaded
          ? 'button-disabled'
          : null}"
        disabled={!uploaded}
        on:click={checkTypes}>Convert</button
      >
    </div>
  {:else}
    <div class="content px text-center">
      <h2
        class="downsplash lg:text-7xl md:text-6xl text-5xl sm:mb-3 mb-4 font-bold"
      >
        code has been converted
      </h2>
      <div
        class="mt-5 flex flex-row gap-5 flex-wrap align-items-center justify-content-center"
      >
        <button on:click={download} class="button">Download</button>
        <button
          on:click={() => {
            deleteFolder();
            window.location.reload();
          }}
          class="button button-inverse">Convert Again</button
        >
      </div>
    </div>
  {/if}
  <!-- TODO: refactor these two if and else components into separate files -->
  <footer class="px sm:mt-4 mt-0">
    <p class="text-center">
      Made with <span style="color:#FF6B6B">&#9829;</span> by GDSC
    </p>
  </footer>
</main>

<style>
  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }

  .loading-blackout {
    background-color: rgba(14, 20, 25, 0.6);
  }

  .loading-div {
    background-color: #161e25;
    color: var(--blue);
    display: flex;
    flex-direction: column;
  }

  header > h2 {
    font-family: 'SometypeMono Bold';
  }

  .px {
    padding-left: 2rem;
    padding-right: 2rem;
  }

  .text-center {
    text-align: center;
  }

  .content {
    margin-top: auto;
    margin-bottom: auto;
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .tagline {
    width: 90%;
    max-width: 650px;
    color: var(--blue);
  }

  h1,
  .downsplash {
    color: var(--green);
  }

  @keyframes blink-animation {
    50% {
      opacity: 0;
    }
  }
  @-webkit-keyframes blink-animation {
    50% {
      opacity: 0;
    }
  }

  h1:after,
  .downsplash:after {
    content: '';
    width: 3px;
    margin-left: 0.3rem;
    height: 6.8rem;
    position: absolute;
    background-color: var(--green);
    animation: blink-animation 1s step-start infinite;
    -webkit-animation: blink-animation 1s step-start infinite;
  }

  .downsplash:after {
    height: 4.7rem;
    width: 2.5px;
  }

  .file-name {
    position: relative;
  }

  /* .file-name .file-name-tooltip {
    visibility: hidden;
    width: max-content;
    background-color: var(--primary);
    color: var(--bg-color);
    text-align: center;
    padding: 5px 0;
    border-radius: 6px;
    position: absolute;
    bottom: 0;
  }

  .file-name:hover .file-name-tooltip {
    visibility: visible;
  } */

  @media (max-width: 991px) {
    h1:after {
      height: 4.5rem;
      margin-left: 0.2rem;
      width: 2.5px;
    }

    .downsplash:after {
      height: 3.5rem;
      width: 2px;
      margin-left: 0.12rem;
    }
  }

  @media (max-width: 767px) {
    h1:after {
      height: 3.3rem;
      margin-left: 0.2rem;
      width: 2px;
    }

    .downsplash:after {
      height: 2.8rem;
      width: 1.6px;
      margin-left: 0.1rem;
    }
  }
</style>
