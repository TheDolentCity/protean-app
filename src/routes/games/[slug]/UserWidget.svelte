<script>
	import { onMount } from 'svelte';
	import { supabaseClient } from '$lib/db';
	import { channelMessages, channelUsers, presence } from '$lib/stores/channel-store';
	import { user } from '$lib/stores/user-store';
	import Widget from '$lib/components/Widget.svelte';
	import UserPersona from './UserPersona.svelte';

	/** Setup presence */
	onMount(() => {
		$presence = supabaseClient.channel('public:users', {
			config: {
				presence: {
					key: $user.id
				}
			}
		});

		$presence.on('presence', { event: 'sync' }, () => {
			const state = $presence.presenceState();
			let users = [];
			Object.values(state).forEach((userArray) => {
				users = users.concat(userArray);
			});
			users = users.map((user) => user.user);
			$channelUsers = users;
			$channelMessages.forEach((message) => {
				$channelUsers.forEach((channelUser) => {
					if (message.author.id === channelUser.id) {
						message.author.username = channelUser.username;
						message.author.nickname = channelUser.nickname;
						message.author.color = channelUser.color;
					}
				});
			});
			$channelMessages = $channelMessages;
		});

		$presence.subscribe(async (status) => {
			if (status === 'SUBSCRIBED') {
				await $presence.track({
					user: $user
				});
			}
		});
	});
</script>

{#if $user && $channelUsers}
	<Widget
		header="Active users"
		class="col-span-full sm:col-span-6 lg:col-span-4 2xl:col-span-4 row-span-1"
	>
		<div class="flex flex-wrap w-full min-h-[4rem] p-3 gap-3">
			{#each $channelUsers as user}
				<UserPersona {user} />
			{/each}
		</div>
	</Widget>
{:else}
	<Widget
		header="Active users"
		class="col-span-full sm:col-span-6 lg:col-span-4 2xl:col-span-4 row-span-1"
		loading
	>
		<div class="relative flex-auto min-h-[4rem]" />
	</Widget>
{/if}
