<script>
	import '../app.css';
	import Navbar from '$lib/Navbar.svelte';
</script>

<Navbar />

<!-- Hero and other full-width sections go here, outside prose -->
<slot name="hero" />

<!-- Main content with prose -->
<div class="prose max-w-none dark:prose-invert">
	<slot />
</div>
