<script lang="ts">
  import { onMount } from 'svelte';
  import {login, handleRedirectAfterLogin, getFetch, SOLID_IDENTITY_PROVIDER } from './lib/SessionManager.js'; 
  import { readProfile, writeProfile  } from './lib/readWrite.js';
  let webId = undefined;
  let isLogged = false;

  onMount(async () => {
		webId = await handleRedirectAfterLogin();
    if(webId) { isLogged = true; }
	});

  const onLoginClick = async () => {
    isLogged =  await login();
    console.log('isLogged', isLogged)
  }

  let input_name = '';
  let writeSuccess = false;
  const onWriteSubmit = async (event) => {
    console.log('onWriteSubmit', isLogged)
    event.preventDefault();
    if(isLogged) {
       writeSuccess = await writeProfile(input_name, {webId, fetch: getFetch()});
    }
  }

  let readName = ''
  let readError = undefined;
  const onReadSubmit = async (event) => {
    console.log('onReadSubmit', isLogged)
    event.preventDefault();
    const {error, data } = await readProfile({webId, fetch: getFetch()});
    console.log({error, data })
    if(error) {
      readError = error;
    } else {
      readError = undefined;
      readName = data;
    }
  }
  
</script>

<main>
  <header>
    <h2>Getting Started</h2>
    <h3>with Inrupt JavaScript Client Libraries</h3>
  </header>
  <section id="login" class="panel">
    <div class="row">
      <label id="labelLogin" for="btnLogin"
        >1. Click the button to log into
        <span id="solid_identity_provider"
          ><a target="_blank" href="{SOLID_IDENTITY_PROVIDER}">{SOLID_IDENTITY_PROVIDER}</a></span
        >:
      </label>
      <button name="btnLogin" id="btnLogin" on:click={onLoginClick}>Login</button>
      <p id="labelStatus" class="labelStatus" role="alert">
        {#if webId === undefined} 
 ...not logged in yet - but enter any WebID to read from its profile...
        {:else}
          Your session is logged in with the WebID [<a target="_blank" href="{webId}">{webId}</a>].
        {/if}
      </p>
    </div>
  </section>

  <div id="read" class="panel">
    <div class="row">
      <form id="writeForm" on:submit={onWriteSubmit}>
        <label id="writelabel" for="input_name">2. Write your name: </label>
        <input
          type="text"
          id="input_name"
          name="name"
          size="50"
          placeholder="Your name here"
          bind:value={input_name}
        />
        <button type="submit">
          Write to Profile
        </button>
      </form>
    </div>

    <dl class="display">
      <dt>Writing status:&nbsp;</dt>
      <dd id="labelWriteStatus" class="labelStatus" role="alert">
        {#if !isLogged}
            ...you can't write [{name}] until you first login!
        {:else}
          Wrote [{name}] as name successfully!
        {/if}
        </dd>
    </dl>
  </div>

  <div id="read" class="panel">
    <div class="row">
      <form id="readForm"  on:submit={onReadSubmit}>
        <label id="readlabel" for="webID"
          >3. Read back name (anyone's!) from their WebID:
        </label>
        <input
          type="url"
          id="webID"
          name="webID"
          size="50"
          placeholder="...not logged in yet - but enter any WebID to read from its profile..."
          bind:value={webId}
        />
        <button type="submit" name="btnRead" id="btnRead">
          Read Profile
        </button>
      </form>
    </div>
    <dl class="display">
      <dt>Formatted Name (FN) read from Pod:&nbsp;</dt>
      <dd id="labelFN">
      {#if writeSuccess} 
        ...click the 'Read Profile' button to to see what the name might be now...?!
      {:else if readName}
        {readName}
      {:else if readError}
        Entered value [{webID}] does not appear to be a WebID. Error: [{readError}]
      {:else}
        Login first, or enter a WebID (any WebID!) to read from its profile
      {/if}
      </dd>
    </dl>
  </div>
</main>

<style>
  :root {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen,
      Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  }

  main {
    text-align: center;
    padding: 1em;
    margin: 0 auto;
  }

  
    h2,
    h3 {
      margin: 1rem 1.2rem 1rem 1.4rem;
    }

    header {
      border-bottom: #5795b9 solid;
      padding-left: 0.5rem;
    }

    .panel {
      border: 1px solid #005b81;
      border-radius: 4px;
      box-shadow: rgb(184, 196, 194) 0px 4px 10px -4px;
      box-sizing: border-box;

      padding: 1rem 1.5rem;
      margin: 1rem 1.2rem 1rem 1.2rem;
    }

    #login {
      background: white;
    }

    #read {
      background: #e6f4f9;
    }

    .labelStatus[role="alert"] {
      padding-left: 1rem;
      color: purple;
    }

    .display {
      margin-left: 1rem;
      color: gray;
    }

    dl {
      display: grid;
      grid-template-columns: max-content auto;
    }

    dd {
      font-weight: bold;
    }
</style>
