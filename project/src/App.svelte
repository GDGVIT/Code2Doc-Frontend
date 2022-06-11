<script>
  import '../scripts/registerServiceWorker';
  import { onMount, onDestroy } from 'svelte';
  import axios from 'axios';
  import 'primeflex/primeflex.css';

  let userID;
  let processed = false;
  let downloaded = false;
  let uploaded = false;
  let fileFormat = [];
  let uploadedFiles = [];
  let convertingLoader = false;

  onMount(async () => {
    await axios
      .get('https://code2doc2022.herokuapp.com/')
      .then((res) => {
        userID = res.data.userId;
      })
      .catch((e) => console.log(e));
    function handleFiles() {
      const fileList = this.files;
      for (let i = 0; i < fileList.length; i++) {
        uploadedFiles.push(fileList[i]);
        fileFormat.push(fileList[i].name.split('.').pop());
        uploadedFiles = uploadedFiles;
      }
    }
    const uploadButtons = document.querySelectorAll('.upload');
    uploadButtons.forEach((button) => {
      button.addEventListener('change', handleFiles, false);
    });
  });

  onDestroy(async () => {
    axios.delete('https://code2doc2022.herokuapp.com/upload/clearFolder', {
      headers: {
        'User-Name': userID,
      },
    });
  });

  const convert = async () => {
    convertingLoader = true;
    let formatString = '';
    formatString = fileFormat.join();
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
      console.log(uploadRes);
      uploaded = true;
      const procRes = await axios.get(
        'https://code2doc2022.herokuapp.com/process/',
        {
          headers: {
            'User-Name': userID,
          },
        }
      );
      console.log(procRes);
      processed = true;
      convertingLoader = false;
    } catch (error) {
      console.log(error);
    }
  };

  const download = async () => {
    console.log('Downloading pdf.');
    await axios
      .get('https://code2doc2022.herokuapp.com/download/', {
        headers: {
          'User-Name': userID,
        },
        responseType: 'blob',
      })
      .then((res) => {
        window.open(URL.createObjectURL(res.data));
        downloaded = true;
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
          uploadedFiles.push(file);
          fileFormat.push(file.name.split('.').pop());
          uploadedFiles = uploadedFiles;
        }
      }
    } else {
      for (var i = 0; i < ev.dataTransfer.files.length; i++) {
        uploadedFiles.push(ev.dataTransfer.files[i]);
        fileFormat.push(ev.dataTransfer.files[i].name.split('.').pop());
        uploadedFiles = uploadedFiles;
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

  $: if (uploadedFiles.length) {
    uploaded = true;
  } else {
    uploaded = false;
  }
</script>

<main>
  <header class="px-5 py-4 flex flex-row align-items-center gap-2">
    <img src="./logo.svg" class="w-min h-3rem" alt="logo" />
    <h2>code2doc</h2>
  </header>
  {#if !processed}
    <div class="content px text-center">
      <h1 class="lg:text-8xl md:text-7xl text-6xl sm:mb-3 mb-4 font-bold">
        code2doc
      </h1>
      <p class="sm:mb-6 mb-5 xl:w-3 lg:w-6 tagline">
        A quick solution to get all your code converted to a document.
      </p>
      <div
        id="drop-zone"
        on:drop={dropHandler}
        on:dragover={dragOverHandler}
        on:dragleave={dragOutHandler}
        class="z-1 relative p-5 sm:w-10 w-9 sm:h-23rem sm:max-h-23rem h-15rem max-h-15rem mt-1"
      >
        <div class="overflow-auto h-full flex flex-row flex-wrap row-gap-4">
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
              <div class="xl:w-2 lg:w-3 md:w-4 sm:w-6 w-12">
                <p class="px-2">{fileFormat[i]}</p>
                <div class="file-name px-2 mt-2">
                  {#if file.name.length > 16}
                    {file.name.slice(0, 12)}{file.name.length > 12
                      ? '...'
                      : null}
                    <!-- <span class="z-2 file-name-tooltip">{file.name}</span> -->
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
        on:click={convert}>Convert</button
      >
    </div>
  {:else}
    <div class="content px text-center">
      <h2
        class="downsplash lg:text-7xl md:text-6xl text-5xl sm:mb-3 mb-4 font-bold"
      >
        code has been converted
      </h2>
      <p class="sm:mb-6 mb-5 xl:w-3 lg:w-6 tagline">
        Woot woot lorem ipsum dolor sit amet!
      </p>
      <div
        class="mt-5 flex flex-row gap-5 flex-wrap align-items-center justify-content-center"
      >
        <button on:click={download} class="button">Download</button>
        <button
          on:click={() => window.location.reload()}
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
