<script lang="ts">
  import { Button, Modal, Spinner } from 'flowbite-svelte';
  import { onMount } from 'svelte';

  export let open = false;

  const loginEndpoint = '/api/Plugin/Authenticate';
  const scanEndpoint = '/api/Library/scan-all';

  let apiAddress = '';
  let apiKey = '';
  let jwtToken = '';
  onMount(() => {
    apiAddress = localStorage.getItem('kavitaHostAdress') || ''; // TODO: Cors?
    apiKey = localStorage.getItem('kavitaApiKey') || '';
    if (apiKey && apiAddress) {
      authenticate();
    }
  });

  let odpsUrl = '';
  function parseUrl() {
    try {
      const parsedUrl = new URL(odpsUrl);
      if (parsedUrl.pathname.startsWith('/api/opds/')) {
        // Save with api endpoint
        apiAddress = `${parsedUrl.protocol}//${parsedUrl.host}/api/`;
        apiKey = parsedUrl.pathname.split('/api/opds/')[1];

        localStorage.setItem('kavitaHostAdress', apiAddress);
        localStorage.setItem('kavitaApiKey', apiKey);

        odpsUrl = '';
        alert('Host Address and API Key saved!');
      } else {
        alert('Invalid URL path. The URL must contain /api/opds/.');
      }
    } catch (error) {
      alert('Invalid URL format.');
    }
  }

  async function authenticate() {
    console.log('Authenticating with Kavita');
    const loginUrl = `${loginEndpoint}?apiKey=${apiKey}&pluginName=mokuro`;
    const loginResponse = await fetch(loginUrl, {
      method: 'POST'
    });

    if (!loginResponse.ok) {
      throw new Error(`Error during authentication: ${loginResponse.statusText}`);
    }

    const loginData = await loginResponse.json();
    console.log(loginData);
    jwtToken = loginData.token;
  }
</script>

<Modal title="Kavita" bind:open outsideclose>
  <div class="flex flex-col gap-2">
    <input type="text" placeholder="Enter ODPS URL" bind:value={odpsUrl} />
    <Button on:click={parseUrl}>Save API Key</Button>

    {#if apiKey}
      <p>Saved API key: {apiKey}</p>
    {/if}

    <Button on:click={authenticate}>Authenticate</Button>

    {#if jwtToken}
      <p>{jwtToken}</p>
    {/if}
  </div>
</Modal>
