<!--
	Thin Svelte wrapper around chessground (@lichess-org/chessground),
	the framework-agnostic lichess.org board. Renders the board element,
	instantiates chessground on mount, and exposes its Api to the parent.
-->
<script lang="ts">
	import { Chessground } from '@lichess-org/chessground';
	import type { Api as CgApi } from '@lichess-org/chessground/api';
	import type { Config } from '@lichess-org/chessground/config';
	import { onMount } from 'svelte';
	import '@lichess-org/chessground/assets/chessground.base.css';
	import '@lichess-org/chessground/assets/chessground.brown.css';
	import '@lichess-org/chessground/assets/chessground.cburnett.css';

	let { class: className = undefined, config = {} }: { class?: string, config?: Config } = $props();

	let el: HTMLElement;
	let cg: CgApi | undefined;

	// Chessground Api, for direct board manipulation. Only available after mount.
	export function getApi(): CgApi {
		if ( ! cg ) throw new Error( 'board not mounted yet' );
		return cg;
	}

	onMount( () => {
		cg = Chessground( el, config );
		return () => cg?.destroy();
	} );
</script>

<div class="board-wrap {className ?? ''}" bind:this={el}></div>

<style>
	.board-wrap {
		width: 100%;
		aspect-ratio: 1 / 1;
	}
</style>
