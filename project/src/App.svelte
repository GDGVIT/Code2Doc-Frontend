<script>
  import { onMount } from 'svelte';

  let userID;

  onMount(async () => {
    userID = 'adsw8239nssj';
  });
</script>

<main>
  <h1>Welcome to Code2Doc!</h1>
  <p>User ID: {userID}</p>
</main>

<style>
  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>
