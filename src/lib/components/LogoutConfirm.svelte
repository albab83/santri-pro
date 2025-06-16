<script lang="ts">
  import { createEventDispatcher } from 'svelte';
  import { goto } from '$app/navigation';
  import { fade } from 'svelte/transition';

  export let visible = false;
  const dispatch = createEventDispatcher();

  function cancel() {
    dispatch('cancel');
  }

  function confirm() {
    localStorage.removeItem('token');
    localStorage.removeItem('user');
    localStorage.removeItem('role');
    goto('/');
  }
</script>

{#if visible}
  <div class="fixed inset-0 flex items-center justify-center z-50 bg-black/40">
    <div class="bg-white rounded-xl shadow-2xl p-8 max-w-sm w-full text-center mx-5"
    transition:fade >
      <h2 class="text-xl font-bold mb-2">Konfirmasi Logout</h2>
      <p class="mb-6">Apakah Anda yakin ingin keluar dari akun?</p>
      <div class="flex justify-center space-x-4">
        <button on:click={cancel} class="px-4 py-2 rounded-lg bg-gray-100 hover:bg-gray-200">Batal</button>
        <button on:click={confirm} class="px-4 py-2 rounded-lg bg-red-600 text-white hover:bg-red-700">Logout</button>
      </div>
    </div>
  </div>
{/if}
