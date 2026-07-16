<script lang="ts">
	import { resolve } from '$app/paths';

	interface Props {
		className: string;
		isBus?: boolean;
	}

	let { className, isBus = true }: Props = $props();
</script>

<div class="flex flex-wrap gap-2 {className}">
	<a
		class="rounded-md bg-neutral-800 px-3 py-1 no-underline hover:bg-neutral-700"
		href={resolve('/')}
	>
		🚍 Lista fermate autobus
	</a>

	<a
		class="rounded-md bg-neutral-800 px-3 py-1 no-underline hover:bg-neutral-700"
		href={resolve('/treni')}
	>
		🚆 Lista stazioni treni
	</a>

	{#if isBus}
		<a
			class="rounded-md bg-neutral-800 px-3 py-1 no-underline hover:bg-neutral-700"
			href={resolve('/aiuto')}
		>
			❓ Istruzioni per trovare le fermate
		</a>
	{/if}

	<a
		class="rounded-md bg-neutral-800 px-3 py-1 no-underline hover:bg-neutral-700"
		href={resolve('/info')}
	>
		ℹ️ Informazioni
	</a>
</div>
