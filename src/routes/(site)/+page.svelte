<script lang="ts">
	import { onMount } from 'svelte';
	import HeroLoadSpread from '$lib/sections/hero-load-spread.svelte';
	import hero from '$lib/content/page_home/hero-load-spread.json';
	import { hasIdentityToken, loadIdentity } from '$lib/utils/identity';

	const cta = {
		primary: hero.cta.primary,
		secondary: undefined
	};

	// Netlify sends CMS invite and recovery links to the home page.
	onMount(() => {
		if (!hasIdentityToken()) return;
		loadIdentity().then((identity) => identity.on('login', () => location.assign('/admin/')));
	});
</script>

<HeroLoadSpread
	id="hero"
	words={hero.words as [string, string, string]}
	description={hero.description}
	{cta}
/>
