<script>
  import '../scripts/registerServiceWorker';
  import { onMount, onDestroy } from 'svelte';
  import axios from 'axios';
  import 'primeflex/primeflex.css';

  let userID;
  let processed = false;
  let uploaded = false;
  let downloaded = false;
  let fileFormat = [];
  let uploadedFiles = [];

  onMount(async () => {
    await axios.get('https://code2doc2022.herokuapp.com/').then((res) => {
      userID = res.data.userId;
    });
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

  const process = async () => {
    console.log('Processing');
    processed = true;
    await axios.get('https://code2doc2022.herokuapp.com/process/', {
      headers: {
        'User-Name': userID,
      },
    });
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
    console.log(uploadedFiles);
    console.log(fileFormat);
  } else {
    uploaded = false;
  }
</script>

<main>
  <header class="px-5 py-4 flex flex-row align-items-center gap-2">
    <img src="./logo.svg" class="h-3rem" alt="logo" />
    <h2>code2doc</h2>
  </header>
  <div class="content px text-center">
    <h1 class="lg:text-8xl md:text-7xl text-6xl sm:mb-5 mb-4 font-bold">
      code2doc
    </h1>
    <p class="sm:mb-6 mb-5 xl:w-3 lg:w-6 tagline">
      A quick solution to get all your code converted to a document.
    </p>
    <div
      id="drop_zone"
      on:drop={dropHandler}
      on:dragover={dragOverHandler}
      on:dragleave={dragOutHandler}
      class="z-1 relative p-5 sm:w-10 w-9 sm:h-25rem sm:max-h-25rem h-15rem max-h-15rem mt-1"
    >
      <div class="overflow-auto h-full flex flex-row flex-wrap">
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
              <p>{file.name}</p>
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
      disabled={!uploaded}>Convert</button
    >
  </div>
  <footer class="px">
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

  h1 {
    font-size: 4rem;
    color: var(--green);
  }
</style>
