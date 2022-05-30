<script>
  import { onMount, onDestroy } from 'svelte';
  import axios from 'axios';
  import Dropzone from 'dropzone';

  let userID;
  let processed = false;

  onMount(async () => {
    await axios.get('https://code2doc2022.herokuapp.com/').then((res) => {
      userID = res.data.userId;
    });

    let myDropzone = new Dropzone('#drop-form', {
      paramName: 'files',
      //   uploadMultiple: true,
      headers: {
        'User-Name': userID,
        'File-Format': 'py',
      },
    });
    myDropzone.on('addedfile', (file) => {
      console.log(`File added: ${file.name}`);
    });
    myDropzone.on('');
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
      .then((res) => window.open(URL.createObjectURL(res.data)));
  };
</script>

<main>
  <h1>Welcome to Code2Doc!</h1>
  <p>User ID: {userID}</p>
  <form
    id="drop-form"
    action="https://code2doc2022.herokuapp.com/upload/uploadFiles"
    class="dropzone"
  />
  {#if processed}
    <button on:click={download}>Download</button>
  {:else}
    <button on:click={process}>Process</button>
  {/if}
</main>

<style>
  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>
