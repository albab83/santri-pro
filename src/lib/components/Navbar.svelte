<script lang="ts">
	import { onMount } from 'svelte';
	import LogoutConfirm from '$lib/components/LogoutConfirm.svelte';
	import { fade, slide } from 'svelte/transition';

	let isLoggedIn = false;
	let role = '';
	let showLogoutModal = false;
	let mobileMenuOpen = false;

	function openLogoutModal() {
		showLogoutModal = true;
	}

	function closeLogoutModal() {
		showLogoutModal = false;
	}

	function toggleMobileMenu() {
		mobileMenuOpen = !mobileMenuOpen;
	}

	onMount(() => {
		const token = localStorage.getItem('token');
		const storedRole = localStorage.getItem('role');
		isLoggedIn = !!token;
		role = storedRole || '';
	});
</script>

{#if isLoggedIn}
	<nav class="sticky top-0 z-50 bg-white shadow-md">
		<div class="max-w-7xl mx-auto px-4 py-3 flex justify-between items-center">
			<div class="text-xl font-bold text-green-600">SantriPro</div>

			<!-- Tombol hamburger di mobile -->
			<button aria-label="Buka menu" class="md:hidden text-gray-700" on:click={toggleMobileMenu}>
				<svg
					xmlns="http://www.w3.org/2000/svg"
					class="h-6 w-6"
					fill="none"
					viewBox="0 0 24 24"
					stroke="currentColor"
					stroke-width="2"
				>
					<path stroke-linecap="round" stroke-linejoin="round" d="M4 6h16M4 12h16M4 18h16" />
				</svg>
			</button>

			<!-- Menu Desktop -->
			<ul class="hidden md:flex items-center space-x-6 text-gray-700 font-medium">
				{#if role === 'admin'}
					<li><a href="/admin" class="hover:text-green-600">Dashboard Admin</a></li>
               <li><a href="/project" class="hover:text-green-600">Project</a></li>
               {:else if role === 'santri'}
					<li><a href="/santri" class="hover:text-blue-600">Beranda</a></li>
               <li><a href="/santri" class="hover:text-green-600">Project</a></li>
				{/if}
				<!-- <li><a href="/jurnal" class="hover:text-green-600">Jurnal</a></li> -->
				{#if role === 'admin'}
					<li><a href="/admin" class="hover:text-green-600">Admin Panel</a></li>
				{/if}
				<li>
					<button
						on:click={openLogoutModal}
						class="bg-red-500 hover:bg-red-600 text-white px-4 py-2 rounded-lg"
					>
						Logout
					</button>
				</li>
			</ul>
		</div>

		<!-- Menu Mobile -->
		{#if mobileMenuOpen}
			<div class="md:hidden px-4 pb-4" transition:slide>
				<ul class="flex flex-col space-y-3 text-gray-700 font-medium">
					{#if role === 'admin'}
						<li><a href="/admin" class="hover:text-green-600">Dashboard Admin</a></li>
					{:else if role === 'santri'}
						<li><a href="/santri" class="hover:text-blue-600">Beranda</a></li>
					{/if}
					<li><a href="/santri" class="hover:text-green-600">Project</a></li>
					<!-- <li><a href="/jurnal" class="hover:text-green-600">Jurnal</a></li> -->
					{#if role === 'admin'}
						<li><a href="/admin" class="hover:text-green-600">Admin Panel</a></li>
					{/if}
					<li>
						<button
							on:click={() => {
								openLogoutModal();
								mobileMenuOpen = false;
							}}
							class="bg-red-500 hover:bg-red-600 text-white px-4 py-2 rounded-lg w-full text-center"
						>
							Logout
						</button>
					</li>
				</ul>
			</div>
		{/if}
	</nav>

	<LogoutConfirm visible={showLogoutModal} on:cancel={closeLogoutModal} />
{/if}
