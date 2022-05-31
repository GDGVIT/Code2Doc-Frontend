<script>
  import { register } from 'register-service-worker';

  register('/service-worker.js', {
    registrationOptions: { scope: './' },
    ready(registration) {
      console.log('Service worker is active.');
    },
    registered(registration) {
      console.log('Service worker has been registered.');
    },
    cached(registration) {
      console.log('Content has been cached for offline use.');
    },
    updatefound(registration) {
      console.log('New content is downloading.');
    },
    updated(registration) {
      console.log('New content is available; please refresh.');
    },
    offline() {
      console.log(
        'No internet connection found. App is running in offline mode.'
      );
    },
    error(error) {
      console.error('Error during service worker registration:', error);
    },
  });
  import { onMount, onDestroy } from 'svelte';
  import axios from 'axios';
  import Dropzone from 'dropzone';
  import 'primeflex/primeflex.css';

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
</style>
