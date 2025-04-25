<script>
	import { Modal, Button } from 'flowbite-svelte';
	import { Spinner } from 'flowbite-svelte';

	export let open = false;
	export let onClose = () => {};

	let images = [];
	let currentIndex = 0;
	let loading = true;
	let error = false;

	async function fetchImages() {
		try {
			loading = true;
			error = false;
			// Fetch from all active rovers to get more recent images
			const rovers = ['perseverance', 'curiosity'];
			const allImages = [];

			for (const rover of rovers) {
				const response = await fetch(
					`https://api.nasa.gov/mars-photos/api/v1/rovers/${rover}/latest_photos?api_key=${import.meta.env.VITE_NASA_API_KEY}`
				);
				const data = await response.json();
				allImages.push(...data.latest_photos);
			}

			// Sort by date, most recent first
			images = allImages
				.sort((a, b) => new Date(b.earth_date) - new Date(a.earth_date))
				.slice(0, 25); // Limit to 25 most recent
		} catch (err) {
			error = true;
			console.error('Error fetching images:', err);
		} finally {
			loading = false;
		}
	}

	function nextImage() {
		currentIndex = (currentIndex + 1) % images.length;
	}

	function previousImage() {
		currentIndex = (currentIndex - 1 + images.length) % images.length;
	}

	$: if (open) {
		fetchImages();
		currentIndex = 0;
	}
</script>

<Modal bind:open size="xl" class="w-full" outsideclose on:close={onClose}>
	<div class="relative min-h-[50vh]">
		{#if loading}
			<div class="flex h-[50vh] items-center justify-center">
				<Spinner size="12" />
			</div>
		{:else if error}
			<div class="flex h-[50vh] items-center justify-center">
				<p class="text-red-500">Failed to load images</p>
			</div>
		{:else if images.length === 0}
			<div class="flex h-[50vh] items-center justify-center">
				<p>No images found</p>
			</div>
		{:else}
			<img
				src={images[currentIndex].img_src}
				alt="Mars Surface"
				class="h-auto max-h-[70vh] w-full object-contain"
			/>
			<div class="mt-4 flex items-center justify-between">
				<Button color="primary" on:click={previousImage}>Previous</Button>
				<div class="text-center">
					<p class="font-bold">{images[currentIndex].rover.name} Rover</p>
					<p class="text-sm text-gray-600">
						Camera: {images[currentIndex].camera.full_name}
					</p>
					<p class="text-sm text-gray-600">
						Date: {new Date(images[currentIndex].earth_date).toLocaleDateString()}
					</p>
				</div>
				<Button color="primary" on:click={nextImage}>Next</Button>
			</div>
		{/if}
	</div>
</Modal>
