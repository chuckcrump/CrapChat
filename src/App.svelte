<script lang="ts">
  import { Marked } from 'marked';
  import ollama from 'ollama'
  import { onMount } from 'svelte';
  import Sidebar from './components/Sidebar.svelte';
  import hljs from 'highlight.js';
  import { markedHighlight } from 'marked-highlight';
  import 'highlight.js/styles/atom-one-dark.css'
  import { slide } from 'svelte/transition';

  //prompt and response
  let theMessage: string = '';
  let prompt: string = '';
  let showResponse = false;
  let responses: any[] = [];
  let chatMessage = {};

  //models
  let currentModel = '';
  let showModelDropdown = false;
  let modelWarn = false;

  let isGenerating = false;
  
  //conversations
  let history: any[] = [];

  //const renderer = new marked.Renderer();
  //renderer.code = function(code: string, language: string | undefined, isEscaped: boolean) {
  //  const valLang = language && hljs.getLanguage(language) ? language : "plaintext";
  //  return `<pre><code class="hljs ${valLang}">${hljs.highlightAuto(code, [valLang]).value}</code></pre>`;
  //}

  const marked = new Marked(
    markedHighlight({
      emptyLangClass: "hljs",
      langPrefix: "hljs language-",
      highlight(code, lang, info) {
        const language = hljs.getLanguage(lang) ? lang : "plaintext"
        return hljs.highlight(code, {language}).value
      }
    })
  )

  let htmlContent = marked.parse(theMessage);


  function addPrompt() {
    if (prompt === '') {
      alert('prompt cannot be empty');
      return;
    }

    chatMessage = {
      role: 'user',
      userContent: prompt
    }

    responses = [...responses, chatMessage];
  }

  //get and format ollama response

  function addCopy() {
    const pre = document.querySelectorAll("pre")

    pre.forEach(pre => {

      const button = document.createElement("button")

      let isCopied = false

      const updateBtn = () => {
        button.textContent = isCopied ? "Copied!" : "Copy"

        button.className = isCopied ? `flex items-center justify-center bg-green-500 p-[1px] m-[2px] rounded-md ml-auto mr-1` :
        `flex items-center justify-center bg-blue-500 p-[1px] m-[2px] rounded-md ml-auto mr-1`
      }

      updateBtn()

      button.addEventListener("click", () => {
        isCopied = true
        updateBtn()
        const content = pre.querySelector("code")?.innerText;
        navigator.clipboard.writeText(content)
      })

      pre.style.position = "realtive";
      button.style.position = "flex"
      pre.appendChild(button)
    })
  }

  function removeBtn() {
    const pre = document.querySelectorAll("pre")

    pre.forEach(pre => {
      pre.querySelectorAll("button").forEach(e => e.remove())
    })
  }

  async function ollamaResponse() {
      if (currentModel === '') {
        modelWarn = true;  
        return;
      }

      addPrompt();

      theMessage = '';
      showResponse = true;
      isGenerating = true;
      modelWarn = false;
      removeBtn();

      const message = { role: 'user', content: prompt }
      const response = await ollama.chat({ model: currentModel, messages: [message], stream: true })
      for await (const part of response) {
        theMessage = theMessage + part.message.content;
        htmlContent = marked.parse(theMessage);
      }

      setTimeout(() => {
        addCopy()
      }, 100)
      //setTimeout(() => {
      //  document.querySelectorAll('code').forEach((block) => {
      //    hljs.highlightElement(block as HTMLElement)
      //  });
      //  addCopy()
      //}, 100)

      chatMessage = {
      role: 'ollama',
      ollamaContent: theMessage
      }

      responses = [...responses, chatMessage ];
      theMessage = '';
      isGenerating = false;
      showResponse = false;
      prompt = ""
  }

  let modelNames: string[] = [];

  //handle ollama models
  async function ollamaModels() {
      const ollamaModel = await ollama.list();
      modelNames = ollamaModel.models.map((model: any) => model.name);
  }

  function selectModel(button: any) {
      console.log(button)
      currentModel = button
      showModelDropdown = false;
  }

  function modelDropdown() {
      showModelDropdown = !showModelDropdown;
  }

  function newChat() {
    history = [...history, {
      responses: [...responses],
      currentModel: currentModel,
      date: new Date(),
      name: responses[0].userContent
    }];

    responses = [];
    currentModel = '';
  }

  const contents = responses.map(responses => responses.content)
  console.log(contents)

  onMount(() => {
      ollamaModels();
      hljs.configure({ignoreUnescapedHTML: true})
      let input = document.getElementById("prompt")
      input?.addEventListener("keypress", function(event) {
        if (event.key === "Enter") {
          event.preventDefault();
          ollamaResponse()
        }
      })
  })

</script>


<Sidebar
conversations={history}
newConvo={newChat}
model={currentModel}
responses={responses}
/>

<div class="ml-56">
<div class="flex justify-center items-center p-2 bg-gray-400">
    <div class="bg-gray-300 rounded-lg p-1 text-gray-600 mr-1">
    {#if currentModel === ''}
        <button class="model-btn" onclick={modelDropdown}>No model selected</button>
    {:else}
        <button class="model-btn" onclick={modelDropdown}>Using <strong>{currentModel}</strong></button>
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
  <div class="flex w-full ml-56">
    <div class="flex justify-center items-center top-0 ml-56 left-[224px]">
        <div class="bg-gray-400 fixed w-96 justify-center items-center p-2 rounded-lg top-[48px] left-[590px]">
            <div class="flex flex-col">
            {#each modelNames as names}
                <button onclick={() => selectModel(names)} class="bg-blue-500 p-1 m-1 text-white rounded-lg">{names}</button>
            {/each}
            </div>
        </div>
    </div>
  </div>
{/if}

<div id="warning">
  {#if modelWarn}
    <h1>Please select a model!</h1>
  {/if}
</div>

<div class="fixed ml-56 bottom-1 right-0 left-0 z-10 flex justify-center p-2">
  <input id="prompt" type="text" bind:value={prompt} class="bg-gray-300 p-2 rounded-lg w-64 transition-all duration-300"/>
  {#if isGenerating}
    <button onclick={() => {
      if (isGenerating === true) {
        isGenerating = false;
        ollama.abort();
      } else {
        return;
      }
    }} class="bg-[#5e98c5] rounded-lg p-2" aria-label="stop">
      <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="#ffffff" viewBox="0 0 256 256"><path d="M188,88a27.75,27.75,0,0,0-12,2.71V60a28,28,0,0,0-41.36-24.6A28,28,0,0,0,80,44v6.71A27.75,27.75,0,0,0,68,48,28,28,0,0,0,40,76v76a88,88,0,0,0,176,0V116A28,28,0,0,0,188,88Zm12,64a72,72,0,0,1-144,0V76a12,12,0,0,1,24,0v44a8,8,0,0,0,16,0V44a12,12,0,0,1,24,0v68a8,8,0,0,0,16,0V60a12,12,0,0,1,24,0v68.67A48.08,48.08,0,0,0,120,176a8,8,0,0,0,16,0,32,32,0,0,1,32-32,8,8,0,0,0,8-8V116a12,12,0,0,1,24,0Z"></path></svg>
    </button>
    {:else if prompt === ''}
    <button aria-label="noprompt" class="bg-gray-500 rounded-lg p-2">
      <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="#ffffff" viewBox="0 0 256 256"><path d="M227.32,28.68a16,16,0,0,0-15.66-4.08l-.15,0L19.57,82.84a16,16,0,0,0-2.49,29.8L102,154l41.3,84.87A15.86,15.86,0,0,0,157.74,248q.69,0,1.38-.06a15.88,15.88,0,0,0,14-11.51l58.2-191.94c0-.05,0-.1,0-.15A16,16,0,0,0,227.32,28.68ZM157.83,231.85l-.05.14,0-.07-40.06-82.3,48-48a8,8,0,0,0-11.31-11.31l-48,48L24.08,98.25l-.07,0,.14,0L216,40Z"></path></svg> 
    </button>
  {:else}
    <button aria-label="send" onclick={ollamaResponse} class="bg-[#5e98c5] rounded-lg p-2">
      <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" fill="#ffffff" viewBox="0 0 256 256"><path d="M227.32,28.68a16,16,0,0,0-15.66-4.08l-.15,0L19.57,82.84a16,16,0,0,0-2.49,29.8L102,154l41.3,84.87A15.86,15.86,0,0,0,157.74,248q.69,0,1.38-.06a15.88,15.88,0,0,0,14-11.51l58.2-191.94c0-.05,0-.1,0-.15A16,16,0,0,0,227.32,28.68ZM157.83,231.85l-.05.14,0-.07-40.06-82.3,48-48a8,8,0,0,0-11.31-11.31l-48,48L24.08,98.25l-.07,0,.14,0L216,40Z"></path></svg>
    </button>
  {/if}

</div>

{#each responses as responses}
  {#if responses.role === 'user'}
    <p class="bg-[#242424] p-2 rounded-lg text-white m-1 text-right">{responses.userContent}</p>
  {:else if responses.role === 'ollama'}
    <p class="bg-[#282828] p-2 rounded-lg text-white m-1">{@html marked.parse(responses.ollamaContent)}</p>
  {/if}
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
</div>
