<script module lang="ts">
	export type HeroLoadSpreadProps = Pick<SectionCopyProps, 'description' | 'cta'> & {
		/** The three words of the heading, left to right. */
		words: [string, string, string];
		/** The font size of the words, any CSS length. */
		size?: string;
		/** At least the height of the screen. Taller content still grows past it. */
		fullScreen?: boolean;
		/** The section's anchor, so a link or the CMS preview can point to `#id`. */
		id?: string;
		/** Skips the reveal and the movement, for the CMS preview. */
		static?: boolean;
		class?: string;
	};
</script>

<script lang="ts">
	import { onMount } from 'svelte';
	import { textReveal } from '$lib/attachments/text-reveal';
	import SectionCopy, { type SectionCopyProps } from '$lib/components/section-copy.svelte';
	import { loading } from '$lib/utils/loading.svelte';

	let {
		words,
		size = 'clamp(40px, 10vw, 180px)',
		fullScreen = true,
		id,
		static: isStatic = false,
		class: className,
		description,
		cta
	}: HeroLoadSpreadProps = $props();

	const REVEAL = {
		type: 'chars',
		duration: 0.8,
		stagger: 0.05,
		ease: 'power3.out',
		once: true
	} as const;
	const ROW_DELAY = 0.15;
	const HOLD = 0.3;
	const MOVE = 1200;

	// Seconds until the last letter of the last word has landed.
	const revealTime = $derived(
		Math.max(
			...words.map(
				(word, index) =>
					index * ROW_DELAY + ([...word].length - 1) * REVEAL.stagger + REVEAL.duration
			)
		)
	);

	let section = $state<HTMLElement>();
	let row = $state<HTMLElement>();
	let phase = $state<'reveal' | 'edges' | 'corners'>('reveal');
	let shown = $state(false);
	let distances = $state({ x1: 0, x3: 0, y1: 0, y3: 0 });

	const final = $derived(isStatic);
	const stage = $derived(final ? 'corners' : phase);

	const reveal = (index: number) =>
		isStatic || !loading.done
			? undefined
			: textReveal({ ...REVEAL, delay: index * ROW_DELAY, trigger: section });

	// Distances are from where the grid lays the words out to the padded edges and corners of the section.
	function measure() {
		if (!row) return;
		const [first, , last] = [...row.children] as HTMLElement[];
		const style = getComputedStyle(row);
		const [top, right, bottom, left] = [
			style.paddingTop,
			style.paddingRight,
			style.paddingBottom,
			style.paddingLeft
		].map(parseFloat);
		const overhang = parseFloat(getComputedStyle(first).fontSize) * 0.125;

		distances = {
			x1: left - first.offsetLeft,
			x3: row.clientWidth - right - (last.offsetLeft + last.offsetWidth),
			y1: top - first.offsetTop - overhang,
			y3: row.clientHeight - bottom - (last.offsetTop + last.offsetHeight) + overhang
		};
	}

	onMount(() => {
		measure();
		const watch = new ResizeObserver(measure);
		watch.observe(row!);
		for (const child of row!.children) watch.observe(child);
		return () => watch.disconnect();
	});

	$effect(() => {
		if (isStatic || !loading.done) return;
		if (matchMedia('(prefers-reduced-motion: reduce)').matches) {
			phase = 'corners';
			shown = true;
			return;
		}

		const timers: ReturnType<typeof setTimeout>[] = [];
		const watch = new IntersectionObserver(([entry]) => {
			if (!entry.isIntersecting) return;
			watch.disconnect();
			timers.push(
				setTimeout(() => (phase = 'edges'), (revealTime + HOLD) * 1000),
				setTimeout(() => (phase = 'corners'), (revealTime + HOLD) * 1000 + MOVE),
				setTimeout(() => (shown = true), (revealTime + HOLD) * 1000 + MOVE * 2)
			);
		});
		watch.observe(section!);

		return () => {
			watch.disconnect();
			timers.forEach(clearTimeout);
		};
	});
</script>

<section
	bind:this={section}
	{id}
	class="hero-load-spread {className ?? ''}"
	class:full-screen={fullScreen}
	data-stage={stage}
	style="--size: {size}; --x1: {distances.x1}px; --x3: {distances.x3}px; --y1: {distances.y1}px; --y3: {distances.y3}px"
>
	<h1 class="visually-hidden">{words.join(' ')}</h1>

	<div class="row" bind:this={row} aria-hidden="true">
		{#each words as word, index (index)}
			<div class="slot slot-{index + 1}">
				<span class="word" {@attach reveal(index)}>{word}</span>
			</div>
		{/each}
	</div>

	{#if description || cta?.primary?.text}
		<div class="copy" class:shown={shown || isStatic}>
			<SectionCopy
				{description}
				descriptionAttach={shown && !isStatic
					? textReveal({
							from: 'top',
							duration: 0.6,
							stagger: 0.03,
							once: true,
							trigger: section
						})
					: undefined}
				{cta}
				showEyebrow={false}
				showTitle={false}
			/>
		</div>
	{/if}
</section>

<style lang="scss">
	@use 'base/mixins';

	.hero-load-spread {
		--line-height: 1.3;

		position: relative;
		min-height: 28rem;
		overflow: clip;
	}

	.full-screen {
		min-height: 100lvh;
	}

	// The middle column is the page's center, so the middle word stays put while the others leave.
	.row {
		position: absolute;
		inset: 0;
		display: grid;
		grid-template-columns: 1fr auto 1fr;
		align-items: center;
		column-gap: 0.3em;
		padding: var(--body-padding);
		font-size: var(--size);
	}

	.slot {
		@include mixins.mq-motion-allow {
			transition:
				translate 1.2s var(--ease),
				transform 1.2s var(--ease);
		}
	}

	.slot-1 {
		justify-self: end;
	}

	.slot-3 {
		justify-self: start;
	}

	[data-stage='edges'] .slot-1,
	[data-stage='corners'] .slot-1 {
		translate: var(--x1) 0;
	}

	[data-stage='edges'] .slot-3,
	[data-stage='corners'] .slot-3 {
		translate: var(--x3) 0;
	}

	[data-stage='corners'] .slot-1 {
		transform: translateY(var(--y1));
	}

	[data-stage='corners'] .slot-3 {
		transform: translateY(var(--y3));
	}

	.word {
		display: block;
		font-family: var(--font-heading);
		font-size: var(--size);
		font-weight: 300;
		line-height: var(--line-height);
		letter-spacing: 0;
		white-space: nowrap;
	}

	.copy {
		position: absolute;
		bottom: var(--body-padding);
		left: var(--body-padding);
		width: calc(100% - var(--body-padding) * 2);
		max-width: 320px;

		// The last word takes the bottom right; on a narrow screen the copy rises above it.
		@include mixins.max-md {
			bottom: calc(var(--body-padding) + var(--size) * 1.2);
		}

		:global(.description) {
			text-wrap: balance;
		}

		@media (scripting: enabled) {
			opacity: 0;
		}

		&.shown {
			opacity: 1;

			@include mixins.mq-motion-allow {
				transition: opacity 0.6s var(--ease);
			}
		}
	}
</style>
