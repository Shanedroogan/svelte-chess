<script lang="ts">
	import Chess, { type Move, type GameOver } from '$lib/Chess.svelte';
	import { flip } from 'svelte/animate' ;
	import { fade } from 'svelte/transition';

	let messages: {title:string, details:string}[] = $state([]);
	function moveHandler( move: Move ) {
		messages.unshift( {
			title: "move: " + move.san,
			details: JSON.stringify(move, null, 2)
		} );
	}
	function gameOverHandler( gameOver: GameOver ) {
		messages.unshift( {
			title: "gameOver: " + gameOver.reason,
			details: JSON.stringify(gameOver, null, 2)
		} );
	}
	function readyHandler() {
		messages.unshift( {
			title: "ready",
			details: ""
		} );
	}
</script>

<div style="max-width:512px;margin:0 auto;">
	<p>This example listens for <code>move</code> and <code>gameOver</code> callbacks.</p>
	<Chess onmove={moveHandler} ongameOver={gameOverHandler} onready={readyHandler} />
</div>
	<div class="messages">
		{#each messages as message (message.details)}
			<div animate:flip in:fade title="{message.details}">
				{message.title}
			</div>
		{/each}
	</div>

<style>
	div.messages {
		margin-top:16px;
		display:flex;
		flex-wrap:wrap;
		gap:8px;
		justify-content:center;
	}
	div.messages div {
		padding:8px 12px;
		border-radius:8px;
		background-color: #f0d9b5;
	}
</style>
