<script lang="ts">
	import { onMount } from 'svelte';

	let email = '';
	let message = '';
	let error = '';
	let loading = false;

	const baseUrl = import.meta.env.VITE_API_BASE_URL;

	async function handleSubmit() {
		error = '';
		message = '';
		loading = true;

		try {
			const res = await fetch(`${baseUrl}/api/auth/lupa-password`, {
				method: 'POST',
				headers: { 'Content-Type': 'application/json' },
				body: JSON.stringify({ email })
			});

			const data = await res.json();

			if (res.ok) {
				message = data.message;
			} else {
				error = data.message || 'Terjadi kesalahan';
			}
		} catch (err) {
			error = 'Gagal terhubung ke server';
		} finally {
			loading = false;
		}
	}

	onMount(() => {
		email = '';
		message = '';
		error = '';
	});
</script>

<div class="max-w-md mx-auto mt-10 p-6 bg-white rounded-lg shadow-md">
	<h1 class="text-2xl font-bold mb-6 text-gray-800 text-center">Lupa Password</h1>

	{#if message}
		<div class="mb-4 p-3 bg-green-50 border border-green-200 rounded-md">
			<p class="text-green-700 text-sm">{message}</p>
		</div>
	{/if}

	{#if error}
		<div class="mb-4 p-3 bg-red-50 border border-red-200 rounded-md">
			<p class="text-red-700 text-sm">{error}</p>
		</div>
	{/if}

	<form on:submit|preventDefault={handleSubmit} class="space-y-4">
		<div>
			<label for="email" class="block text-sm font-medium text-gray-700 mb-2">
				Email Address
			</label>
			<input
				id="email"
				type="email"
				bind:value={email}
				placeholder="Masukkan email Anda"
				class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition-colors duration-200"
				required
				disabled={loading}
			/>
		</div>

		<button
			type="submit"
			class="w-full bg-blue-600 hover:bg-blue-700 disabled:bg-gray-400 text-white font-medium py-2 px-4 rounded-md transition-colors duration-200 flex justify-center items-center"
			disabled={loading}
		>
			{#if loading}
				<svg
					class="animate-spin h-5 w-5 mr-2 text-white"
					xmlns="http://www.w3.org/2000/svg"
					fill="none"
					viewBox="0 0 24 24"
				>
					<circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"
					></circle>
					<path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v4a4 4 0 00-4 4H4z"
					></path>
				</svg>
				Mengirim...
			{:else}
				Kirim Link Reset
			{/if}
		</button>
	</form>
</div>
