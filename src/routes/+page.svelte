<script lang="ts">
	import Chess, { Engine } from '$lib/Chess.svelte';
	let chess: ReturnType<typeof Chess> | undefined;
</script>

<div style="max-width:512px;margin:0 auto;">
	<Chess
		bind:this={chess}
		orientation='b'
		engine={new Engine({depth: 20, moveTime: 1500, color: 'w'})}
	/>
	<button onclick={()=>chess?.reset()}>Reset board</button>
	<button onclick={()=>chess?.undo()}>Undo</button>
	<button onclick={()=>chess?.playEngineMove()}>Play engine move</button>
</div>

<style>
	button {
		margin-top:16px;
	}
</style>
