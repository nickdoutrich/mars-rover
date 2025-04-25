<script>
	import { onMount } from 'svelte';
	import { page } from '$app/stores';
	import { Spinner, Pagination } from 'flowbite-svelte';

	let photos = [];
	let loading = true;
	let error = false;
	let params;
	let currentPage = 1;
	const PHOTOS_PER_PAGE = 24;

	// Get the current page's photos
	$: paginatedPhotos = photos.slice(
		(currentPage - 1) * PHOTOS_PER_PAGE,
		currentPage * PHOTOS_PER_PAGE
	);

	// Calculate total pages
	$: totalPages = Math.ceil(photos.length / PHOTOS_PER_PAGE);

	// Generate array of page numbers for pagination
	$: pages = totalPages > 0 ? [...Array(totalPages).keys()].map((i) => i + 1) : [];

	function getDatesInRange(startDate, endDate) {
		const dates = [];
		let currentDate = new Date(startDate);
		const end = new Date(endDate);

		while (currentDate <= end) {
			dates.push(currentDate.toISOString().split('T')[0]);
			currentDate.setDate(currentDate.getDate() + 1);
		}
		return dates;
	}

	async function fetchRoverPhotos(rover, searchParams) {
		const queryParams = new URLSearchParams(searchParams);
		queryParams.set('api_key', import.meta.env.VITE_NASA_API_KEY);

		if (
			!searchParams.get('start_date') &&
			!searchParams.get('end_date') &&
			!searchParams.get('sol')
		) {
			const endpoint = `https://api.nasa.gov/mars-photos/api/v1/rovers/${rover}/latest_photos`;
			const response = await fetch(`${endpoint}?${queryParams}`);
			const data = await response.json();
			return data.latest_photos || [];
		}

		if (searchParams.get('start_date') && searchParams.get('end_date')) {
			const dates = getDatesInRange(searchParams.get('start_date'), searchParams.get('end_date'));
			const allPhotos = [];

			for (const date of dates) {
				queryParams.set('earth_date', date);
				const endpoint = `https://api.nasa.gov/mars-photos/api/v1/rovers/${rover}/photos`;
				const response = await fetch(`${endpoint}?${queryParams}`);
				const data = await response.json();
				if (data.photos) {
					allPhotos.push(...data.photos);
				}
			}
			return allPhotos;
		}

		const endpoint = `https://api.nasa.gov/mars-photos/api/v1/rovers/${rover}/photos`;
		const response = await fetch(`${endpoint}?${queryParams}`);
		const data = await response.json();
		return data.photos || [];
	}

	onMount(async () => {
		try {
			params = $page.url.searchParams;
			currentPage = parseInt(params.get('page')) || 1;

			const rovers =
				params.get('rover') && params.get('rover') !== 'all'
					? [params.get('rover')]
					: ['perseverance', 'curiosity', 'opportunity', 'spirit'];

			const allPhotos = [];
			for (const rover of rovers) {
				const roverPhotos = await fetchRoverPhotos(rover, params);
				allPhotos.push(...roverPhotos);
			}

			photos = allPhotos.sort((a, b) => new Date(b.earth_date) - new Date(a.earth_date));
		} catch (err) {
			error = true;
			console.error('Error fetching photos:', err);
		} finally {
			loading = false;
		}
	});

	function handlePageClick(e) {
		console.log('Page clicked:', e.detail); // For debugging
		currentPage = e.detail;
		window.scrollTo({ top: 0, behavior: 'smooth' });
	}

	$: if (currentPage) {
		// Update URL whenever currentPage changes
		const url = new URL(window.location);
		url.searchParams.set('page', currentPage.toString());
		window.history.replaceState({}, '', url);
	}

	function getPageNumbers(current, total) {
		if (total <= 7) {
			// If 7 or fewer pages, show all
			return Array.from({ length: total }, (_, i) => i + 1);
		}

		if (current <= 3) {
			// Near start: 1 2 3 4 5 ... last
			return [1, 2, 3, 4, 5, '...', total];
		}

		if (current >= total - 2) {
			// Near end: 1 ... total-4 total-3 total-2 total-1 total
			return [1, '...', total - 4, total - 3, total - 2, total - 1, total];
		}

		// In middle: 1 ... current-1 current current+1 ... last
		return [1, '...', current - 1, current, current + 1, '...', total];
	}

	$: pageNumbers = getPageNumbers(currentPage, totalPages);
</script>

<div class="container mx-auto px-4 py-8">
	<div class="mb-8">
		<h1 class="text-3xl font-bold">Mars Rover Photos</h1>
		<p class="mt-2 text-gray-600 dark:text-gray-400">
			{#if params?.toString()}
				Showing filtered results for:
				{#if params.get('rover')}
					{params.get('rover').charAt(0).toUpperCase() + params.get('rover').slice(1)} Rover
				{:else}
					All Rovers
				{/if}
				{#if params.get('camera')}
					• {params.get('camera')} Camera
				{/if}
				{#if params.get('start_date') && params.get('end_date')}
					• Photos from {params.get('start_date')} to {params.get('end_date')}
				{:else if params.get('sol')}
					• Photos from Sol {params.get('sol')}
				{:else}
					• Most recent available photos from each rover
				{/if}
			{:else}
				Showing the most recent available photos from all rovers
			{/if}
		</p>
	</div>

	{#if loading}
		<div class="flex items-center justify-center py-8">
			<Spinner size="12" />
		</div>
	{:else if error}
		<div class="rounded-lg bg-red-50 p-4 text-red-800 dark:bg-gray-800 dark:text-red-400">
			Error loading photos. Please try again.
		</div>
	{:else if photos.length === 0}
		<div class="rounded-lg bg-blue-50 p-4 text-blue-800 dark:bg-gray-800 dark:text-blue-400">
			No photos found for these filters.
		</div>
	{:else}
		<div class="mb-8">
			<p class="text-gray-600 dark:text-gray-400">
				Showing {(currentPage - 1) * PHOTOS_PER_PAGE + 1} - {Math.min(
					currentPage * PHOTOS_PER_PAGE,
					photos.length
				)} of {photos.length} photos
			</p>
		</div>

		<div class="grid gap-4 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4">
			{#each paginatedPhotos as photo}
				<div class="overflow-hidden rounded-lg bg-white shadow-lg dark:bg-gray-800">
					<img src={photo.img_src} alt="Mars surface" class="h-48 w-full object-cover" />
					<div class="p-4">
						<p class="font-bold">{photo.rover.name}</p>
						<p class="text-sm text-gray-600 dark:text-gray-400">
							Camera: {photo.camera.full_name}
						</p>
						<p class="text-sm text-gray-600 dark:text-gray-400">
							Date: {new Date(photo.earth_date).toLocaleDateString()}
						</p>
					</div>
				</div>
			{/each}
		</div>

		{#if totalPages > 1}
			<div class="mt-8 flex justify-center space-x-2">
				<button
					class="rounded border px-3 py-1 hover:bg-orange-100 disabled:opacity-50 disabled:hover:bg-transparent"
					disabled={currentPage === 1}
					on:click={() => {
						currentPage--;
						window.scrollTo({ top: 0, behavior: 'smooth' });
					}}
				>
					Previous
				</button>

				{#each pageNumbers as item}
					{#if item === '...'}
						<span class="px-3 py-1">...</span>
					{:else}
						<button
							class="rounded border px-3 py-1 {currentPage === item
								? 'bg-orange-500 text-white hover:bg-orange-600'
								: 'hover:bg-orange-100'}"
							on:click={() => {
								currentPage = item;
								window.scrollTo({ top: 0, behavior: 'smooth' });
							}}
						>
							{item}
						</button>
					{/if}
				{/each}

				<button
					class="rounded border px-3 py-1 hover:bg-orange-100 disabled:opacity-50 disabled:hover:bg-transparent"
					disabled={currentPage === totalPages}
					on:click={() => {
						currentPage++;
						window.scrollTo({ top: 0, behavior: 'smooth' });
					}}
				>
					Next
				</button>
			</div>
		{/if}
	{/if}
</div>
