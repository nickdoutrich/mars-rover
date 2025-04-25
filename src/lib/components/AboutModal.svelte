<script>
	import { Modal, Button } from 'flowbite-svelte';
	export let showModal = false;
</script>

<Modal bind:open={showModal} size="lg" outsideclose>
	<div class="space-y-6 p-6">
		<h3 class="text-xl font-medium text-orange-900 dark:text-orange-100">Mars Rover Explorer</h3>

		<div class="space-y-4">
			<div>
				<h4 class="mb-2 font-semibold text-orange-800 dark:text-orange-200">What is this?</h4>
				<p class="text-orange-700 dark:text-orange-300">
					An interactive platform that lets you explore Mars through the eyes of NASA's rovers.
					Browse thousands of photos taken directly from the Martian surface!
				</p>
			</div>

			<div>
				<h4 class="mb-2 font-semibold text-orange-800 dark:text-orange-200">Key Features</h4>
				<ul class="list-inside list-disc text-orange-700 dark:text-orange-300">
					<li>View latest Mars rover photos</li>
					<li>Search by date, rover, and camera type</li>
					<li>Get detailed information about each image</li>
					<li>Learn about different Mars rovers</li>
				</ul>
			</div>

			<div>
				<h4 class="mb-2 font-semibold text-orange-800 dark:text-orange-200">About the Project</h4>
				<p class="text-orange-700 dark:text-orange-300">
					This is a Code2College student project using NASA's public APIs to create an engaging way
					to explore Mars photography while learning modern web development.
				</p>
			</div>
		</div>
	</div>
	<svelte:fragment slot="footer">
		<div class="flex w-full justify-end">
			<Button color="primary" on:click={() => (showModal = false)}>Close</Button>
		</div>
	</svelte:fragment>
</Modal>
