<script>
	import { goto } from '$app/navigation';
	let email = '';
	let password = '';
	let message = '';
	let isLoading = false;
	let showPassword = false;
	const baseUrl = import.meta.env.VITE_API_BASE_URL;

	async function handleLogin() {
		message = '';
		isLoading = true;

		try {
			const res = await fetch(`${baseUrl}/api/auth/login`, {
				method: 'POST',
				headers: { 'Content-Type': 'application/json' },
				body: JSON.stringify({ email, password })
			});
			const data = await res.json();

			if (res.ok) {
				localStorage.setItem('token', data.token);
				localStorage.setItem('user', JSON.stringify(data.user));
				localStorage.setItem('role', data.user.role);
				message = 'Login berhasil!';
				setTimeout(() => {
					if (data.user && data.user.role === 'admin') {
						goto('/admin');
					} else if (data.user && data.user.role === 'santri') {
						goto('/santri');
					} else {
						goto('/'); // fallback jika role tidak dikenali
					}
				}, 1000);
			} else {
				message = data.message;
			}
		} catch (err) {
			message = 'Gagal terhubung ke server';
		} finally {
			isLoading = false;
		}
	}

	function togglePasswordVisibility() {
		showPassword = !showPassword;
	}
</script>

<div
	class="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 flex items-center justify-center p-4"
>
	<div class="w-full max-w-md">
		<!-- Login Card -->
		<div class="bg-white rounded-2xl shadow-xl p-8 space-y-6">
			<!-- Header -->
			<div class="text-center space-y-2">
				<div
					class="w-16 h-16 bg-blue-600 rounded-full flex items-center justify-center mx-auto mb-4"
				>
					<svg class="w-8 h-8 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
						<path
							stroke-linecap="round"
							stroke-linejoin="round"
							stroke-width="2"
							d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z"
						/>
					</svg>
				</div>
				<h2 class="text-2xl font-bold text-gray-900">Selamat Datang</h2>
				<p class="text-gray-600">Silakan masuk ke akun Anda</p>
			</div>

			<!-- Form -->
			<form on:submit|preventDefault={handleLogin} class="space-y-5">
				<!-- Email Field -->
				<div class="space-y-2">
					<label for="email" class="block text-sm font-medium text-gray-700"> Email </label>
					<div class="relative">
						<div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
							<svg
								class="h-5 w-5 text-gray-400"
								fill="none"
								stroke="currentColor"
								viewBox="0 0 24 24"
							>
								<path
									stroke-linecap="round"
									stroke-linejoin="round"
									stroke-width="2"
									d="M16 12a4 4 0 10-8 0 4 4 0 008 0zm0 0v1.5a2.5 2.5 0 005 0V12a9 9 0 10-9 9m4.5-1.206a8.959 8.959 0 01-4.5 1.207"
								/>
							</svg>
						</div>
						<input
							id="email"
							type="email"
							bind:value={email}
							placeholder="nama@email.com"
							required
							disabled={isLoading}
							class="w-full pl-10 pr-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition-colors disabled:bg-gray-50 disabled:cursor-not-allowed"
						/>
					</div>
				</div>

				<!-- Password Field -->
				<div class="space-y-2">
					<label for="password" class="block text-sm font-medium text-gray-700"> Password </label>
					<div class="relative">
						<div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
							<svg
								class="h-5 w-5 text-gray-400"
								fill="none"
								stroke="currentColor"
								viewBox="0 0 24 24"
							>
								<path
									stroke-linecap="round"
									stroke-linejoin="round"
									stroke-width="2"
									d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z"
								/>
							</svg>
						</div>
						<input
							id="password"
							type={showPassword ? 'text' : 'password'}
							bind:value={password}
							placeholder="Masukkan password"
							required
							disabled={isLoading}
							class="w-full pl-10 pr-12 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition-colors disabled:bg-gray-50 disabled:cursor-not-allowed"
						/>
						<button
							type="button"
							on:click={togglePasswordVisibility}
							disabled={isLoading}
							class="absolute inset-y-0 right-0 pr-3 flex items-center hover:text-gray-600 disabled:cursor-not-allowed"
						>
							{#if showPassword}
								<svg
									class="h-5 w-5 text-gray-400"
									fill="none"
									stroke="currentColor"
									viewBox="0 0 24 24"
								>
									<path
										stroke-linecap="round"
										stroke-linejoin="round"
										stroke-width="2"
										d="M13.875 18.825A10.05 10.05 0 0112 19c-4.478 0-8.268-2.943-9.543-7a9.97 9.97 0 011.563-3.029m5.858.908a3 3 0 114.243 4.243M9.878 9.878l4.242 4.242M9.878 9.878L3 3m6.878 6.878L21 21"
									/>
								</svg>
							{:else}
								<svg
									class="h-5 w-5 text-gray-400"
									fill="none"
									stroke="currentColor"
									viewBox="0 0 24 24"
								>
									<path
										stroke-linecap="round"
										stroke-linejoin="round"
										stroke-width="2"
										d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"
									/>
									<path
										stroke-linecap="round"
										stroke-linejoin="round"
										stroke-width="2"
										d="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z"
									/>
								</svg>
							{/if}
						</button>
					</div>
				</div>

				<!-- Submit Button -->
				<button
					type="submit"
					disabled={isLoading}
					class="w-full bg-blue-600 hover:bg-blue-700 disabled:bg-blue-400 text-white font-medium py-3 px-4 rounded-lg transition-colors duration-200 flex items-center justify-center space-x-2 disabled:cursor-not-allowed"
				>
					{#if isLoading}
						<svg
							class="animate-spin h-5 w-5 text-white"
							xmlns="http://www.w3.org/2000/svg"
							fill="none"
							viewBox="0 0 24 24"
						>
							<circle
								class="opacity-25"
								cx="12"
								cy="12"
								r="10"
								stroke="currentColor"
								stroke-width="4"
							></circle>
							<path
								class="opacity-75"
								fill="currentColor"
								d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
							></path>
						</svg>
						<span>Sedang masuk...</span>
					{:else}
						<span>Masuk</span>
					{/if}
				</button>
			</form>

			<!-- Message Display -->
			{#if message}
				<div
					class="rounded-lg p-4 {message === 'Login berhasil!'
						? 'bg-green-50 border border-green-200'
						: 'bg-red-50 border border-red-200'}"
				>
					<div class="flex items-center">
						{#if message === 'Login berhasil!'}
							<svg
								class="h-5 w-5 text-green-400 mr-2"
								fill="none"
								stroke="currentColor"
								viewBox="0 0 24 24"
							>
								<path
									stroke-linecap="round"
									stroke-linejoin="round"
									stroke-width="2"
									d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"
								/>
							</svg>
							<p class="text-sm text-green-800">{message}</p>
						{:else}
							<svg
								class="h-5 w-5 text-red-400 mr-2"
								fill="none"
								stroke="currentColor"
								viewBox="0 0 24 24"
							>
								<path
									stroke-linecap="round"
									stroke-linejoin="round"
									stroke-width="2"
									d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
								/>
							</svg>
							<p class="text-sm text-red-800">{message}</p>
						{/if}
					</div>
				</div>
			{/if}

			<!-- Additional Links -->
			<div class="text-center space-y-4 pt-4 border-t border-gray-200">
				<a
					href="/lupa-password"
					class="text-sm text-blue-600 hover:text-blue-800 hover:underline"
				>
					Lupa password?
				</a>
				<div class="text-sm text-gray-600">
					Belum punya akun?
					<a href="/register" class="text-blue-600 hover:text-blue-800 font-medium hover:underline">
						Daftar disini
					</a>
				</div>
			</div>
		</div>

		<!-- Footer -->
		<div class="text-center mt-8 text-sm text-gray-500">
			© 2024 Sistem Santri. All rights reserved.
		</div>
	</div>
</div>
