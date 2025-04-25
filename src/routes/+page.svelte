<script>
	import { Card, Button } from 'flowbite-svelte';
	import MarsHero from '$lib/MarsHero.svelte';
	import ImageModal from '$lib/ImageModal.svelte';

	let showImageModal = false;
</script>

<MarsHero slot="hero" />

<div class="container mx-auto px-4 py-8">
	<div class="grid gap-6 md:grid-cols-2 lg:grid-cols-3">
		<Card
			class="bg-orange-50 transition-transform duration-300 hover:scale-105 hover:shadow-xl dark:bg-stone-800"
		>
			<h5 class="mb-2 text-2xl font-bold tracking-tight text-orange-900 dark:text-orange-100">
				Latest Images
			</h5>
			<p class="mb-3 font-normal leading-tight text-orange-800 dark:text-orange-200">
				View the most recent photographs captured by Mars rovers.
			</p>
			<Button color="primary" on:click={() => (showImageModal = true)}>View Images</Button>
		</Card>

		<Card
			class="bg-orange-50 transition-transform duration-300 hover:scale-105 hover:shadow-xl dark:bg-stone-800"
		>
			<h5 class="mb-2 text-2xl font-bold tracking-tight text-orange-900 dark:text-orange-100">
				Rover Information
			</h5>
			<p class="mb-3 font-normal leading-tight text-orange-800 dark:text-orange-200">
				Learn about the different Mars rovers and their missions.
			</p>
			<Button
				href="https://mars.nasa.gov/mars-exploration/missions/"
				color="primary"
				target="_blank"
				rel="noopener noreferrer"
				class="no-underline"
			>
				Learn More
			</Button>
		</Card>

		<Card
			class="bg-orange-50 transition-transform duration-300 hover:scale-105 hover:shadow-xl dark:bg-stone-800"
		>
			<h5 class="mb-2 text-2xl font-bold tracking-tight text-orange-900 dark:text-orange-100">
				Mission Timeline
			</h5>
			<p class="mb-3 font-normal leading-tight text-orange-800 dark:text-orange-200">
				Explore the chronological journey of Mars exploration.
			</p>
			<Button
				href="https://mars.nasa.gov/mars-exploration/timeline/"
				target="_blank"
				rel="noopener noreferrer"
				color="primary"
				class="no-underline">View Timeline</Button
			>
		</Card>
	</div>
</div>

<ImageModal bind:open={showImageModal} onClose={() => (showImageModal = false)} />
