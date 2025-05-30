<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import Chart from 'chart.js/auto';

	export let data: any;
	export let options: any = {};

	let canvas: HTMLCanvasElement;
	let chart: Chart;

	onMount(() => {
		chart = new Chart(canvas, {
			type: 'pie',
			data,
			options
		});
	});

	onDestroy(() => {
		chart?.destroy();
	});
</script>

<canvas bind:this={canvas} width="250" height="250"></canvas>
