<script lang="ts">
	import { onMount, tick } from 'svelte';
	import { slide } from 'svelte/transition';
	import { Button } from '$lib/components/ui/button/index.js';
	import { Input } from '$lib/components/ui/input/index.js';

	let messages: Array<{ id: number; sender: string; text: string }> = $state([]);
	let userInput = $state('');
	let isLoading = $state(false);
	let chatContainer: HTMLElement | null = $state(null);

	const BACKEND_URL = 'https://coinfantasy-be.onrender.com/chat';

	onMount(() => {
		messages = [
			{
				id: Date.now(),
				sender: 'agent',
				text: 'I am the CoinFantasy Trade Agent. You can ask me about my trading style, risk management, or favorite assets.'
			}
		];
	});

	async function scrollToBottom() {
		await tick();
		if (chatContainer) {
			chatContainer.scrollTop = chatContainer.scrollHeight;
		}
	}

	async function handleSubmit() {
		if (userInput.trim() === '' || isLoading) return;

		const userMessageText = userInput;
		isLoading = true;
		messages = [...messages, { id: Date.now(), sender: 'user', text: userMessageText }];
		userInput = '';
		await scrollToBottom();

		try {
			await new Promise((resolve) => setTimeout(resolve, 2000));

			const response = await fetch(BACKEND_URL, {
				method: 'POST',
				headers: { 'Content-Type': 'application/json' },
				body: JSON.stringify({ query: userMessageText })
			});
			if (!response.ok) throw new Error('Network response failed.');
			const data = await response.json();
			messages = [...messages, { id: Date.now() + 1, sender: 'agent', text: data.agent_response }];
		} catch (error) {
			messages = [
				...messages,
				{
					id: Date.now() + 1,
					sender: 'agent',
					text: 'An error occurred. Please ensure the backend is running and try again.'
				}
			];
			console.error('Fetch error:', error);
		} finally {
			isLoading = false;
			await scrollToBottom();
		}
	}
</script>

<svelte:head>
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="anonymous" />
	<link
		href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600&display=swap"
		rel="stylesheet"
	/>
</svelte:head>

<main class="flex flex-col h-screen w-full max-w-3xl mx-auto bg-[#111113] text-[#F0F0F0]">
	<header class="py-5 px-6 flex-shrink-0 text-left">
		<h1 class="text-lg font-medium tracking-wide">CoinFantasy</h1>
	</header>

	<div class="flex-grow p-4 sm:p-6 overflow-y-auto flex flex-col gap-4" bind:this={chatContainer}>
		{#each messages as message (message.id)}
			<div
				class="py-2 px-4 rounded-2xl max-w-[85%] leading-relaxed text-base font-normal
                       {message.sender === 'user'
					? 'bg-[#9444FF] text-white self-end rounded-br-md'
					: 'bg-[#2A2B30] self-start rounded-bl-md'}"
				transition:slide|local={{ duration: 250 }}
			>
				{message.text}
			</div>
		{/each}

		<!-- MODIFIED SKELETON BLOCK -->
		{#if isLoading}
			<div
				class="py-3 px-4 rounded-2xl w-full bg-[#2A2B30]"
				transition:slide|local={{ duration: 250 }}
			>
				<div class="animate-pulse flex space-x-4">
					<div class="flex-1 space-y-3 py-1">
						<div class="h-2 bg-slate-700 rounded"></div>
						<div class="space-y-2">
							<div class="grid grid-cols-3 gap-4">
								<div class="h-2 bg-slate-700 rounded col-span-2"></div>
								<div class="h-2 bg-slate-700 rounded col-span-1"></div>
							</div>
							<div class="h-2 bg-slate-700 rounded"></div>
						</div>
					</div>
				</div>
			</div>
		{/if}
	</div>

	<footer class="p-4 sm:p-6 bg-[#111113]">
		<form
			class="flex gap-3 bg-[#1C1D21] rounded-xl p-2 border border-[#2A2B30]"
			onsubmit={(e) => {
				e.preventDefault();
				handleSubmit();
			}}
		>
			<Input
				type="text"
				bind:value={userInput}
				placeholder="Ask the agent a question..."
				class="flex-grow bg-transparent border-none text-base text-[#F0F0F0] placeholder:text-[#888888] focus:ring-0"
				autocomplete="off"
				disabled={isLoading}
			/>
			<Button
				type="submit"
				disabled={isLoading}
				class="bg-[#9444FF] text-white rounded-lg text-base font-medium hover:bg-[#833add]"
				>Send</Button
			>
		</form>
	</footer>
</main>
