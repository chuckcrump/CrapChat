<script lang="ts">
    import { marked } from 'marked';
    import ollama from 'ollama'
    import { onMount } from 'svelte';
    import eject from './components/eject.svelte'

    //prompt and response
    let theMessage: string = '';
    let prompt: string = '';
    let htmlContent = marked(theMessage);
    let showResponse = false;
    let responses: any[] = [];

    let currentModel = '';
    let showModelDropdown = false;

    onMount(() => {
        ollamaModels();
    })

    async function ollamaResponse() {
        if (prompt === '') {
            alert('prompt cannot be empty');
            return;
        }
        
        if (currentModel === '') {
            alert('please select a model')
            return;
        }

        theMessage = '';
        showResponse = true;

        const message = { role: 'user', content: prompt }
        const response = await ollama.chat({ model: currentModel, messages: [message], stream: true })
        for await (const part of response) {
            theMessage = theMessage + part.message.content;
            htmlContent = marked(theMessage);
        }
        responses = [...responses, theMessage ];
        theMessage = '';
    }

    let modelNames: string[] = [];

    async function ollamaModels() {
        const ollamaModel = await ollama.list();
        modelNames = ollamaModel.models.map((model: any) => model.name);
        console.log(modelNames)
    }

    function selectModel(button: any) {
        console.log(button)
        currentModel = button
        showModelDropdown = false;
    }

    function modelDropdown() {
        showModelDropdown = !showModelDropdown;
    }

</script>

<div class="flex justify-center items-center p-2 bg-gray-400">
    <div class="bg-gray-300 rounded-lg p-1 text-gray-600 mr-1">
    {#if currentModel === ''}
        <button class="w-96 text-center" onclick={modelDropdown}>No model selected</button>
    {:else}
        <button class="w-96 text-center" onclick={modelDropdown}>Using <strong>{currentModel}</strong></button>
    {/if}
    </div>
    <button onclick={() => {
        currentModel = '';
    }} 
    class="flex items-center justfy-between bg-red-500 p-1 text-white rounded-lg"
    aria-label="eject"
    >
        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="#ffffff" class="m-1" viewBox="0 0 256 256"><path d="M232,200a8,8,0,0,1-8,8H32a8,8,0,1,1,0-16H224A8,8,0,0,1,232,200ZM25.59,150.84a16,16,0,0,1,2-17.07L109.26,32.94a24.11,24.11,0,0,1,37.48,0l81.65,100.83A16.1,16.1,0,0,1,215.91,160H40.09A16,16,0,0,1,25.59,150.84ZM40,143.91s0,.09.08.11l175.83,0s.08-.09.08-.13L134.3,43a8.1,8.1,0,0,0-12.6,0L40,143.84A.28.28,0,0,0,40,143.91Z"></path></svg>
    </button>
</div>

{#if showModelDropdown}
<div class="flex justify-center items-center">
<div class="bg-gray-400 fixed w-96 justify-center items-center p-2 rounded-lg top-[48px]">
    <div class="flex flex-col">
    {#each modelNames as names}
        <button onclick={() => selectModel(names)} class="bg-blue-500 p-1 m-1 text-white rounded-lg">{names}</button>
    {/each}
    </div>
</div>
</div>
{/if}

<div class="fixed bottom-1 right-0 left-0 z-10 flex justify-center p-2">
<input type="text" bind:value={prompt} class="bg-gray-300 p-2 rounded-lg"/>
<button onclick={ollamaResponse} class="bg-blue-500 rounded-lg p-2">clickme</button>
</div>

{#each responses as chats}
    <p class="bg-[#282828] p-2 rounded-lg text-white m-1">{@html marked(chats)}</p>
{/each}

{#if showResponse === true} 
    {#if htmlContent}
    <div class="bg-[#282828] p-2 rounded-lg text-white m-1">
        {@html htmlContent}
    </div>
    {:else}
        <p>Loading...</p>
    {/if}
{/if}