<script>
	import { onMount } from 'svelte';
	import { getUser } from '$lib/api';
	import { supabaseClient } from '$lib/db';
	import { channelMessages } from '$lib/stores/channel-store';
	import Message from './Message.svelte';
	import MessageInput from './MessageInput.svelte';
	import Widget from '$lib/components/Widget.svelte';
	import WarningAltFilled from 'carbon-icons-svelte/lib/WarningAltFilled.svelte';

	/** Setup realtime */
	onMount(() => {
		supabaseClient
			.channel(`public:messages`)
			.on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'messages' }, (payload) =>
				onInsertMessage(payload.new)
			)
			.subscribe();
	});

	/** Component methods */
	async function onInsertMessage(message) {
		if (message) {
			let author = await getUser(supabaseClient, message.user_id);
			message.author = author;
			$channelMessages = [message, ...$channelMessages];
		}
	}

	$: loading = !Array.isArray($channelMessages);
</script>

<Widget
	header="Messages"
	class="col-span-full sm:col-span-6 lg:col-span-4 row-span-4 overflow-hidden"
	bodyClass="relative overflow-y-auto"
	{loading}
>
	{#if $channelMessages}
		{#if $channelMessages.length === 0}
			<MessageInput />
			<div class="absolute left-0 right-0 top-10 bottom-0 flex p-4 items-center justify-center">
				<div class="flex flex-col gap-2 items-center">
					<WarningAltFilled size={32} class="block text-amber-500" />
					<h4>No messages could be loaded</h4>
				</div>
			</div>
		{:else}
			<MessageInput />
			<div class="mt-10 space-y-6">
				{#each $channelMessages as message, i (message.id)}
					<Message
						author={message?.author}
						message={message?.message}
						timestamp={message?.inserted_at}
					/>
				{/each}
				<pre class="w-full min-h-[1px] bg-base-900" />
			</div>
		{/if}
	{:else}
		<div class="relative flex-auto min-h-[32rem]" />
	{/if}
</Widget>
