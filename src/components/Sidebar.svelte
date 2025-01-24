<script lang="ts">
  let {conversations, newConvo, model, responses} = $props()

  let showConvoDelete = $state(false) 

  function selectConversation(conversation: any) {
    model = conversation.currentModel
    responses = conversation.responses
  }

  function removeConvo(index: number) {
    conversations.splice(index, 1);
    conversations = conversations    
  }
</script>

<div class="flex flex-col h-screen w-56 bg-gray-800 text-white fixed">
  <h1 class="text-center"><strong>CrapChat</strong></h1>
  <button onclick={newConvo}>Add</button>
  {#each conversations as conversations, index}
    <button
      onclick={() => selectConversation(conversations)}
      onmouseenter={() => (
        showConvoDelete = true, 
        console.log(showConvoDelete)
      )}
      onmouseleave={() => (showConvoDelete = false)}
      class="bg-gray-500 p-1 m-1 rounded-lg w-[200px] text-left"
    >
      {conversations.name}
      {#if showConvoDelete}
        <button
          class="bg-red-500 h-[25px] w-[25px] rounded-lg"
          onclick={() => removeConvo(index)}
        >X</button>
      {/if}
    </button>
  {/each}
</div>
