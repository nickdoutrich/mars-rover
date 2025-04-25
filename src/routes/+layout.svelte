<script>
	import '$lib/styles/app.css';
	import Navbar from '$lib/components/Navbar.svelte';
</script>

<Navbar />

<!-- Hero and other full-width sections go here, outside prose -->
<slot name="hero" />

<!-- Main content with prose -->
<div class="prose max-w-none dark:prose-invert">
	<slot />
</div>
