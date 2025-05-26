<script lang="ts">
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';


  let judul = '';
  let deskripsi = '';
  let tujuan = '';
  let message = '';
  let projects: any[] = [];
  const baseUrl = import.meta.env.VITE_API_BASE_URL;

  const token = localStorage.getItem('token');

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
      } else {
        message = data.message || 'Gagal mengirim proposal';
      }
    } catch (err) {
      message = 'Terjadi kesalahan koneksi';
    }
  }

  onMount(() => {
    fetchMyProjects();
  });
</script>

<div class="max-w-4xl mx-auto mt-10 p-6 bg-white rounded shadow space-y-10">
  <!-- Form Pengajuan -->
  <div>
    <h1 class="text-2xl font-bold mb-4">Ajukan Self Project</h1>
    <form on:submit|preventDefault={handleSubmit} class="space-y-4">
      <input bind:value={judul} type="text" placeholder="Judul Project" required class="w-full p-2 border rounded" />
      <textarea bind:value={deskripsi} placeholder="Deskripsi" required class="w-full p-2 border rounded"></textarea>
      <textarea bind:value={tujuan} placeholder="Tujuan" required class="w-full p-2 border rounded"></textarea>
      <button type="submit" class="w-full bg-blue-600 text-white p-2 rounded hover:bg-blue-700">
        Kirim Proposal
      </button>
    </form>
    {#if message}
      <p class="mt-2 text-sm text-center text-red-600">{message}</p>
    {/if}
  </div>

  <!-- Daftar Project -->
  <div>
    <h2 class="text-xl font-semibold mb-4">Project Saya</h2>

    {#if projects.length === 0}
      <p>Belum ada project yang diajukan.</p>
    {:else}
      <ul class="space-y-4">
        {#each projects as project}
          <li class="p-4 border rounded bg-gray-50">
            <h3 class="text-lg font-semibold">{project.judul}</h3>
            <p><strong>Tujuan:</strong> {project.tujuan}</p>
            <p><strong>Status:</strong> <span class="capitalize">{project.status}</span></p>
            <p class="text-sm text-gray-500">Diajukan pada: {project.tanggal_pengajuan}</p>
          </li>
        {/each}
      </ul>
    {/if}
  </div>
</div>
