<script>
	/*  
        TAILWINDCSS
    */
	import '../app.css';

	/* 
		THEME CHANGE
	*/
	import { onMount } from 'svelte';
	import { themeChange } from 'theme-change';
	onMount(() => {
		themeChange(false);
	});
</script>

<slot />

<svelte:head>
	<title>Fourier Transformer - Are Olsen.</title>
</svelte:head>

<style>
	:root {
		width: 100%;
		padding: 0;
		margin: 0;
		min-height: 100vh;
		display: flex;
		justify-content: center;
		text-align: center;
	}
</style>
