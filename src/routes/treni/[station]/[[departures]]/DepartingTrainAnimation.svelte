<script lang="ts">
	import { onMount } from 'svelte';

	let el: HTMLSpanElement;

	function syncAnimation() {
		el.getAnimations().forEach((a) => (a.startTime = 0));
	}

	onMount(() => {
		syncAnimation();
	});
</script>

<span
	bind:this={el}
	onanimationstart={syncAnimation}
	class="block h-2 w-2 animate-train rounded-full bg-white"
></span>
