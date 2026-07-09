<script lang="ts">
	import Chess from '$lib/Chess.svelte';
	let chess: ReturnType<typeof Chess> | undefined;
</script>

<div style="max-width:512px;margin:0 auto;">
	<Chess bind:this={chess} />
	<button onclick={()=>chess?.reset()}>Reset board</button>
	<button onclick={()=>chess?.undo()}>Undo</button>
	<button onclick={()=>chess?.toggleOrientation()}>Flip board</button>
</div>

<style>
	button {
		margin-top:16px;
	}
</style>
