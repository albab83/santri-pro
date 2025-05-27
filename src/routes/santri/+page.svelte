<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  import { goto } from '$app/navigation';

  let judul = '';
  let deskripsi = '';
  let tujuan = '';
  let message = '';
  let projects : any[] = [];
  let showModal = false;
  let loading = false;
  let user: any = null;
  let token: string | null = null;
  let intervalId: any = null;
  let loadingProjectId: string | null = null;

  const baseUrl = import.meta.env.VITE_API_BASE_URL 

  //tandaiSelesai

  async function tandaiSelesai(projectId: string) {
  if (!confirm('Yakin ingin menandai project ini sebagai selesai?')) return;
  loadingProjectId = projectId;
  try {
    const res = await fetch(`${baseUrl}/api/project/${projectId}/status`, {
      method: 'PUT',
      headers: {
        Authorization: `Bearer ${token}`
      }
    });
    const data = await res.json();
    if (res.ok) {
      message = 'Project berhasil diselesaikan!';
      await fetchMyProjects();
    } else {
      message = data.message || 'Gagal menyelesaikan project';
    }
  } catch (err) {
    message = 'Terjadi kesalahan koneksi';
  } finally {
    loadingProjectId = null;
  }
}

   // Ambil data user dari token
  async function fetchUser() {
    
    if (!token) return;
    try {
      const res = await fetch(`${baseUrl}/api/auth/getMe`, {
        headers: { Authorization: `Bearer ${token}` }
      });
      if (res.ok) {
        user = await res.json();
        console.log(user);
      }
    } catch (err) {
        console.error('Error fetching user:', err);
      // Optional: handle error
    }
  }

  // Ambil daftar project user
  async function fetchMyProjects() {
    try {
      const res = await fetch(`${baseUrl}/api/project/my`, {
        headers: { Authorization: `Bearer ${token}` }
      });

      if (res.ok) {
        projects = await res.json();
      } else {
        message = 'Gagal mengambil data project';
      }
    } catch (err) {
      message = 'Gagal terhubung ke server';
    }
  }

  // Kirim proposal project
  async function handleSubmit() {
    loading = true;
    message = '';
    try {
      const res = await fetch(`${baseUrl}/api/project`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          Authorization: `Bearer ${token}`
        },
        body: JSON.stringify({ judul, deskripsi, tujuan })
      });

      const data = await res.json();
      if (res.ok) {
        message = 'Proposal berhasil dikirim!';
        judul = deskripsi = tujuan = '';
        fetchMyProjects(); // refresh list
        setTimeout(() => {
          showModal = false;
          message = '';
        }, 1500);
      } else {
        message = data.message || 'Gagal mengirim proposal';
      }
    } catch (err) {
      message = 'Terjadi kesalahan koneksi';
    } finally {
      loading = false;
    }
  }

  function bukaHalamanJurnal(projectId: string) {
    goto(`/journal/${projectId}`);
  }

  function openModal() {
    showModal = true;
    message = '';
  }

  function closeModal() {
    showModal = false;
    message = '';
    judul = deskripsi = tujuan = '';
  }

  function getStatusColor(status: string) {
    switch (status?.toLowerCase()) {
      case 'approved':
      case 'diterima':
        return 'text-green-100 bg-green-600';
      case 'rejected':
      case 'ditolak':
        return 'text-red-700 bg-red-100';
      case 'pending':
      case 'menunggu':
        return 'text-yellow-100 bg-yellow-600';
      default:
        return 'text-gray-700 bg-gray-100';
    }
  }

  function formatDate(dateString: string) {
    return new Date(dateString).toLocaleDateString('id-ID', {
      year: 'numeric',
      month: 'long',
      day: 'numeric'
    });
  }

  onMount(() => {
  token = typeof window !== 'undefined' ? localStorage.getItem('token') : null;
  fetchUser();
  fetchMyProjects();
  intervalId = setInterval(fetchMyProjects, 5000); // refresh tiap 10 detik
});

// Bersihkan interval saat komponen di-unmount
onDestroy(() => {
  clearInterval(intervalId);
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
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"/>
            </svg>
          </div>
          <div>
            <h1 class="text-[20px] sm:text-2xl md:text-3xl lg:text-3xl font-bold text-gray-900">
              Assalamu'alaikum Selamat Datang {user?.nama}, di Self Project
            </h1>
            <p class="text-gray-600">Kelola dan ajukan project mandiri Anda </p>
          </div>
        </div>
        
      </div>
    </div>

    <!-- Project List -->
    <div class="bg-white rounded-2xl shadow-xl p-8">
      <div class="flex justify-between items-center space-x-3 mb-6">
        <div class="flex items-center space-x-3">
          <svg class="w-6 h-6 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 11H5m14 0a2 2 0 012 2v6a2 2 0 01-2 2H5a2 2 0 01-2-2v-6a2 2 0 012-2m14 0V9a2 2 0 00-2-2M5 11V9a2 2 0 012-2m0 0V5a2 2 0 012-2h6a2 2 0 012 2v2M7 7h10"/>
          </svg>
          <h2 class="text-[20px] sm:text-2xl md:text-2xl lg:text-2xl font-bold text-gray-900">Project Saya</h2>
        </div>
        <button
          on:click={openModal}
          class="bg-blue-600 hover:bg-blue-700 text-white font-medium px-6 py-3 rounded-lg transition-colors duration-200 flex items-center space-x-2"
        >
          <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4"/>
          </svg>
          <span class="hidden sm:block">Ajukan Project Baru</span>
        </button>
      </div>

      {#if projects.length === 0}
        <div class="text-center py-12">
          <svg class="w-16 h-16 text-gray-400 mx-auto mb-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"/>
          </svg>
          <h3 class="text-lg font-medium text-gray-900 mb-2">Belum ada project</h3>
          <p class="text-gray-600 mb-4">Mulai dengan mengajukan project mandiri pertama Anda</p>
          <button
            on:click={openModal}
            class="bg-blue-600 hover:bg-blue-700 text-white font-medium px-4 py-2 rounded-lg transition-colors duration-200"
          >
            Ajukan Project
          </button>
        </div>
      {:else}
        <div class="grid gap-6 md:grid-cols-2 lg:grid-cols-3">
          {#each projects as project}
            <div class="bg-gradient-to-br from-white to-gray-50 border border-gray-200 rounded-xl p-6 hover:shadow-lg transition-shadow duration-200">
              <div class="flex items-start justify-between mb-4">
                <div class="flex-1">
                  <h3 class="text-lg font-semibold text-gray-900 mb-2">{project.judul}</h3>
                  <span class="inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium {getStatusColor(project.status)}">
                    {project.status}
                  </span>
                </div>
                <svg class="w-6 h-6 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"/>
                </svg>
              </div>
              
              <div class="space-y-3">
                <div>
                  <p class="text-sm font-medium text-gray-700 mb-1">Deskripsi:</p>
                  <p class="text-sm text-gray-600 line-clamp-3">{project.deskripsi}</p>
                </div>
                
                <div>
                  <p class="text-sm font-medium text-gray-700 mb-1">Tujuan:</p>
                  <p class="text-sm text-gray-600 line-clamp-2">{project.tujuan}</p>
                </div>
                {#if project.status?.toLowerCase() === 'ditolak' || project.status?.toLowerCase() === 'rejected'}
                    <div>
                        <p class="text-sm font-medium text-red-700 mb-1">Alasan Penolakan:</p>
                        <p class="text-sm text-red-600 italic">{project.alasan_penolakan}</p>
                    </div>
                {/if}
                
                <div class="pt-3 border-t border-gray-200">
                  <div class="flex items-center text-xs text-gray-500">
                    <svg class="w-4 h-4 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 7V3a4 4 0 118 0v4m-4 12v-2m-4 2h8a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2z"/>
                    </svg>
                    Diajukan pada {formatDate(project.tanggal_pengajuan)}
                  </div>
                </div>

                {#if project.status?.toLowerCase() === 'diterima' || project.status?.toLowerCase() === 'approved'}
                    <div class="mt-4">
                      <button
                        on:click={() => bukaHalamanJurnal(project.id)}
                        class="w-full text-center bg-indigo-600 hover:bg-indigo-700 text-white text-sm font-medium py-2 px-4 rounded-lg transition"
                        >
                        Buka Jurnal
                      </button>
                      <button
                        on:click={() => tandaiSelesai(project.id)}
                        class="mt-4 w-full text-center bg-green-600 hover:bg-green-700 text-white text-sm font-medium py-2 px-4 rounded-lg transition"
                        disabled={loadingProjectId === project.id}
                      >
                        {#if loadingProjectId === project.id}
                          <svg class="animate-spin h-5 w-5 inline mr-2" fill="none" viewBox="0 0 24 24">
                            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                          </svg>
                          Menyelesaikan...
                        {:else}
                          Tandai Selesai
                        {/if}
                      </button>
                    </div>
                {/if}
              </div>
            </div>
          {/each}
        </div>
      {/if}
    </div>
  </div>
</div>

<!-- Modal -->
{#if showModal}
  <div class="fixed inset-0 bg-black/50 flex items-center justify-center p-4 z-50">
    <div class="bg-white rounded-2xl shadow-2xl w-full max-w-2xl h-9/10 overflow-y-auto ">
      <!-- Modal Header -->
      <div class="flex items-center justify-between p-6 border-b border-gray-200">
        <div class="flex items-center space-x-3">
          <div class="w-10 h-10 bg-blue-600 rounded-full flex items-center justify-center">
            <svg class="w-5 h-5 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4"/>
            </svg>
          </div>
          <div>
            <h3 class="text-xl font-bold text-gray-900">Ajukan Self Project</h3>
            <p class="text-sm text-gray-600">Isi form di bawah untuk mengajukan project mandiri</p>
          </div>
        </div>
        <button
          on:click={closeModal}
          class="text-gray-400 hover:text-gray-600 transition-colors"
          aria-label="Tutup modal"
        >
          <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"/>
          </svg>
        </button>
      </div>

      <!-- Modal Body -->
      <div class="m-4 p-6">
        <form on:submit|preventDefault={handleSubmit} class="space-y-6">
          <!-- Judul Field -->
          <div class="space-y-2">
            <label for="judul" class="block text-sm font-medium text-gray-700">
              Judul Project
            </label>
            <div class="relative">
              <div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
                <svg class="h-5 w-5 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 7h.01M7 3h5c.512 0 1.024.195 1.414.586l7 7a2 2 0 010 2.828l-7 7a1.994 1.994 0 01-2.828 0l-7-7A1.994 1.994 0 013 12V7a4 4 0 014-4z"/>
                </svg>
              </div>
              <input
                id="judul"
                type="text"
                bind:value={judul}
                placeholder="Masukkan judul project yang menarik"
                required
                disabled={loading}
                class="w-full pl-10 pr-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition-colors disabled:bg-gray-50 disabled:cursor-not-allowed"
              />
            </div>
          </div>

          <!-- Deskripsi Field -->
          <div class="space-y-2">
            <label for="deskripsi" class="block text-sm font-medium text-gray-700">
              Deskripsi Project
            </label>
            <div class="relative">
              <div class="absolute top-3 left-3 pointer-events-none">
                <svg class="h-5 w-5 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"/>
                </svg>
              </div>
              <textarea
                id="deskripsi"
                bind:value={deskripsi}
                placeholder="Jelaskan secara detail tentang project yang akan Anda kerjakan..."
                required
                disabled={loading}
                rows="4"
                class="w-full pl-10 pr-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition-colors disabled:bg-gray-50 disabled:cursor-not-allowed resize-none"
              ></textarea>
            </div>
          </div>

          <!-- Tujuan Field -->
          <div class="space-y-2">
            <label for="tujuan" class="block text-sm font-medium text-gray-700">
              Tujuan Project
            </label>
            <div class="relative">
              <div class="absolute top-3 left-3 pointer-events-none">
                <svg class="h-5 w-5 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4M7.835 4.697a3.42 3.42 0 001.946-.806 3.42 3.42 0 014.438 0 3.42 3.42 0 001.946.806 3.42 3.42 0 013.138 3.138 3.42 3.42 0 00.806 1.946 3.42 3.42 0 010 4.438 3.42 3.42 0 00-.806 1.946 3.42 3.42 0 01-3.138 3.138 3.42 3.42 0 00-1.946.806 3.42 3.42 0 01-4.438 0 3.42 3.42 0 00-1.946-.806 3.42 3.42 0 01-3.138-3.138 3.42 3.42 0 00-.806-1.946 3.42 3.42 0 010-4.438 3.42 3.42 0 00.806-1.946 3.42 3.42 0 013.138-3.138z"/>
                </svg>
              </div>
              <textarea
                id="tujuan"
                bind:value={tujuan}
                placeholder="Apa tujuan dan manfaat yang ingin dicapai dari project ini..."
                required
                disabled={loading}
                rows="3"
                class="w-full pl-10 pr-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition-colors disabled:bg-gray-50 disabled:cursor-not-allowed resize-none"
              ></textarea>
            </div>
          </div>

          <!-- Message Display -->
          {#if message}
            <div class="rounded-lg p-4 {message.includes('berhasil') ? 'bg-green-50 border border-green-200' : 'bg-red-50 border border-red-200'}">
              <div class="flex items-center">
                {#if message.includes('berhasil')}
                  <svg class="h-5 w-5 text-green-400 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"/>
                  </svg>
                  <p class="text-sm text-green-800">{message}</p>
                {:else}
                  <svg class="h-5 w-5 text-red-400 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"/>
                  </svg>
                  <p class="text-sm text-red-800">{message}</p>
                {/if}
              </div>
            </div>
          {/if}

          <!-- Modal Footer -->
          <div class="flex items-center justify-end space-x-3 pt-6 border-t border-gray-200">
            <button
              type="button"
              on:click={closeModal}
              disabled={loading}
              class="px-4 py-2 text-sm font-medium text-gray-700 bg-white border border-gray-300 rounded-lg hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 disabled:cursor-not-allowed"
            >
              Batal
            </button>
            <button
              type="submit"
              disabled={loading}
              class="bg-blue-600 hover:bg-blue-700 disabled:bg-blue-400 text-white font-medium py-2 px-6 rounded-lg transition-colors duration-200 flex items-center space-x-2 disabled:cursor-not-allowed"
            >
              {#if loading}
                <svg class="animate-spin h-4 w-4 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                  <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                  <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                </svg>
                <span>Mengirim...</span>
              {:else}
                <svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 19l9 2-9-18-9 18 9-2zm0 0v-8"/>
                </svg>
                <span>Kirim Proposal</span>
              {/if}
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
{/if}