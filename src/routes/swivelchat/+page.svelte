<script lang="ts">
    import { onMount, onDestroy } from 'svelte';
    import { fade, fly } from 'svelte/transition';
    import { Client, Databases, Functions, ID, Query } from 'appwrite';
    import { Send, Loader2 } from 'lucide-svelte';
  
    const client = new Client()
      .setEndpoint('https://cloud.appwrite.io/v1')
      .setProject('671cb149000760017e54');
  
    const databases = new Databases(client);
    const functions = new Functions(client);
  
    const DATABASE_ID = '671cb3dd000c42f0d136';
    const COLLECTION_ID = '671cb3eb001a558ee5f4';
  
    interface Message {
      id: string;
      role: 'user' | 'assistant';
      content: string;
      createdAt: Date;
    }
  
    let messages: Message[] = [];
    let input = '';
    let isLoading = false;
    let error = '';
    let scrollContainer: HTMLDivElement;
  
    onMount(async () => {
      await fetchMessages();
      subscribeToMessages();
    });
  
    onDestroy(() => {
      client.subscribe(`databases.${DATABASE_ID}.collections.${COLLECTION_ID}.documents`, () => {});
    });
  
    async function fetchMessages() {
      try {
        const response = await databases.listDocuments(DATABASE_ID, COLLECTION_ID, [
          Query.orderAsc('createdAt'),
        ]);
        messages = response.documents.map((document) => document as unknown as Message);
      } catch (err) {
        console.error('Error fetching messages:', err);
        error = 'Failed to load messages. Please try again.';
      }
    }
  
    function subscribeToMessages() {
      client.subscribe(`databases.${DATABASE_ID}.collections.${COLLECTION_ID}.documents`, (response) => {
        if (response.events.includes('databases.*.collections.*.documents.*.create')) {
          messages = [...messages, response.payload as Message];
          scrollToBottom();
        }
      });
    }
  
    async function sendMessage() {
      if (!input.trim() || isLoading) return;
  
      isLoading = true;
      error = '';
      const userMessage = input.trim();
      input = '';
  
      try {
        await databases.createDocument(DATABASE_ID, COLLECTION_ID, ID.unique(), {
          role: 'user',
          content: userMessage,
          createdAt: new Date(),
        });
  
        const response = await functions.createExecution('LLM', JSON.stringify({ message: userMessage }));
  
        const responseData = JSON.parse(response.responseBody);
        if (typeof responseData.reply !== 'string') {
          throw new Error('Invalid response from LLM function');
        }
  
        await databases.createDocument(DATABASE_ID, COLLECTION_ID, ID.unique(), {
          role: 'assistant',
          content: responseData.reply,
          createdAt: new Date(),
        });
      } catch (err) {
        console.error('Error sending message:', err);
        error = 'Failed to send message. Please try again.';
      } finally {
        isLoading = false;
      }

    }  
    function scrollToBottom() {
      setTimeout(() => {
        if (scrollContainer) {
          scrollContainer.scrollTop = scrollContainer.scrollHeight;
        }
      }, 0);
    }
</script>
  
  <div class="flex flex-col h-screen bg-gray-900" in:fade={{ duration: 300 }}>
    <div class="flex-1 overflow-hidden">
      <div class="max-w-2xl mx-auto p-4 h-full flex flex-col">
        <h1 class="text-2xl font-bold text-center text-gray-100 mb-4" in:fly={{ y: -20, duration: 500 }}>SwivelChat</h1>
        <div 
          bind:this={scrollContainer}
          class="flex-1 overflow-y-auto space-y-4 pr-4"
        >
          {#each messages as message (message.id)}
            <div 
              class="flex {message.role === 'user' ? 'justify-end' : 'justify-start'}"
              in:fly={{ y: 20, duration: 300 }}
              out:fade={{ duration: 200 }}
            >
              <div 
                class="rounded-lg p-3 max-w-[80%] {message.role === 'user' ? 'bg-blue-600 text-white' : 'bg-gray-700 text-gray-100'}"
              >
                {message.content}
              </div>
            </div>
          {/each}
          {#if isLoading}
            <div class="flex justify-start" in:fly={{ y: 20, duration: 300 }} out:fade={{ duration: 200 }}>
              <div class="rounded-lg p-3 bg-gray-700 text-gray-100">
                <Loader2 class="w-5 h-5 animate-spin" />
              </div>
            </div>
          {/if}
        </div>
        {#if error}
          <p class="text-red-500 text-center my-2" in:fly={{ y: 20, duration: 300 }} out:fade={{ duration: 200 }}>{error}</p>
        {/if}
        <form on:submit|preventDefault={sendMessage} class="mt-4 flex gap-2" in:fly={{ y: 20, duration: 500 }}>
          <input
            bind:value={input}
            placeholder="Type your message here..."
            class="flex-grow p-2 rounded-md bg-gray-700 text-white placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-blue-500 transition-all duration-300 ease-in-out"
          />
          <button 
            type="submit" 
            disabled={isLoading}
            class="p-2 rounded-md bg-blue-600 text-white disabled:opacity-50 focus:outline-none focus:ring-2 focus:ring-blue-500 transition-all duration-300 ease-in-out hover:bg-blue-700"
          >
            {#if isLoading}
              <Loader2 class="w-5 h-5 animate-spin" />
            {:else}
              <Send class="w-5 h-5" />
            {/if}
          </button>
        </form>
      </div>
    </div>
  </div>
  
  <style>
    :global(body) {
      @apply bg-gray-900 text-gray-100;
    }
  
    @keyframes pulse {
      0%, 100% {
        opacity: 1;
      }
      50% {
        opacity: 0.5;
      }
    }
  
    .animate-pulse {
      animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite;
    }
  </style>