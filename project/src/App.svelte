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

  onMount(async () => {
    await axios.get('https://code2doc2022.herokuapp.com/').then((res) => {
      userID = res.data.userId;
    });

    let myDropzone = new Dropzone('#drop-form', {
      paramName: 'files',
      //   uploadMultiple: true,
      headers: {
        'User-Name': userID,
        'File-Format': '',
      },
      addRemoveLinks: true,
      previewTemplate: document.querySelector('#template-container').innerHTML,
      dictDefaultMessage: 'Add Files to Convert',
      previewsContainer: document.querySelector('#previews-container'),
    });
    myDropzone.on('addedfile', (file) => {
      console.log(`File added: ${file.name}`);
      uploaded = true;
    });
  });

  $: try {
    document.querySelector('.dropzone').dropzone.options.headers[
      'File-Format'
    ] = fileFormat;
  } catch (error) {
    console.log(error);
  }

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
</script>

<main>
  <header class="px">
    <h2>LOGO</h2>
  </header>
  <div class="content px text-center">
    <h1 class="mb-5">Code2Doc</h1>
    <p class="mb-6 tagline">
      Lorem ipsum dolor sit amet, consectetur adipiscing elit. Vestibulum
      consectetur quam turpis, at pulvinar lorem ullamcorper eu. Morbi nec nulla
      eget justo venenatis condimentum.
    </p>
    <div id="previews-container" class="dropzone-previews" />
    <div class="buttons">
      {#if !processed}
        <form
          id="drop-form"
          action="https://code2doc2022.herokuapp.com/upload/uploadFiles"
          class="dropzone"
        >
          <input
            bind:value={fileFormat}
            required
            placeholder="py, js, cpp, etc."
          />
        </form>
      {/if}
      {#if processed}
        <button on:click={download}>Download</button>
      {:else if uploaded}
        <button on:click={process}>Process</button>
      {/if}
      {#if downloaded}
        <button on:click={() => window.location.reload()}
          >Convert More Files</button
        >
      {/if}
    </div>
    <template id="template-container">
      <div class="box dz-preview dz-file-preview">
        <div class="dz-details">
          <div class="dz-filename"><span data-dz-name /></div>
          <img alt="thumbnail" data-dz-thumbnail />
          <div class="dz-size" data-dz-size />
        </div>
        <div class="dz-progress">
          <span class="dz-upload" data-dz-uploadprogress />
        </div>
        <div class="dz-error-message"><span data-dz-errormessage /></div>
      </div>
    </template>
  </div>
  <footer class="px">
    <p class="text-center">Footer</p>
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
  }

  h1 {
    font-size: 4rem;
  }

  .box {
    border: 1px solid black;
    border-radius: 0.5rem;
    padding: 1rem;
  }

  .dropzone-previews {
    display: flex;
    flex-direction: row;
    gap: 1rem;
    margin-bottom: 3rem;
    flex-wrap: wrap;
    align-items: center;
    justify-content: center;
  }

  .buttons {
    display: flex;
    flex-direction: row;
    gap: 1rem;
  }
</style>
