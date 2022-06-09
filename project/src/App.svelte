<script>
  import '../scripts/registerServiceWorker';
  import { onMount, onDestroy } from 'svelte';
  import axios from 'axios';
  import Dropzone from 'dropzone';
  import 'primeflex/primeflex.css';

  let userID;
  let processed = false;
  let uploaded = false;
  let downloaded = false;
  let fileFormat = '';
  let uploadedFiles = [];

  onMount(async () => {
    await axios.get('https://code2doc2022.herokuapp.com/').then((res) => {
      userID = res.data.userId;
    });
    function handleFiles() {
      const fileList = this.files;
      for (let i = 0; i < fileList.length; i++) {
        uploadedFiles.push(fileList[i]);
        console.log(fileList[i]);
        uploadedFiles = uploadedFiles;
      }
    }
    const uploadButtons = document.querySelectorAll('.upload');
    uploadButtons.forEach((button) => {
      button.addEventListener('change', handleFiles, false);
      console.log('ok');
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
    console.log('File(s) dropped');

    // Prevent default behavior (Prevent file from being opened)
    ev.preventDefault();

    if (ev.dataTransfer.items) {
      // Use DataTransferItemList interface to access the file(s)
      for (var i = 0; i < ev.dataTransfer.items.length; i++) {
        // If dropped items aren't files, reject them
        if (ev.dataTransfer.items[i].kind === 'file') {
          var file = ev.dataTransfer.items[i].getAsFile();
          console.log('... file[' + i + '].name = ' + file.name);
          uploadedFiles.push(file);
          uploadedFiles = uploadedFiles;
        }
      }
    } else {
      // Use DataTransfer interface to access the file(s)
      for (var i = 0; i < ev.dataTransfer.files.length; i++) {
        console.log(
          '... file[' + i + '].name = ' + ev.dataTransfer.files[i].name
        );
        uploadedFiles.push(ev.dataTransfer.files[i]);
        uploadedFiles = uploadedFiles;
      }
    }
  }

  function dragOverHandler(ev) {
    console.log('File(s) in drop zone');
    // Prevent default behavior (Prevent file from being opened)
    ev.preventDefault();
  }

  $: if (uploadedFiles.length) {
    uploaded = true;
  } else {
    uploaded = false;
  }
</script>

<main>
  <header class="px-5 py-4 flex flex-row align-items-center gap-3">
    <img src="./logo.svg" class="h-3rem" alt="logo" />
    <h2>code2doc</h2>
  </header>
  <div class="content px text-center">
    <h1 class="lg:text-8xl md:text-7xl text-6xl mb-5">code_2_doc</h1>
    <p class="text-xl mb-6 xl:w-3 lg:w-6 tagline">
      A quick solution to get all your code converted to a document.
    </p>
    <div
      id="drop_zone"
      on:drop={dropHandler}
      on:dragover={dragOverHandler}
      class="relative border-dashed border-round-md p-5 sm:w-10 w-9 flex flex-row flex-wrap h-25rem mt-1"
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
        {uploadedFiles.length}
      {/if}
      <div
        style="top: -20px; right: -20px"
        class="absolute {!uploaded ? 'hidden' : null}"
      >
        <label
          class="inline-block button button-circle text-center"
          for="upload">+</label
        >
        <input type="file" id="upload" class="upload" multiple />
      </div>
    </div>
    <button class="mt-7 button button-inverse">Convert</button>
  </div>
  <footer class="px">
    <p class="text-center">
      Made with <span style={{ color: '#FF6B6B' }}>&#9829;</span> by GDSC
    </p>
  </footer>
</main>

<style>
  @media (min-width: 640px) {
    main {
      max-width: none;
    }
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

  #drop_zone {
    border-color: var(--primary);
  }
</style>
