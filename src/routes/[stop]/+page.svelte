<script lang="ts">
	import { PUBLIC_BASE_URL } from '$env/static/public';
	import FooterNavigation from '$lib/components/FooterNavigation.svelte';
	import { onMount, setContext } from 'svelte';
	import { invalidateAll } from '$app/navigation';
	import ModesSwitch from '$lib/components/ModesSwitch.svelte';
	import LiveTripAnimation from './LiveTripAnimation.svelte';
	import StopFavoriteButton from '$lib/components/StopFavoriteButton.svelte';
	import { Flag } from '@lucide/svelte';
	import type { ExpandedTripState } from '$lib/Trip';
	import Direction from './Direction.svelte';

	let { data } = $props();

	let details = $derived(data.details);

	const tripState: ExpandedTripState = {
		id: null,
	};
	const expandedTrip = $state(tripState);
	setContext('expandedTrip', expandedTrip);

	const REFRESH_INTERVAL_SECONDS = 30;
	let timer: ReturnType<typeof setInterval>;

	let lastUpdatedAgo = $state("")
	updateTime()

	function onVisibilityChange() {
		clearInterval(timer);
		if (document.visibilityState != 'hidden') {
			invalidateAll();
			timer = setInterval(updateTime, 5 * 1000);
		}
	}

	function updateTime() {
		const millis = Date.now() - details.lastUpdatedAt.getTime();
		const seconds = Math.floor(millis / 1000);
		// Round to nearest 5s
		const rounded_seconds = Math.floor(seconds / 5) * 5;
		lastUpdatedAgo = rounded_seconds > 0 ? `${rounded_seconds}s fa` : "ora";
		if (seconds > REFRESH_INTERVAL_SECONDS) {
			invalidateAll()
		}
	}

	onMount(() => {
		timer = setInterval(updateTime, 5 * 1000);
		document.addEventListener('visibilitychange', onVisibilityChange);
		return () => {
			clearInterval(timer);
			document.removeEventListener('visibilitychange', onVisibilityChange);
		};
	});
</script>

<svelte:head>
	<title>{details.name}</title>
	<link rel="canonical" href="{PUBLIC_BASE_URL}/{details.canonicalSlug}" />
</svelte:head>

<header>
	<div class="text-center">
		<h1 class="inline text-center text-4xl font-semibold">
			{details.name}
		</h1>
		<StopFavoriteButton stopCode={details.code} className="pl-2" />
	</div>
	<div class="mt-1 text-center text-sm">
		aggiornato {lastUpdatedAgo}
	</div>

	{#if details.trainStationSlug}
		<div class="mt-6 flex justify-center">
			<ModesSwitch
				isBus={true}
				stopSlug={details.canonicalSlug}
				stationSlug={details.trainStationSlug}
			/>
		</div>
	{/if}
</header>

<main>
	<!-- eslint-disable-next-line svelte/require-each-key -->
	{#each details.directions as direction}
		<Direction {direction} alone={details.directions.length < 2} />
	{/each}
</main>

<footer class="my-12">
	<div class="space-y-2 text-sm text-neutral-500">
		<p>
			Il pallino verde
			<LiveTripAnimation className="inline-block mx-1" live="green" />
			indica che i dati sono in tempo reale.
		</p>

		<p>
			Il pallino è giallo
			<LiveTripAnimation className="inline-block mx-1" live="yellow" />
			se l'autobus non ha trasmesso aggiornamenti negli ultimi 5 minuti.
		</p>

		<p>
			Il simbolo
			<Flag class="inline size-4" />
			indica che la corsa terminerà a questa fermata.
		</p>

		<p>La pagina si aggiorna automaticamente ogni 30 secondi.</p>
	</div>

	<FooterNavigation className="mt-6" />
</footer>
