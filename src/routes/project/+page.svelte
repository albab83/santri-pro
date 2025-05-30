<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { io, Socket } from 'socket.io-client';

	let socket: Socket | null = null;
	let projects: any[] = [];
	let filtered: any[] = [];
	let loading = true;
	let error = '';

	let filterNama = '';
	let filterStatus = '';
	let filterTanggal = '';

	const statusList = [
		{ label: 'Semua', value: '' },
		{ label: 'Diterima', value: 'diterima' },
		{ label: 'Menunggu', value: 'menunggu' },
		{ label: 'Ditolak', value: 'ditolak' },
		{ label: 'Selesai', value: 'selesai' }
	];

	const baseUrl = import.meta.env.VITE_API_BASE_URL;
	let token: string | null = null;

	async function fetchProjects() {
		loading = true;
		error = '';
		try {
			const res = await fetch(`${baseUrl}/api/project`, {
				headers: { Authorization: `Bearer ${token}` }
			});
			if (!res.ok) throw new Error('Gagal mengambil data project');
			projects = await res.json();
			filterProjects();
		} catch (err) {
			error = err instanceof Error ? err.message : String(err);
		} finally {
			loading = false;
		}
	}

	function filterProjects() {
		filtered = projects.filter((p) => {
			const namaMatch = p.judul.toLowerCase().includes(filterNama.toLowerCase());
			const statusMatch = filterStatus ? p.status.toLowerCase() === filterStatus : true;
			const tanggalMatch = filterTanggal
				? new Date(p.tanggal_pengajuan).toISOString().slice(0, 10) === filterTanggal
				: true;
			return namaMatch && statusMatch && tanggalMatch;
		});
	}

	function formatDate(dateString: string) {
		if (!dateString) return '-';
		const date = new Date(dateString);
		return date.toLocaleDateString('id-ID', {
			day: 'numeric',
			month: 'long',
			year: 'numeric'
		});
	}

	onMount(() => {
		token = typeof window !== 'undefined' ? localStorage.getItem('token') : null;
		fetchProjects();

		socket = io(baseUrl, {
			auth: { token }
		});

		socket.on('connect', () => {
			console.log('Socket connected');
		});

		socket.on('project_update', () => {
			fetchProjects();
		});

		socket.on('disconnect', () => {
			console.log('Socket disconnected');
		});
	});

	onDestroy(() => {
		if (socket) socket.disconnect();
	});
</script>

<div class="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 p-4">
	<div class="max-w-7xl mx-auto">
		<div class="bg-white rounded-2xl shadow-xl p-8 mb-8">
			<h1 class="text-3xl font-bold text-gray-900 mb-2">Daftar Semua Project</h1>
			<p class="text-gray-600 mb-6">
				Filter dan cari project berdasarkan nama, tanggal, atau status.
			</p>

			<!-- Filter Bar -->
			<!-- Filter Bar -->
			<div class="flex flex-col md:flex-row md:items-end gap-4 mb-6">
				<div class="flex-1">
					<label for="filter-nama" class="block text-sm font-medium text-gray-700 mb-1"
						>Nama Project</label
					>
					<input
						id="filter-nama"
						type="text"
						placeholder="Cari nama project..."
						class="w-full border border-gray-300 rounded-lg px-4 py-2 focus:ring-2 focus:ring-blue-500"
						bind:value={filterNama}
						on:input={filterProjects}
					/>
				</div>
				<div>
					<label for="filter-tanggal" class="block text-sm font-medium text-gray-700 mb-1"
						>Tanggal Pengajuan</label
					>
					<input
						id="filter-tanggal"
						type="date"
						class="border border-gray-300 rounded-lg px-4 py-2 focus:ring-2 focus:ring-blue-500"
						bind:value={filterTanggal}
						on:input={filterProjects}
					/>
				</div>
				<div>
					<label for="filter-status" class="block text-sm font-medium text-gray-700 mb-1"
						>Status</label
					>
					<select
						id="filter-status"
						class="border border-gray-300 rounded-lg px-4 py-2 focus:ring-2 focus:ring-blue-500"
						bind:value={filterStatus}
						on:change={filterProjects}
					>
						{#each statusList as s}
							<option value={s.value}>{s.label}</option>
						{/each}
					</select>
				</div>
			</div>

			<!-- Table -->
			<div class="overflow-x-auto rounded-xl border border-gray-200 bg-white">
				{#if loading}
					<div class="p-8 text-center text-gray-500">Memuat data project...</div>
				{:else if error}
					<div class="p-8 text-center text-red-600">{error}</div>
				{:else if filtered.length === 0}
					<div class="p-8 text-center text-gray-500">Tidak ada project ditemukan.</div>
				{:else}
					<table class="min-w-full divide-y divide-gray-200">
						<thead class="bg-gray-50">
							<tr>
								<th class="px-4 py-3 text-left text-xs font-semibold text-gray-700">Nama Project</th
								>
								<th class="px-4 py-3 text-left text-xs font-semibold text-gray-700">Pengaju</th>
								<th class="px-4 py-3 text-left text-xs font-semibold text-gray-700">Tanggal</th>
								<th class="px-4 py-3 text-left text-xs font-semibold text-gray-700">Status</th>
								<th class="px-4 py-3 text-left text-xs font-semibold text-gray-700">Aksi</th>
							</tr>
						</thead>
						<tbody class="divide-y divide-gray-100">
							{#each filtered as p}
								<tr class="hover:bg-blue-50 transition">
									<td class="px-4 py-3 font-medium text-gray-900">{p.judul}</td>
									<td class="px-4 py-3 text-gray-700"
										>{p.User?.nama} <br /><span class="text-xs text-gray-500">{p.User?.email}</span
										></td
									>
									<td class="px-4 py-3 text-gray-700">{formatDate(p.tanggal_pengajuan)}</td>
									<td class="px-4 py-3">
										<span
											class="inline-block px-3 py-1 rounded-full text-xs font-semibold
                                            {p.status.toLowerCase() === 'diterima'
												? 'bg-green-100 text-green-700'
												: ''}
                                            {p.status.toLowerCase() === 'menunggu'
												? 'bg-yellow-100 text-yellow-700'
												: ''}
                                            {p.status.toLowerCase() === 'ditolak'
												? 'bg-red-100 text-red-700'
												: ''}
                                            {p.status.toLowerCase() === 'selesai'
												? 'bg-emerald-100 text-emerald-700'
												: ''}
                                            "
										>
											{p.status}
										</span>
									</td>
									<td class="px-4 py-3">
										<!-- Aksi: detail, dsb -->
										<a href={`/journal/${p.id}`} class="text-blue-600 hover:underline text-sm"
											>Journal</a
										>
									</td>
								</tr>
							{/each}
						</tbody>
					</table>
				{/if}
			</div>
		</div>
	</div>
</div>
