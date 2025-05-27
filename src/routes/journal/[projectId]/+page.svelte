<script lang="ts">
	import { page } from '$app/stores';
	import { onMount } from 'svelte';
	import { browser } from '$app/environment';

	interface Journal {
		id: number | string;
		created_at: string;
		isi: string;
		status?: string;
		// Tambahkan field lain jika perlu
	}

	let journals: Journal[] = [];
	let isi = '';
	let status = '';
	let message = '';
	let error = '';
	let loading = false;
	let projectId = '';
	let projectName = '';
	let editId: string | number | null = null;
	let formEl: HTMLFormElement | null = null; // referensi DOM form

	const baseUrl = import.meta.env.VITE_API_BASE_URL || 'http://localhost:5000';

	onMount(async () => {
		if (!browser) return;
		projectId = $page.params.projectId;
		await loadJournals();
	});

	function scrollToForm() {
		formEl?.scrollIntoView({ behavior: 'smooth', block: 'start' });
	}

	function startEdit(journal: Journal) {
		isi = journal.isi;
		status = journal.status || '';
		editId = journal.id;
		scrollToForm();
	}

	async function loadJournals() {
		loading = true;
		error = '';
		try {
			const token = localStorage.getItem('token');
			const res = await fetch(`${baseUrl}/api/journal/${projectId}`, {
				headers: {
					Authorization: `Bearer ${token}`
				}
			});
			const data = await res.json();
			if (res.ok) {
				projectName = data.projectTitle || '';
				journals = data.journals || [];
			} else {
				error = data.message || 'Gagal memuat jurnal';
			}
		} catch (e) {
			error = 'Terjadi kesalahan saat memuat jurnal';
		} finally {
			loading = false;
		}
	}

	async function kirimJurnal(event?: Event) {
		if (event) event.preventDefault();
		message = '';
		error = '';
		try {
			const token = localStorage.getItem('token');
			const method = editId ? 'PUT' : 'POST';
			const endpoint = editId
				? `${baseUrl}/api/journal/${editId}`
				: `${baseUrl}/api/journal/${projectId}`;

			const res = await fetch(endpoint, {
				method,
				headers: {
					'Content-Type': 'application/json',
					Authorization: `Bearer ${token}`
				},
				body: JSON.stringify({ isi, status })
			});
			let data: any = {};
			const contentType = res.headers.get('Content-Type');
			if (contentType && contentType.includes('application/json')) {
				data = await res.json();
			} else {
				data = { message: 'Response tidak dalam format JSON' };
			}
			if (res.ok) {
				message = editId ? 'Jurnal berhasil diperbarui' : 'Jurnal berhasil dikirim';
				isi = '';
				status = '';
				editId = null;
				await loadJournals();
			} else {
				error = data.message || 'Gagal menyimpan jurnal';
			}
		} catch (e) {
			error = 'Terjadi kesalahan saat menyimpan jurnal';
		}
	}

	function formatDate(dateString: string) {
		const date = new Date(dateString);
		return date.toLocaleDateString('id-ID', {
			day: 'numeric',
			month: 'long',
			year: 'numeric',
			hour: '2-digit',
			minute: '2-digit'
		});
	}

	function getStatusColor(status?: string) {
		if (!status) return '';
		switch (status.toLowerCase()) {
			case 'selesai':
			case 'completed':
				return 'bg-green-100 text-green-800';
			case 'progress':
			case 'berlangsung':
				return 'bg-blue-100 text-blue-800';
			case 'pending':
			case 'menunggu':
				return 'bg-yellow-100 text-yellow-800';
			case 'blocked':
			case 'terhambat':
				return 'bg-red-100 text-red-800';
			default:
				return 'bg-gray-100 text-gray-800';
		}
	}
</script>

<div class="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 p-4">
	<div class="max-w-4xl mx-auto">
		<!-- Header -->
		<div class="bg-white rounded-2xl shadow-xl p-8 mb-8">
			<div class="flex items-center space-x-4">
				<div class="w-12 h-12 bg-green-600 rounded-full flex items-center justify-center">
					<svg class="w-6 h-6 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
						<path
							stroke-linecap="round"
							stroke-linejoin="round"
							stroke-width="2"
							d="M12 6.253v13m0-13C10.832 5.477 9.246 5 7.5 5S4.168 5.477 3 6.253v13C4.168 18.477 5.754 18 7.5 18s3.332.477 4.5 1.253m0-13C13.168 5.477 14.754 5 16.5 5c1.746 0 3.332.477 4.5 1.253v13C19.832 18.477 18.246 18 16.5 18c-1.746 0-3.332.477-4.5 1.253"
						/>
					</svg>
				</div>
				<div>
					<h1 class="text-3xl font-bold text-gray-900">Jurnal Project</h1>
					<p class="text-lg text-gray-600">{projectName || 'Loading...'}</p>
				</div>
			</div>
		</div>

		<!-- Form Jurnal -->
		<div class="bg-white rounded-2xl shadow-xl p-8 mb-8">
			<div class="flex items-center space-x-3 mb-6">
				<svg class="w-6 h-6 text-green-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						stroke-width="2"
						d="M15.232 5.232l3.536 3.536m-2.036-5.036a2.5 2.5 0 113.536 3.536L6.5 21.036H3v-3.572L16.732 3.732z"
					/>
				</svg>
				<h2 class="text-2xl font-bold text-gray-900">Tulis Jurnal Baru</h2>
			</div>

			<form onsubmit={kirimJurnal} bind:this={formEl} class="space-y-6">
				<div>
					<label for="isi" class="block text-sm font-medium text-gray-700 mb-2">Isi Jurnal</label>
					<textarea
						class="w-full p-4 border border-gray-300 rounded-lg focus:ring-2 focus:ring-green-500 focus:border-green-500 transition-colors resize-none"
						rows="6"
						placeholder="Ceritakan progress, tantangan, atau pencapaian hari ini..."
						bind:value={isi}
						required
					></textarea>
				</div>

				<div>
					<label for="status" class="block text-sm font-medium text-gray-700 mb-2"
						>Status (Opsional)</label
					>
					<select
						class="w-full p-4 border border-gray-300 rounded-lg focus:ring-2 focus:ring-green-500 focus:border-green-500 transition-colors"
						bind:value={status}
					>
						<option value="">Pilih status...</option>
						<option value="berlangsung">Berlangsung</option>
						<option value="selesai">Selesai</option>
						<option value="menunggu">Menunggu</option>
						<option value="terhambat">Terhambat</option>
					</select>
				</div>

				<div class="flex items-center space-x-4">
					<button
						type="submit"
						disabled={loading}
						class="bg-green-600 hover:bg-green-700 disabled:bg-green-400 text-white font-medium py-3 px-6 rounded-lg transition-colors duration-200 flex items-center space-x-2 disabled:cursor-not-allowed"
					>
						{#if loading}
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
							<span>Mengirim...</span>
						{:else}
							<svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
								<path
									stroke-linecap="round"
									stroke-linejoin="round"
									stroke-width="2"
									d="M12 19l9 2-9-18-9 18 9-2zm0 0v-8"
								/>
							</svg>
							<span> {editId ? 'Update Jurnal' : 'Kirim Jurnal'}</span>
						{/if}
					</button>
				</div>

				{#if message}
					<div class="rounded-lg p-4 bg-green-50 border border-green-200">
						<div class="flex items-center">
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
						</div>
					</div>
				{/if}

				{#if error}
					<div class="rounded-lg p-4 bg-red-50 border border-red-200">
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
			</form>
		</div>

		<!-- Riwayat Jurnal -->
		<div class="bg-white rounded-2xl shadow-xl p-8">
			<div class="flex items-center space-x-3 mb-6">
				<svg class="w-6 h-6 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						stroke-width="2"
						d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"
					/>
				</svg>
				<h2 class="text-2xl font-bold text-gray-900">Riwayat Jurnal</h2>
				<span class="bg-blue-100 text-blue-800 text-sm font-medium px-2.5 py-0.5 rounded-full">
					{journals.length} entri
				</span>
			</div>

			{#if loading}
				<div class="flex items-center justify-center py-12">
					<svg
						class="animate-spin h-8 w-8 text-blue-600"
						xmlns="http://www.w3.org/2000/svg"
						fill="none"
						viewBox="0 0 24 24"
					>
						<circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"
						></circle>
						<path
							class="opacity-75"
							fill="currentColor"
							d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
						></path>
					</svg>
					<span class="ml-2 text-gray-600">Memuat jurnal...</span>
				</div>
			{:else if journals.length === 0}
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
							d="M12 6.253v13m0-13C10.832 5.477 9.246 5 7.5 5S4.168 5.477 3 6.253v13C4.168 18.477 5.754 18 7.5 18s3.332.477 4.5 1.253m0-13C13.168 5.477 14.754 5 16.5 5c1.746 0 3.332.477 4.5 1.253v13C19.832 18.477 18.246 18 16.5 18c-1.746 0-3.332.477-4.5 1.253"
						/>
					</svg>
					<h3 class="text-lg font-medium text-gray-900 mb-2">Belum ada jurnal</h3>
					<p class="text-gray-600">Mulai tulis jurnal pertama Anda untuk project ini!</p>
				</div>
			{:else}
				<div class="space-y-6">
					{#each journals as journal, index}
						<div
							class="bg-gradient-to-br from-white to-gray-50 border border-gray-200 rounded-xl p-6 hover:shadow-lg transition-shadow duration-200"
						>
							<div class="flex items-start justify-between mb-4">
								<div class="flex items-center space-x-3">
									<div class="w-10 h-10 bg-blue-100 rounded-full flex items-center justify-center">
										<span class="text-sm font-semibold text-blue-600"
											>#{journals.length - index}</span
										>
									</div>
									<div>
										<p class="text-sm font-medium text-gray-900">
											{formatDate(journal.created_at)}
										</p>
										{#if journal.status}
											<span
												class="inline-flex items-center px-2 py-1 rounded-full text-xs font-medium {getStatusColor(
													journal.status
												)} mt-1"
											>
												{journal.status}
											</span>
										{/if}
									</div>
								</div>
								<button onclick={() => startEdit(journal)} aria-label="Edit jurnal">
									<svg
										class="w-5 h-5 text-gray-400"
										fill="none"
										stroke="currentColor"
										viewBox="0 0 24 24"
									>
										<path
											stroke-linecap="round"
											stroke-linejoin="round"
											stroke-width="2"
											d="M15.232 5.232l3.536 3.536m-2.036-5.036a2.5 2.5 0 113.536 3.536L6.5 21.036H3v-3.572L16.732 3.732z"
										/>
									</svg>
								</button>
							</div>

							<div class="prose prose-sm max-w-none">
								<p class="text-gray-700 leading-relaxed whitespace-pre-wrap">{journal.isi}</p>
							</div>
						</div>
					{/each}
				</div>
			{/if}
		</div>
	</div>
</div>
