<script>
	import { Navbar, NavBrand, NavUl, NavLi } from 'flowbite-svelte';
	import AboutModal from './AboutModal.svelte';

	let showAboutModal = false;
</script>

<Navbar class="border-b border-orange-200 bg-orange-100 dark:border-stone-700 dark:bg-stone-800">
	<NavBrand href="/">
		<!-- Your logo or brand name -->
		<!-- <img src="/path-to-logo.png" alt="Logo" class="mr-3 h-8" /> -->
		<span
			class="self-center whitespace-nowrap text-xl font-semibold text-orange-900 dark:text-orange-100"
			>CPAL x Code2College</span
		>
	</NavBrand>
	<NavUl>
		<NavLi href="/" class="text-orange-900 dark:text-orange-100">Home</NavLi>
		<NavLi
			href="/"
			on:click={(e) => {
				e.preventDefault();
				showAboutModal = true;
			}}
			class="text-orange-900 dark:text-orange-100">About</NavLi
		>
		<!-- <NavLi href="/services">Services</NavLi>
		<NavLi href="/contact">Contact</NavLi> -->
	</NavUl>
</Navbar>

<AboutModal bind:showModal={showAboutModal} />
