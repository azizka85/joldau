<script lang="ts">
import { _, locale }  from 'svelte-i18n';
import { goto } from '@sapper/app';
import { stores } from '@sapper/app';
import { saveSession, session } from '../../../stores/session';
import Loader from '../../../components/Loader.svelte';
import Alert from '../../../components/Alert.svelte';

let title = $_('sign-in');
let loading = false;

let emailElem: HTMLInputElement;
let passwordElem: HTMLInputElement;

let messageBox: Alert;

const { preloading } = stores();

$: {
  if($preloading) {
    title = $_('loading');
  } else {
    title = $_('sign-in');
  }
}

export function cancel() {
  goto($locale);
}

export async function submit() {
  loading = true;

  const body = {
    email: emailElem.value,
    password: passwordElem.value
  };

  let error = null;

  try {
    const res = await fetch(`${$locale}/signin.json`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(body)
    });
    const data = await res.json();

    if(res.status == 201) {
      session.set(data);
      saveSession();     
      goto($locale);       
    } else {
      if(data.code == 1) {
        error = `${res.status}.${data.code}: ${$_('incorrect_email_password')}`;
      } else {
        error = `${res.status}: ${$_('fetch_error')}`;
      }      
    }  
  } catch(e) {    
    error = e;
  }

  if(error) { 
    console.error(error);
    messageBox.addMessage(error);
  }

  loading = false;
}
</script>

<svelte:head>
  <title>{title}</title>
</svelte:head>

{#if $preloading}
  <Loader />
{:else}
  <div class="single">
    <div class="card">
      <div class="card-title">
        <h2 class="card-title-content">{$_('sign-in')}</h2>
      </div>
      <form class="form" on:submit|preventDefault={submit}>
        <div class="form-item">
          <label for="email" class="form-item-name">{$_('email')}: </label>        
          <input 
            id="email" name="email" 
            type="email" 
            class="form-item-value" 
            placeholder={$_('email')} 
            required
            disabled={loading}
            bind:this={emailElem}
          >
        </div>
        <div class="form-item">
          <label for="password" class="form-item-name">{$_('password')}: </label>        
          <input 
            id="password" name="password" 
            type="password" 
            class="form-item-value" 
            placeholder={$_('password')} 
            required
            disabled={loading}
            bind:this={passwordElem}
          >
        </div>
        <div class="form-item right">
          {#if loading}
            {$_('loading')}
          {:else}
            <a href="{$locale}/signup" class="form-item-link">{$_('sign-up')}</a>
          {/if}
        </div>
        <div class="form-item right">
          {#if loading}
            <Loader />        
          {/if}
          <button 
            class="form-item-button bg-primary" 
            type="submit"
            disabled={loading}
          >
            {$_('sign-in')}
          </button>
          <button 
            class="form-item-button bg-secondary" 
            on:click={cancel} 
            type="button"
            disabled={loading}
          >
            {$_('cancel')}
          </button>        
        </div>
      </form>
    </div>    
  </div>
  <Alert bind:this={messageBox} />
{/if}
