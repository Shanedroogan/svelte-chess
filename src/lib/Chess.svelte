<script lang="ts" module>
	export type { Square, Color, PieceSymbol, Move, GameOver } from '$lib/api.js';
	export { Engine } from '$lib/engine.js';
</script>
<script lang="ts">
	import Board from '$lib/Board.svelte';
	import PromotionDialog from '$lib/PromotionDialog.svelte';
	import { Api, type Square, type Color, type PieceSymbol, type Move, type GameOver } from '$lib/api.js';
	import type { Engine } from '$lib/engine.js';

	import { onMount, mount, unmount } from 'svelte';

	interface Props {
		// bindable read-only props
		moveNumber?: number;
		turn?: Color;
		inCheck?: boolean;
		history?: string[];
		isGameOver?: boolean;
		// initial values used, also bindable
		fen?: string;
		orientation?: Color;
		// non-bindable
		engine?: Engine;
		class?: string;
		// event callbacks
		onmove?: ( move: Move ) => void;
		ongameOver?: ( gameOver: GameOver ) => void;
		onready?: () => void;
		onuci?: ( message: string ) => void;
	}

	let {
		moveNumber = $bindable(0),
		turn = $bindable('w'),
		inCheck = $bindable(false),
		history = $bindable([]),
		isGameOver = $bindable(false),
		fen = $bindable('rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR w KQkq - 0 1'),
		orientation = $bindable('w'),
		engine = undefined,
		class: className = undefined,
		onmove = undefined,
		ongameOver = undefined,
		onready = undefined,
		onuci = undefined,
	}: Props = $props();

	let board: ReturnType<typeof Board>;
	let container: HTMLElement;

	// API: only accessible through props and methods
	let api: Api | undefined = undefined;

	/*
	 * Methods -- passed to API
	 */

	export function load(newFen: string) {
		if ( ! api ) throw new Error( 'component not mounted yet' );
		api.load( newFen );
	}
	export function move(moveSan: string) {
		if ( ! api ) throw new Error( 'component not mounted yet' );
		api.move(moveSan);
	}
	export function getHistory(): string[]
	export function getHistory({ verbose }: { verbose: true }): Move[]
	export function getHistory({ verbose }: { verbose: false }): string[]
	export function getHistory({ verbose }: { verbose: boolean }): string[] | Move[]
	export function getHistory({ verbose = false }: { verbose?: boolean } = {}) {
		if ( ! api ) throw new Error( 'component not mounted yet' );
		return api.history({verbose});
	}
	export function getBoard() {
		if ( ! api ) throw new Error( 'component not mounted yet' );
		return api.board();
	}
	export function undo(): Move | null {
		if ( ! api ) throw new Error( 'component not mounted yet' );
		return api.undo();
	}
	export function reset(): void {
		if ( ! api ) throw new Error( 'component not mounted yet' );
		api.reset();
	}
	export function toggleOrientation(): void {
		if ( ! api ) throw new Error( 'component not mounted yet' );
		api.toggleOrientation();
	}
	export async function playEngineMove(): Promise<void> {
		if ( ! api ) throw new Error( 'component not mounted yet' );
		return api.playEngineMove();
	}

	/*
	 * API Construction
	 */

	function stateChangeCallback(api: Api) {
		fen = api.fen();
		orientation = api.orientation();
		moveNumber = api.moveNumber();
		turn = api.turn();
		inCheck = api.inCheck();
		history = api.history();
		isGameOver = api.isGameOver();
	}

	function promotionCallback( square: Square ): Promise<PieceSymbol> {
		return new Promise((resolve) => {
			const dialog = mount( PromotionDialog, {
				target: container,
				props: {
					square,
					orientation,
					callback: (piece: PieceSymbol) => {
						unmount( dialog );
						resolve( piece );
					}
				},
			});
		});
	}

	function moveCallback( move: Move ) {
		onmove?.( move );
	}
	function gameOverCallback( gameOver: GameOver ) {
		ongameOver?.( gameOver );
	}

	onMount( async () => {
		if ( engine ) {
			engine.setUciCallback( (message) => onuci?.( message ) );
		}
		api = new Api( board.getApi(), fen, stateChangeCallback, promotionCallback, moveCallback, gameOverCallback, orientation, engine );
		api.init().then( () => {
			// Call onready: Simply letting the parent observe when the component is mounted is not enough due to async onMount.
			onready?.();
		} );
	} );

</script>

<div style="position:relative;" bind:this={container}>
	<Board bind:this={board} class={className}/>
</div>
