<script>
	import { redirect } from '@sveltejs/kit';
	import { fly } from 'svelte/transition';
	import { supabaseClient } from '$lib/db';
	import { useOutsideClick } from '$lib/actions/use-outside-click';
	import { user } from '$lib/stores/user-store';
	import Persona from '$lib/components/Persona.svelte';
	import AccountPersonaForm from './AccountPersonaForm.svelte';
	import Logout from 'carbon-icons-svelte/lib/Logout.svelte';
	import Popover from '../lib/components/Popover.svelte';

	let open = false;

	function toggleOpen() {
		open = !open;
	}

	function close() {
		open = false;
	}

	async function signout() {
		await supabaseClient.auth.signOut();
		throw redirect(303, '/auth');
	}
</script>

{#if $user}
	<Popover bind:open position="right">
		<Persona slot="trigger" onclick={toggleOpen} letter={$user?.username[0]} color={$user?.color} />
		<svelte:fragment slot="content">
			<div class="-mx-6 mb-4 px-6 pb-4 border-b border-base-800">
				<div>Signed in as</div>
				<div class="font-medium text-focus">{$user?.username}</div>
			</div>
			<AccountPersonaForm onsubmit={close} />
			<div class="-mx-6 mt-4 -mb-4 border-t border-base-800">
				<button
					on:click={signout}
					class="flex w-full px-6 py-4 gap-2 items-center rounded-b-md font-semibold text-focus hover:outline hover:outline-1 hover:outline-base-700 hover:bg-base-800 focus:outline focus:outline-1 focus:outline-base-700 focus:bg-base-800"
				>
					<Logout size={20} class="p-1 rounded-full bg-base-600 text-white" />
					<span>Sign out</span>
				</button>
			</div>
		</svelte:fragment>
	</Popover>
{:else}
	<Persona loading />
{/if}
