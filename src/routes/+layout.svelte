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

	/* 
		SIMPLE FIREBASE INIT.
	*/
	import { initializeApp } from 'firebase/app';
	const firebaseConfig = {
		apiKey: 'AIzaSyCrYOsdjtb1KV6Cisx-yu88ZJZHKLGaQs4',
		authDomain: 'fftgen-65317.firebaseapp.com',
		projectId: 'fftgen-65317',
		storageBucket: 'fftgen-65317.appspot.com',
		messagingSenderId: '924445806685',
		appId: '1:924445806685:web:6e262eaafe03e9de5c3db4',
		measurementId: 'G-8B4W1Z3R5F'
	};
	const app = initializeApp(firebaseConfig);
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
