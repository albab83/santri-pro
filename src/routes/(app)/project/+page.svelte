<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { io, Socket } from 'socket.io-client';

	let socket: Socket | null = null;
	let projects: any[] = [];
	let loading = true;
	let error = '';

	let filterNama = '';
	let filterStatus = '';
	let filterTanggal = '';
	let filterSantri = '';

	const statusList = [
		{ label: 'Semua', value: '' },
		{ label: 'Diterima', value: 'diterima' },
		{ label: 'Menunggu', value: 'menunggu' },
		{ label: 'Ditolak', value: 'ditolak' },
		{ label: 'Selesai', value: 'selesai' },
		{ label: 'Proses', value: 'proses' }
	];

	const baseUrl = import.meta.env.VITE_API_BASE_URL;
	let token: string | null = null;

	// Pagination
	let page = 1;
	let limit = 10;
	let total = 0;
	let totalPages = 1;

	async function fetchProjects() {
		loading = true;
		error = '';

		try {
			const params = new URLSearchParams();
			params.append('page', String(page));
			params.append('limit', String(limit));
			if (filterNama) params.append('nama', filterNama);
			if (filterStatus) params.append('status', filterStatus);
			if (filterSantri) params.append('santri', filterSantri);
			if (filterTanggal) params.append('tanggal', filterTanggal);
			//  params.append('tanggal_selesai', filterTanggal);

			const res = await fetch(`${baseUrl}/api/project/filter?${params.toString()}`, {
				headers: { Authorization: `Bearer ${token}` }
			});
			if (!res.ok) throw new Error('Gagal mengambil data project');
			const data = await res.json();
			projects = data.projects;
			total = data.total;
			totalPages = data.totalPages;
		} catch (err) {
			error = err instanceof Error ? err.message : String(err);
		} finally {
			loading = false;
		}
	}

	async function tandaiJurnalSudahDibaca(projectId: number) {
		try {
			const token = localStorage.getItem('token');
			const res = await fetch(`${baseUrl}/api/project/${projectId}/journal/read`, {
				method: 'PUT',
				headers: {
					Authorization: `Bearer ${token}`
				}
			});
			if (!res.ok) throw new Error('Gagal update status jurnal');
			console.log('Status jurnal diperbarui ke sudah_dibaca');
		} catch (e) {
			console.error(e);
		}
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

	function handleFilterChange() {
		page = 1;
		fetchProjects();
	}

	function handlePageChange(newPage: number) {
		if (newPage < 1 || newPage > totalPages) return;
		page = newPage;
		fetchProjects();
	}

	onMount(async () => {
		token = typeof window !== 'undefined' ? localStorage.getItem('token') : null;
		fetchProjects();

		socket = io(baseUrl, {
			auth: { token }
		});

		socket.on('connect', () => {
			console.log('Socket connected');
		});

		socket.on('journal_update', () => {
			fetchProjects();
		});

		socket.on('project_update', () => {
			fetchProjects();
		});

		socket.on('journal_read', () => {
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
				Filter dan cari project berdasarkan nama, tanggal, status, atau santri.
			</p>

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
						on:input={handleFilterChange}
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
						on:input={handleFilterChange}
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
						on:change={handleFilterChange}
					>
						{#each statusList as s}
							<option value={s.value}>{s.label}</option>
						{/each}
					</select>
				</div>
				<div>
					<label for="filter-santri" class="block text-sm font-medium text-gray-700 mb-1"
						>Nama Santri</label
					>
					<input
						id="filter-santri"
						type="text"
						placeholder="Cari nama santri..."
						class="w-full border border-gray-300 rounded-lg px-4 py-2 focus:ring-2 focus:ring-blue-500"
						bind:value={filterSantri}
						on:input={handleFilterChange}
					/>
				</div>
			</div>

			<!-- Table -->
			<div class="overflow-x-auto rounded-xl border border-gray-200 bg-white">
				{#if loading}
					<div class="p-8 text-center text-gray-500">Memuat data project...</div>
				{:else if error}
					<div class="p-8 text-center text-red-600">{error}</div>
				{:else if projects.length === 0}
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
							{#each projects as p}
								<tr class="hover:bg-blue-50 transition">
									<td class="px-4 py-3 font-medium text-gray-900 uppercase">{p.judul}</td>
									<td class="px-4 py-3 text-gray-700 capitalize">
										{p.User?.nama} <br />
										<span class="text-xs text-gray-500 capitalize">{p.User?.email}</span>
									</td>
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
														  {p.status.toLowerCase() === 'proses' ? 'bg-indigo-100 text-indigo-700' : ''}
											
                                            "
										>
											{p.status}
										</span>
									</td>
									<td class="px-4 py-3">
										<a
											on:click={() => tandaiJurnalSudahDibaca(p.id)}
											href={`/journal/${p.id}`}
											class="relative inline-block text-blue-600 hover:underline text-sm"
										>
											Lihat Jurnal
											{#if +p.jumlahJurnalBelumDibaca > 0}
												<span
													class="absolute -top-2 -right-5.5 bg-red-500 text-white text-xs font-bold rounded-full w-5 h-5 flex items-center justify-center"
													>{p.jumlahJurnalBelumDibaca}</span
												>
											{/if}
										</a>
									</td>
								</tr>
							{/each}
						</tbody>
					</table>
				{/if}
			</div>

			<!-- Pagination -->
			<div class="flex justify-between items-center mt-6">
				<button
					class="px-4 py-2 rounded bg-gray-100 hover:bg-gray-200 disabled:opacity-50"
					on:click={() => handlePageChange(page - 1)}
					disabled={page === 1}
				>
					Prev
				</button>
				<span>Halaman {page} dari {totalPages}</span>
				<button
					class="px-4 py-2 rounded bg-gray-100 hover:bg-gray-200 disabled:opacity-50"
					on:click={() => handlePageChange(page + 1)}
					disabled={page === totalPages}
				>
					Next
				</button>
			</div>
		</div>
	</div>
</div>
