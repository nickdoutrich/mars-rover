<script>
	import { onMount } from 'svelte';
	import { Spinner } from 'flowbite-svelte';
	import FilterModal from './FilterModal.svelte';

	let marsImage = '';
	let loading = true;
	let error = false;
	let imageLoaded = false;
	let showFilterModal = false;

	onMount(async () => {
		try {
			const response = await fetch(
				`https://api.nasa.gov/mars-photos/api/v1/rovers/perseverance/latest_photos?api_key=${import.meta.env.VITE_NASA_API_KEY}`
			);
			const data = await response.json();
			marsImage = data.latest_photos[0].img_src;
		} catch (err) {
			error = true;
			console.error('Error fetching Mars image:', err);
		} finally {
			loading = false;
		}
	});

	function handleImageLoad() {
		imageLoaded = true;
	}
</script>

<div class="not-prose relative mb-12 h-[400px] overflow-hidden">
	{#if marsImage}
		<img
			src={marsImage}
			alt="Mars Surface"
			class="absolute inset-0 !mb-0 !mt-0 h-full w-full object-cover"
			on:load={handleImageLoad}
		/>
	{/if}

	{#if loading || !imageLoaded}
		<div
			class="absolute inset-0 z-20 flex items-center justify-center bg-gray-100 dark:bg-gray-800"
		>
			<Spinner size="12" />
		</div>
	{/if}

	{#if error}
		<div
			class="absolute inset-0 z-20 flex items-center justify-center bg-gray-100 dark:bg-gray-800"
		>
			<p class="text-red-500">Failed to load Mars image</p>
		</div>
	{/if}

	<div class="absolute inset-0 z-10 flex items-center justify-center bg-black bg-opacity-50">
		<div class="text-center text-white">
			<h2 class="mb-4 text-3xl font-bold">Explore Mars Like Never Before</h2>
			<p class="mb-6 text-xl">Access thousands of images directly from NASA's Mars rovers</p>
			<button
				class="rounded-lg bg-primary-600 px-6 py-2 font-bold text-white hover:bg-primary-700"
				on:click={() => (showFilterModal = true)}
			>
				Start Exploring
			</button>
		</div>
	</div>
</div>

<FilterModal bind:open={showFilterModal} onClose={() => (showFilterModal = false)} />
