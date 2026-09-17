<script lang="ts">
	import { onMount } from 'svelte';
	import { BadgeAlert as Alert } from '@lucide/svelte';
	import type { Trip } from '$lib/Trip';

	interface Props {
		trip: Trip;
	}

	let { trip }: Props = $props();
	
	const stopElements: (HTMLDivElement | undefined)[] = $state([]);
	let lastUpdated = $state(timeAgo())

	onMount(() => {
		// On load, show the last passed stop in the middle so it's easier to see.
		// In case no live data is available, just have the current stop in the middle.
		const stopNumber =
			trip.delay === null ? trip.userStopSequenceNumber : trip.currentStopSequenceNumber;
		const elementToScroll = stopElements[stopNumber - 1];
		if (elementToScroll) {
			elementToScroll.scrollIntoView({ block: 'center' });
		}
	});

	// do not bother updating periodically if it's more than 1 min old
	if (Date.now() - trip.lastUpdatedTimestamp < 1000 * 60) {
		const interval = setInterval(() => {
			lastUpdated = timeAgo();
		}, 5_000);

		$effect(() => {
			return () => clearInterval(interval);
		});
	}


	function timeAgo() {
		const seconds = Math.floor((Date.now() - trip.lastUpdatedTimestamp) / 1000);

		// tound to nearest 5s
		if (seconds < 60){
			return `${Math.ceil(seconds / 5) * 5}s`;
		}

		const minutes = Math.floor(seconds / 60);
		return `${minutes} min`;
	}
</script>

<!-- This wrapper is needed to be able to add a bottom padding and avoid the slide transition jerkiness -->
<div class="pt-1 pb-3">
	<div class="rounded-lg border border-neutral-700 bg-neutral-800">
		<div class="flex justify-between px-3 py-1.5">
			{#if trip.vehicleId}
				<span class="font-bold">Bus {trip.vehicleId}</span>
			{:else}
				<span class="font-semibold italic">Dati non disponibili</span>
			{/if}
			<Alert></Alert>
			{#if trip.lastUpdatedTimestamp !== 0}
				<span class="font-light">Aggiornato {lastUpdated} fa</span>
			{/if}
		</div>

		<div class="flex max-h-50 flex-col gap-y-2.5 overflow-y-auto px-4 py-3">
			<!-- eslint-disable-next-line svelte/require-each-key -->
			{#each trip.stopTimes as stopTime, i}
				{@const wasPassed = i < trip.currentStopSequenceNumber}
				<div bind:this={stopElements[i]} class="flex items-center gap-x-4">
					<div class="w-10 leading-none font-semibold">
						{stopTime.time}
					</div>

					<div class="relative flex flex-col items-center">
						<span
							class="relative z-10 size-4 rounded-full {wasPassed
								? 'border-[1.5px] border-neutral-100'
								: 'bg-neutral-100'}"
							style:background-color={wasPassed ? trip.routeColor : ''}
						></span>

						<!-- Show the vertical line connecting to the next stop.
								 Since the trip is removed from the list once the bus reaches the end of route,
								 there is no need to hide the connecting element if it's the last one. -->
						{#if wasPassed}
							<div class="absolute top-3 h-4 w-1.5" style:background-color={trip.routeColor}></div>
						{/if}
					</div>

					<div class="leading-none whitespace-nowrap">
						{#if i === trip.userStopSequenceNumber - 1}
							<span class="font-semibold">📍 La tua fermata</span>
						{:else}
							{stopTime.name}
						{/if}
					</div>
				</div>
			{/each}
		</div>
	</div>
</div>
