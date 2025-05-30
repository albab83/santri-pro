<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { io, Socket } from 'socket.io-client';
	import { goto } from '$app/navigation';

	interface User {
		nama: string;
		email: string;
		role: string;
	}

	interface Project {
		id: number;
		judul: string;
		deskripsi: string;
		tujuan: string;
		status: string;
		alasan_penolakan?: string;
		User?: User;
		tanggal_pengajuan?: string;
	}

	let projects: Project[] = [];
	let token: string | null = null;
	let error: string = '';
	let alasan: { [key: number]: string } = {};
	let loadingId: number | null = null;
	let users: User[] = [];

	let socket: Socket | null = null;
	const baseUrl = import.meta.env.VITE_API_BASE_URL;

	async function fetchUsers() {
		if (!token) return;
		try {
			const res = await fetch(`${baseUrl}/api/users`, {
				headers: { Authorization: `Bearer ${token}` }
			});
			if (!res.ok) throw new Error('Gagal mengambil data user');
			users = await res.json();
		} catch (err) {
			error = err instanceof Error ? err.message : String(err);
			// Optional: handle error
		}
	}

	// Statistics computed from projects
	$: stats = {
		totalSantri: users.filter((u) => u.role === 'santri').length,
		totalProject: projects.length,
		projectDiterima: projects.filter(
			(p) => p.status.toLowerCase() === 'diterima' || p.status.toLowerCase() === 'approved'
		).length,
		projectMenunggu: projects.filter(
			(p) => p.status.toLowerCase() === 'menunggu' || p.status.toLowerCase() === 'pending'
		).length,
		projectDitolak: projects.filter(
			(p) => p.status.toLowerCase() === 'ditolak' || p.status.toLowerCase() === 'rejected'
		).length,
		projectSelesai: projects.filter(
			(p) => p.status.toLowerCase() === 'selesai' || p.status.toLowerCase() === 'completed'
		).length
	};

	// Chart data for simple pie chart
	$: chartData = [
		{ label: 'Diterima', value: stats.projectDiterima, color: '#10B981' },
		{ label: 'Menunggu', value: stats.projectMenunggu, color: '#F59E0B' },
		{ label: 'Ditolak', value: stats.projectDitolak, color: '#EF4444' },
		{ label: 'Selesai', value: stats.projectSelesai, color: '#14B8A6' }
	];

	function getStatusColor(status: string) {
		switch (status.toLowerCase()) {
			case 'diterima':
			case 'approved':
				return 'bg-green-100 text-green-800';
			case 'ditolak':
			case 'rejected':
				return 'bg-red-100 text-red-800';
			case 'menunggu':
			case 'pending':
				return 'bg-yellow-100 text-yellow-800';
			default:
				return 'bg-gray-100 text-gray-800';
		}
	}

	function formatDate(dateString?: string) {
		if (!dateString) return '-';
		const date = new Date(dateString);
		return date.toLocaleDateString('id-ID', {
			day: 'numeric',
			month: 'long',
			year: 'numeric'
		});
	}

	async function fetchProjects() {
		error = '';
		try {
			const res = await fetch(`${baseUrl}/api/project`, {
				headers: { Authorization: `Bearer ${token}` }
			});
			if (!res.ok) throw new Error('Gagal mengambil data project');
			projects = await res.json();
		} catch (err) {
			error = err instanceof globalThis.Error ? err.message : String(err);
		}
	}

	// Removed unused onDestroy block for ws (no ws variable defined)

	async function approve(id: number) {
		if (!token) return;
		loadingId = id;
		error = '';
		try {
			const res = await fetch(`${baseUrl}/api/project/${id}/approve`, {
				method: 'PUT',
				headers: { Authorization: `Bearer ${token}` }
			});
			if (!res.ok) throw new Error('Gagal menyetujui project');
			await fetchProjects();
		} catch (err) {
			error = err instanceof Error ? err.message : String(err);
		} finally {
			loadingId = null;
		}
	}

	async function reject(id: number) {
		if (!token) return;
		const reason = alasan[id];
		if (!reason) {
			error = 'Alasan penolakan wajib diisi';
			return;
		}
		loadingId = id;
		error = '';
		try {
			const res = await fetch(`${baseUrl}/api/project/${id}/reject`, {
				method: 'PUT',
				headers: {
					'Content-Type': 'application/json',
					Authorization: `Bearer ${token}`
				},
				body: JSON.stringify({ reason })
			});
			if (!res.ok) throw new Error('Gagal menolak project');
			alasan[id] = '';
			await fetchProjects();
		} catch (err) {
			error = err instanceof Error ? err.message : String(err);
		} finally {
			loadingId = null;
		}
	}

	onMount(() => {
		token = typeof window !== 'undefined' ? localStorage.getItem('token') : null;
		fetchProjects();
		fetchUsers();
		socket = io(baseUrl, {
			auth: { token }
		});

		socket.on('connect', () => {
			console.log('Socket connected');
		});

		socket.on('project_update', () => {
			fetchProjects();
		});

		socket.on('user_update', () => {
			fetchUsers();
		});

		socket.on('disconnect', () => {
			// console.log('Socket disconnected');
		});

		socket.on('connect_error', (error) => {
			// console.error('Socket connection error:', error);
		});
	});
	onDestroy(() => {
		if (socket) socket.disconnect();
	});
</script>

<div class="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 p-4">
	<div class="max-w-6xl mx-auto">
		<!-- Header -->
		<div class="bg-white rounded-2xl shadow-xl p-8 mb-8">
			<div class="flex items-center justify-between">
				<div class="flex items-center space-x-4">
					<div class="w-12 h-12 bg-blue-600 rounded-full flex items-center justify-center">
						<svg class="w-6 h-6 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"
							/>
						</svg>
					</div>
					<div>
						<h1 class="text-3xl font-bold text-gray-900">Persetujuan Project</h1>
						<p class="text-gray-600">Kelola pengajuan project dari anggota</p>
					</div>
				</div>
			</div>
		</div>

		<!-- Statistics Cards -->
		<div class="grid grid-cols-2 md:grid-cols-4 lg:grid-cols-5 gap-6 mb-8">
			<div class="bg-white rounded-xl shadow-lg p-6">
				<div class="flex items-center">
					<div class="p-2 bg-blue-100 rounded-lg">
						<svg
							class="w-6 h-6 text-blue-600"
							fill="none"
							stroke="currentColor"
							viewBox="0 0 24 24"
						>
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M12 4.354a4 4 0 110 5.292M15 21H3v-1a6 6 0 0112 0v1zm0 0h6v-1a6 6 0 00-9-5.197m13.5-9a2.5 2.5 0 11-5 0 2.5 2.5 0 015 0z"
							/>
						</svg>
					</div>
					<div class="ml-4">
						<p class="text-sm font-medium text-gray-600">Total Santri</p>
						<p class="text-2xl font-bold text-gray-900">{stats.totalSantri}</p>
					</div>
				</div>
			</div>

			<div class="bg-white rounded-xl shadow-lg p-6">
				<div class="flex items-center">
					<div class="p-2 bg-purple-100 rounded-lg">
						<svg
							class="w-6 h-6 text-purple-600"
							fill="none"
							stroke="currentColor"
							viewBox="0 0 24 24"
						>
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M19 11H5m14 0a2 2 0 012 2v6a2 2 0 01-2 2H5a2 2 0 01-2-2v-6a2 2 0 012-2m14 0V9a2 2 0 00-2-2M5 11V9a2 2 0 012-2m0 0V5a2 2 0 012-2h6a2 2 0 012 2v2M7 7h10"
							/>
						</svg>
					</div>
					<div class="ml-4">
						<p class="text-sm font-medium text-gray-600">Total Project</p>
						<p class="text-2xl font-bold text-gray-900">{stats.totalProject}</p>
					</div>
				</div>
			</div>

			<div class="bg-white rounded-xl shadow-lg p-6">
				<div class="flex items-center">
					<div class="p-2 bg-green-100 rounded-lg">
						<svg
							class="w-6 h-6 text-green-600"
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
					</div>
					<div class="ml-4">
						<p class="text-sm font-medium text-gray-600">Diterima</p>
						<p class="text-2xl font-bold text-gray-900">{stats.projectDiterima}</p>
					</div>
				</div>
			</div>

			<div class="bg-white rounded-xl shadow-lg p-6">
				<div class="flex items-center">
					<div class="p-2 bg-yellow-100 rounded-lg">
						<svg
							class="w-6 h-6 text-yellow-600"
							fill="none"
							stroke="currentColor"
							viewBox="0 0 24 24"
						>
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"
							/>
						</svg>
					</div>
					<div class="ml-4">
						<p class="text-sm font-medium text-gray-600">Menunggu</p>
						<p class="text-2xl font-bold text-gray-900">{stats.projectMenunggu}</p>
					</div>
				</div>
			</div>

			<div class="bg-white rounded-xl shadow-lg p-6">
				<div class="flex items-center">
					<div class="p-2 bg-red-100 rounded-lg">
						<svg class="w-6 h-6 text-red-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z"
							/>
						</svg>
					</div>
					<div class="ml-4">
						<p class="text-sm font-medium text-gray-600">Ditolak</p>
						<p class="text-2xl font-bold text-gray-900">{stats.projectDitolak}</p>
					</div>
				</div>
			</div>
			<div class="bg-white rounded-xl shadow-lg p-6">
				<div class="flex items-center">
					<div class="p-2 bg-emerald-100 rounded-lg">
						<svg
							class="w-6 h-6 text-emerald-600"
							fill="none"
							stroke="currentColor"
							viewBox="0 0 24 24"
						>
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M5 13l4 4L19 7"
							/>
						</svg>
					</div>
					<div class="ml-4">
						<p class="text-sm font-medium text-gray-600">Selesai</p>
						<p class="text-2xl font-bold text-gray-900">{stats.projectSelesai}</p>
					</div>
				</div>
			</div>
		</div>

		<!-- Chart Section -->
		{#if stats.totalProject > 0}
			<div class="bg-white rounded-2xl shadow-xl p-8 mb-8">
				<h2 class="text-2xl font-bold text-gray-900 mb-6">Status Project Overview</h2>
				<div
					class="flex flex-col lg:flex-row items-center justify-center space-y-6 lg:space-y-0 lg:space-x-12"
				>
					<!-- Simple Pie Chart using CSS -->
					<div class="relative">
						<svg width="200" height="200" viewBox="0 0 200 200" class="transform -rotate-90">
							{#each chartData as item, i}
								{@const total = stats.totalProject}
								{@const percentage = total > 0 ? (item.value / total) * 100 : 0}
								{@const circumference = 2 * Math.PI * 80}
								{@const offset = circumference - (percentage / 100) * circumference}
								{@const prevPercentages = chartData
									.slice(0, i)
									.reduce((sum, prev) => sum + (prev.value / total) * 100, 0)}
								{@const rotation = (prevPercentages / 100) * 360}

								{#if item.value > 0}
									<circle
										cx="100"
										cy="100"
										r="80"
										fill="none"
										stroke={item.color}
										stroke-width="20"
										stroke-dasharray={circumference}
										stroke-dashoffset={offset}
										transform="rotate({rotation} 100 100)"
										opacity="0.8"
									/>
								{/if}
							{/each}
						</svg>

						<div class="absolute inset-0 flex items-center justify-center">
							<div class="text-center">
								<div class="text-3xl font-bold text-gray-900">{stats.totalProject}</div>
								<div class="text-sm text-gray-600">Total</div>
							</div>
						</div>
					</div>

					<!-- Legend -->
					<div class="space-y-4">
						{#each chartData as item}
							<div class="flex items-center space-x-3">
								<div class="w-4 h-4 rounded-full" style="background-color: {item.color}"></div>
								<span class="text-gray-700 font-medium">{item.label}</span>
								<span class="text-gray-600">({item.value})</span>
								<span class="text-sm text-gray-500">
									{stats.totalProject > 0
										? Math.round((item.value / stats.totalProject) * 100)
										: 0}%
								</span>
							</div>
						{/each}
					</div>
				</div>
			</div>
		{/if}

		<div class="mb-8">
			<button
				on:click={() => goto('/project')}
				class="w-full md:w-auto flex items-center justify-center space-x-3 bg-gradient-to-r from-indigo-500 to-blue-500 hover:from-indigo-600 hover:to-blue-600 text-white font-semibold px-6 py-4 rounded-2xl shadow-lg transition-all duration-200"
			>
				<svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						stroke-width="2"
						d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"
					/>
				</svg>
				<span class="text-lg">Lihat Semua Project</span>
			</button>
		</div>

		<!-- Project List -->
		<div class="bg-white rounded-2xl shadow-xl p-8">
			<div class="flex items-center space-x-3 mb-6">
				<svg class="w-6 h-6 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						stroke-width="2"
						d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"
					/>
				</svg>
				<h2 class="text-2xl font-bold text-gray-900">Project Menunggu Persetujuan</h2>
			</div>

			{#if error}
				<div class="rounded-lg p-4 bg-red-50 border border-red-200 mb-6">
					<div class="flex items-center">
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
						<p class="text-sm text-red-800">{error}</p>
					</div>
				</div>
			{/if}

			{#if projects.filter((p) => p.status === 'menunggu').length === 0}
				<div class="text-center py-12">
					<svg
						class="w-16 h-16 text-gray-400 mx-auto mb-4"
						fill="none"
						stroke="currentColor"
						viewBox="0 0 24 24"
					>
						<path
							stroke-linecap="round"
							stroke-linejoin="round"
							stroke-width="2"
							d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"
						/>
					</svg>
					<h3 class="text-lg font-medium text-gray-900 mb-2">
						Tidak ada project yang menunggu persetujuan
					</h3>
					<p class="text-gray-600">Semua project telah diproses</p>
				</div>
			{:else}
				<div class="space-y-6">
					{#each projects.filter((p) => p.status === 'menunggu') as p}
						<div
							class="bg-gradient-to-br from-white to-gray-50 border border-gray-200 rounded-xl p-6 hover:shadow-lg transition-shadow duration-200"
						>
							<div class="flex items-start justify-between mb-4">
								<div class="flex-1">
									<h3 class="text-lg font-semibold text-gray-900 mb-2">{p.judul}</h3>
									<div class="flex items-center space-x-3">
										<span
											class="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium {getStatusColor(
												p.status
											)}"
										>
											{p.status}
										</span>
										<span class="text-xs text-gray-500">
											Diajukan oleh: {p.User?.nama} ({p.User?.email})
										</span>
									</div>
								</div>
								<svg
									class="w-6 h-6 text-gray-400"
									fill="none"
									stroke="currentColor"
									viewBox="0 0 24 24"
								>
									<path
										stroke-linecap="round"
										stroke-linejoin="round"
										stroke-width="2"
										d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"
									/>
								</svg>
							</div>

							<div class="space-y-4">
								<div>
									<p class="text-sm font-medium text-gray-700 mb-1">Deskripsi:</p>
									<p class="text-sm text-gray-600">{p.deskripsi}</p>
								</div>

								<div>
									<p class="text-sm font-medium text-gray-700 mb-1">Tujuan:</p>
									<p class="text-sm text-gray-600">{p.tujuan}</p>
								</div>

								<div class="text-xs text-gray-500 flex items-center">
									<svg class="w-4 h-4 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24">
										<path
											stroke-linecap="round"
											stroke-linejoin="round"
											stroke-width="2"
											d="M8 7V3a4 4 0 118 0v4m-4 12v-2m-4 2h8a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2z"
										/>
									</svg>
									Diajukan pada {formatDate(p.tanggal_pengajuan)}
								</div>

								<div class="pt-4 border-t border-gray-200">
									<textarea
										bind:value={alasan[p.id]}
										placeholder="Masukkan alasan penolakan (wajib diisi jika menolak)"
										class="w-full border border-gray-300 rounded-lg p-3 text-sm focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition-colors disabled:bg-gray-50 disabled:cursor-not-allowed resize-none"
										rows="2"
										disabled={loadingId === p.id}
									></textarea>

									<div class="flex justify-end space-x-3 mt-3">
										<button
											on:click={() => reject(p.id)}
											disabled={loadingId === p.id}
											class="bg-red-600 hover:bg-red-700 disabled:bg-red-400 text-white font-medium py-2 px-5 rounded-lg transition-colors duration-200 flex items-center space-x-2 disabled:cursor-not-allowed"
										>
											{#if loadingId === p.id}
												<svg
													class="animate-spin h-4 w-4 text-white"
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
												<span>Memproses...</span>
											{:else}
												<svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
													<path
														stroke-linecap="round"
														stroke-linejoin="round"
														stroke-width="2"
														d="M6 18L18 6M6 6l12 12"
													/>
												</svg>
												<span>Tolak</span>
											{/if}
										</button>

										<button
											on:click={() => approve(p.id)}
											disabled={loadingId === p.id}
											class="bg-green-600 hover:bg-green-700 disabled:bg-green-400 text-white font-medium py-2 px-5 rounded-lg transition-colors duration-200 flex items-center space-x-2 disabled:cursor-not-allowed"
										>
											{#if loadingId === p.id}
												<svg
													class="animate-spin h-4 w-4 text-white"
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
												<span>Memproses...</span>
											{:else}
												<svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
													<path
														stroke-linecap="round"
														stroke-linejoin="round"
														stroke-width="2"
														d="M5 13l4 4L19 7"
													/>
												</svg>
												<span>Setujui</span>
											{/if}
										</button>
									</div>
								</div>
							</div>
						</div>
					{/each}
				</div>
			{/if}
		</div>
	</div>
</div>
