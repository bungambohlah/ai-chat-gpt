<script lang="ts">
	import SvelteMarkdown from 'svelte-markdown';
	import type { ChatMessage } from './chat/chat.types';

	let messages = [
		{ role: 'system', content: "Hey there! What's on your mind?" }
	] satisfies ChatMessage[];
	let newMessage = '';
	let loading = false;

	async function chat(event: Event) {
		event.preventDefault();
		loading = true;
		messages = [...messages, { role: 'user', content: newMessage }] satisfies ChatMessage[];
		newMessage = '';

		const res = await fetch('/chat', {
			method: 'POST',
			body: JSON.stringify({ messages })
		});

		const chatGPTMessage = await res.json();

		loading = false;

		messages = [...messages, chatGPTMessage] satisfies ChatMessage[];
	}
</script>

<div class="chatbox">
	<div class="header">Let's Chat</div>
	<div class="messages">
		{#each messages as message}
			{#if ['system', 'assistant'].indexOf(message.role) >= 0}
				<div class="message">
					<span class="sender">{message.role}:</span>
					<span class="text"><SvelteMarkdown source={message.content} /></span>
				</div>
			{/if}
			{#if message.role === 'user'}
				<div class="message receiver">
					<span class="text"><SvelteMarkdown source={message.content} /></span>
				</div>
			{/if}
		{/each}
		{#if loading}
			<div class="message">
				<span class="sender">assistant:</span>
				<span class="text">loading...</span>
			</div>
		{/if}
	</div>
	<div class="input-box">
		<input type="text" placeholder="Type your message..." bind:value={newMessage} />
		<button on:click={chat}>Send</button>
	</div>
</div>

<style>
</style>
