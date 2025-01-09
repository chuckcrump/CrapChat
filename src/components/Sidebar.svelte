<script>
  export let conversations
  export let newConvo
  export let model
  export let responses

  function selectConversation(conversation) {
    console.log(conversation)
    model = conversation.currentModel
    responses = conversation.responses
  }
</script>

<div class="flex flex-col h-screen w-56 bg-gray-800 text-white fixed">
  <h1>Hello!</h1>
  <button onclick={newConvo}>Add</button>
  {#each conversations as conversations}
    <button onclick={() => selectConversation(conversations)}>{conversations.name}</button>
  {/each}
</div>