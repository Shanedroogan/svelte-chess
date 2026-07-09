<script lang="ts">
	import Chess, { type Color } from '$lib/Chess.svelte';
	// Svelte 5: bound props with fallback values may not be bound to undefined,
	// so the initial values are provided here.
	let fen: string = $state('rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR w KQkq - 0 1'),
		moveNumber: number = $state(0),
		turn: Color = $state('w'),
		history: string[] = $state([]),
		inCheck: boolean = $state(false);
</script>

<div style="max-width:512px;margin:0 auto;">
	<Chess bind:fen bind:moveNumber bind:turn bind:history bind:inCheck />
</div>

{#if inCheck}
	<p>Check!</p>
{/if}
<p>Move {moveNumber}, {turn == 'w' ? 'White' : 'Black'} to move.</p>
{#if history?.length > 0}
	<p>Moves: {history.join(' ')}</p>
{/if}
<p style="white-space:nowrap;">FEN: {fen}</p>

<style>
	div, p {
		max-width:512px;
		margin:8px auto;
	}
</style>
