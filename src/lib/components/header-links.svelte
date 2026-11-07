<script module lang="ts">
	import type { HeaderNavProps } from './header-types';

	export type HeaderLinksProps = HeaderNavProps & {
		/** `fixed` stays on screen while the page scrolls. `absolute` is part of the top of the page and scrolls away with it. */
		position?: 'fixed' | 'absolute';
		/** Draws over the page in inverted colors (`mix-blend-mode: difference`), so it stays readable over any picture or color. */
		blend?: boolean;
	};
</script>

<script lang="ts">
	import MenuLinks from './menu-links.svelte';

	let {
		links = [],
		showSkipLink = true,
		position = 'fixed',
		blend = false
	}: HeaderLinksProps = $props();

	// Four plain links; a link that opens a dropdown has no page of its own, so it is left out.
	const shown = $derived(links.filter((link) => link.url && !link.links?.length).slice(0, 4));
</script>

<header class="header {position}" class:blend data-header-parts>
	{#if showSkipLink}
		<a class="skip-link" href="#main">Skip to main content</a>
	{/if}

	<MenuLinks class="links" links={shown} label="Main" type="text" />
</header>

<style lang="scss">
	@use 'base/mixins';

	.header {
		--top: 20px;

		top: 0;
		left: 0;
		z-index: var(--z-header);
		width: 100%;
		height: 0;
		pointer-events: none;

		@include mixins.min-lg {
			--top: 32px;
		}
	}

	.fixed {
		position: fixed;
	}

	.absolute {
		position: absolute;
	}

	.blend {
		color: #fff;
		mix-blend-mode: difference;
	}

	.header :global(.links) {
		position: absolute;
		top: var(--top);
		@include mixins.right-spacing;
		pointer-events: auto;

		:global(ul) {
			gap: 2ch;
		}
	}

	.skip-link {
		position: absolute;
		top: var(--top);
		@include mixins.left-spacing;
		width: 1px;
		height: 1px;
		overflow: hidden;
		clip-path: inset(50%);
		white-space: nowrap;
		pointer-events: auto;

		&:focus {
			width: auto;
			height: auto;
			padding: 8px 16px;
			clip-path: none;
			border: 1px solid var(--color-border);
			border-radius: var(--radius-btn);
			background: var(--color-bg);
		}
	}
</style>
