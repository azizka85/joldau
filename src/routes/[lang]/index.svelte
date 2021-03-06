<script lang="ts" context="module">
export async function preload({ params }) {
  const res = await this.fetch(`${params.lang}.json`);
  const data = await res.json();  

  if(res.status == 200) {
    return {
      language: params.lang, 
      data
    };
  } else {
    this.error(res.status, data.message);
  }  
}  
</script>

<script lang="ts">
import { _ } from 'svelte-i18n'; 
import { stores } from '@sapper/app';
import { DEFAULT_TITLE, formatDate } from '../../globals';
import type { SearchSettings } from '../../globals';
import type { Category } from '../../globals';
import Loader from '../../components/Loader.svelte';
import NavLayout from '../_navLayout.svelte';
import Alert from '../../components/Alert.svelte';

export let language: string;
export let data: Category;

let title;
let loading = false;
let messageBox: Alert;

export async function search(evt: CustomEvent<SearchSettings>) {
  loading = true;

  let error = null;

  try {
    let url = `${language}.json`;
    
    const params: string[] = [];   
    if(evt.detail.answersCountFrom) params.push('answersCountFrom=' + evt.detail.answersCountFrom.toString());    
    if(evt.detail.answersCountTo) params.push('answersCountTo=' + evt.detail.answersCountTo.toString());    
    if(evt.detail.categoryTitle) params.push('categoryTitle=' + evt.detail.categoryTitle);    
    if(evt.detail.content) params.push('content=' + evt.detail.content);    
    if(evt.detail.contentTitle) params.push('contentTitle=' + evt.detail.contentTitle);    
    if(evt.detail.createdAtFrom) params.push('createdAtFrom=' + evt.detail.createdAtFrom.getTime().toString());    
    if(evt.detail.createdAtTo) params.push('createdAtTo=' + evt.detail.createdAtTo.getTime().toString());    
    if(evt.detail.description) params.push('description=' + evt.detail.description);    
    if(evt.detail.updatedAtFrom) params.push('updatedAtFrom=' + evt.detail.updatedAtFrom.getTime().toString());    
    if(evt.detail.updatedAtTo) params.push('updatedAtTo=' + evt.detail.updatedAtTo.getTime().toString());  
    
    if(params.length > 0) url += '?' + params.join("&");

    const res = await fetch(url);    

    if(res.status == 200) {
      data = await res.json();    
    } else {
      error = `${res.status}: ${$_('fetch_error')}`;
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

const { preloading } = stores();

$: {
  if($preloading) {
    title = $_('loading');
  } else {
    title = `${$_('guide')} - ${DEFAULT_TITLE}`;
  }
}
</script>

<svelte:head>
	<title>{title}</title>
</svelte:head>

<NavLayout on:search={search}>
  {#if $preloading || loading}
    <Loader />
  {:else}
    <main>
      <div class="title">
        <h1>{$_('guide')}</h1>
        <button class="btn">
          <svg class="btn-icon" viewBox="0 0 16 16">
            <path fill-rule="evenodd" d="M8 3a5 5 0 1 0 4.546 2.914.5.5 0 0 1 .908-.417A6 6 0 1 1 8 2v1z"/>
            <path d="M8 4.466V.534a.25.25 0 0 1 .41-.192l2.36 1.966c.12.1.12.284 0 .384L8.41 4.658A.25.25 0 0 1 8 4.466z"/>
          </svg>			
        </button>
      </div>
      <h2>{data.answersCount} {$_('answers_count', {values: {n: data.answersCount}})}</h2>	
      {#if data.categories.length > 0}
        <div class="title underline">
          <h1>{$_('categories')}</h1>
        </div>	
        <div class="cards">
          {#each data.categories as category}
            <div class="card">
              <div class="card-title">
                <a href="{language}/category/{category.name}" class="card-title-content">
                  {category.title}
                </a>
                <div class="card-title-btns">
                  <svg 
                    class="btn-icon" 
                    class:stroke={category.id % 2 === 0}
                    class:fill={category.id % 2 > 0}
                    viewBox="0 0 21 21"
                  >
                    <path stroke-miterlimit="2" stroke-width="9%" d="M16.79 5.2A4.15 4.15 0 0 0 13.85 4c-1.1 0-2.15.44-2.93 1.21l-.42.4-.41-.4A4.17 4.17 0 0 0 3 8.08c0 1.1.44 2.12 1.22 2.9l5.97 5.88c.09.09.2.13.3.13a.4.4 0 0 0 .3-.12l6-5.88a4.04 4.04 0 0 0 0-5.8z"/>
                  </svg>												
                </div>
              </div>
              <p class="description">{category.description}</p>
              <p class="info">{category.answersCount} {$_('answers_count', {values: {n: category.answersCount}})}</p>
              <p class="date-time">
                {$_('created')}: {formatDate(category.createdAt, language)}
              </p>
            </div>
          {/each}
        </div>		
      {/if}	
      {#if data.answers.length > 0}
        <div class="title underline">
          <h1>{$_('answers')}</h1>
        </div>	
        <div class="cards">
          {#each data.answers as answer}
            <div class="card">
              <div class="card-title">
                <a href="{language}/answer/{answer.path}/{answer.name}" class="card-title-content">
                  {answer.title}
                </a>
                <div class="card-title-btns">
                  <svg 
                    class="btn-icon" 
                    class:stroke={answer.id % 2 === 0}
                    class:fill={answer.id % 2 > 0}
                    viewBox="0 0 21 21"
                  >
                    <path stroke-miterlimit="2" stroke-width="9%" d="M16.79 5.2A4.15 4.15 0 0 0 13.85 4c-1.1 0-2.15.44-2.93 1.21l-.42.4-.41-.4A4.17 4.17 0 0 0 3 8.08c0 1.1.44 2.12 1.22 2.9l5.97 5.88c.09.09.2.13.3.13a.4.4 0 0 0 .3-.12l6-5.88a4.04 4.04 0 0 0 0-5.8z"/>
                  </svg>						              						
                </div>
              </div>
              <p class="info">0 {$_('cnt_messages', {values: {n: 0}})}</p>
              <p class="date-time">
                {$_('updated')}: {formatDate(answer.updatedAt, language)}
              </p>
            </div>
          {/each}
        </div>		
      {/if}	
    </main>    
  {/if}  
  <Alert bind:this={messageBox} />
</NavLayout>
