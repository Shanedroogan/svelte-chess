<!--
	Thin Svelte wrapper around chessground (@lichess-org/chessground),
	the framework-agnostic lichess.org board. Renders the board element,
	instantiates chessground on mount, and exposes its Api to the parent.

	The default board/piece styles are embedded in the component (see the
	note in the style block) rather than imported from the chessground
	assets, so that they are always bundled and hold a predictable
	specificity against consumer stylesheets.
-->
<script lang="ts">
	import { Chessground } from '@lichess-org/chessground';
	import type { Api as CgApi } from '@lichess-org/chessground/api';
	import type { Config } from '@lichess-org/chessground/config';
	import { onMount } from 'svelte';

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
	/*
	 * Chessground board, square and piece styles, adapted from the
	 * @lichess-org/chessground assets (chessground.base.css,
	 * chessground.brown.css, chessground.cburnett.css).
	 *
	 * All selectors are scoped under .board-wrap with their inner parts
	 * wrapped in :where(), pinning every rule to specificity (0,2,0):
	 * high enough to survive broad consumer theme resets (e.g.
	 * `body.dark * { background: ... }`), low enough that deliberate
	 * restyling via the class prop wins by including the runtime
	 * .cg-wrap class, e.g. `.my-theme.cg-wrap cg-board { ... }`.
	 */
	.board-wrap {
		width: 100%;
		aspect-ratio: 1 / 1;
		box-sizing: content-box;
		position: relative;
		display: block;
		container-type: inline-size;
	}

	/*
	 * Chessground's full-size child/overlay elements carry no background of
	 * their own, so a broad consumer reset could paint them and hide the
	 * board. Declare them transparent explicitly at the same specificity.
	 */
	.board-wrap :global(:where(cg-container, coords, coords coord, piece, piece svg, cg-auto-pieces, .cg-shapes, .cg-shapes-below, .cg-custom-svgs, .cg-custom-below)) {
		background-color: transparent;
	}

	/* --- chessground.base.css --- */
	.board-wrap :global(:where(cg-container)) {
		position: absolute;
		width: 100%;
		height: 100%;
		display: block;
		top: 0;
	}
	.board-wrap :global(:where(cg-board)) {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		-webkit-user-select: none;
		-moz-user-select: none;
		-ms-user-select: none;
		user-select: none;
		line-height: 0;
		background-size: cover;
	}
	.board-wrap:global(:where(.manipulable)) :global(:where(cg-board)) {
		cursor: pointer;
	}
	.board-wrap :global(:where(cg-board square)) {
		position: absolute;
		top: 0;
		left: 0;
		width: 12.5%;
		height: 12.5%;
		pointer-events: none;
	}
	.board-wrap :global(:where(cg-board square.move-dest)) {
		pointer-events: auto;
	}
	.board-wrap :global(:where(cg-board square.last-move)) {
		will-change: transform;
	}
	.board-wrap :global(:where(piece)) {
		position: absolute;
		top: 0;
		left: 0;
		width: 12.5%;
		height: 12.5%;
		background-size: cover;
		z-index: 2;
		will-change: transform;
		pointer-events: none;
	}
	.board-wrap :global(:where(cg-board piece.dragging)) {
		cursor: move;
		z-index: 11 !important;
	}
	.board-wrap :global(:where(piece.anim)) {
		z-index: 8;
	}
	.board-wrap :global(:where(piece.fading)) {
		z-index: 1;
		opacity: 0.5;
	}
	.board-wrap :global(:where(piece.ghost)) {
		opacity: 0.3;
	}
	.board-wrap :global(:where(piece svg)) {
		overflow: hidden;
		position: relative;
		top: 0px;
		left: 0px;
		width: 100%;
		height: 100%;
		pointer-events: none;
		z-index: 2;
		opacity: 0.6;
	}
	.board-wrap :global(:where(cg-auto-pieces)),
	.board-wrap :global(:where(.cg-shapes)),
	.board-wrap :global(:where(.cg-custom-svgs)),
	.board-wrap :global(:where(.cg-shapes-below)),
	.board-wrap :global(:where(.cg-custom-below)) {
		overflow: visible;
		position: absolute;
		top: 0px;
		left: 0px;
		width: 100%;
		height: 100%;
		pointer-events: none;
	}
	.board-wrap :global(:where(cg-auto-pieces)) {
		z-index: 2;
	}
	.board-wrap :global(:where(cg-auto-pieces piece)) {
		opacity: 0.3;
	}
	.board-wrap :global(:where(.cg-shapes)) {
		overflow: hidden;
		opacity: 0.6;
		z-index: 2;
	}
	.board-wrap :global(:where(.cg-shapes-below)) {
		overflow: hidden;
		z-index: 1;
	}
	.board-wrap :global(:where(.cg-custom-below)) {
		z-index: 1;
	}
	.board-wrap :global(:where(.cg-custom-svgs)) {
		z-index: 9;
	}
	.board-wrap :global(:where(.cg-custom-svgs svg)),
	.board-wrap :global(:where(.cg-custom-below svg)) {
		overflow: visible;
	}
	.board-wrap :global(:where(coords)) {
		position: absolute;
		display: flex;
		pointer-events: none;
		opacity: 1;
		font-family: sans-serif;
		font-size: 3.2cqw;
		line-height: 1;
		font-weight: 600;
	}
	.board-wrap :global(:where(coords.ranks)) {
		left: 0.1cqw;
		top: 0;
		flex-flow: column-reverse;
		height: 100%;
		width: auto;
	}
	.board-wrap :global(:where(coords.ranks.black)) {
		flex-flow: column;
	}
	.board-wrap :global(:where(coords.ranks.left)) {
		left: -15px;
		align-items: flex-end;
	}
	.board-wrap :global(:where(coords.files)) {
		bottom: 0.1cqw;
		left: 0;
		flex-flow: row;
		width: 100%;
		height: auto;
		text-transform: uppercase;
		text-align: right;
	}
	.board-wrap :global(:where(coords.files coord)) {
		padding-right: 0.8cqw;
	}
	.board-wrap :global(:where(coords.files.black)) {
		flex-flow: row-reverse;
	}
	.board-wrap :global(:where(coords coord)) {
		flex: 1 1 auto;
	}
	.board-wrap :global(:where(coords.ranks coord)) {
		transform: none;
		display: flex;
		align-items: flex-start;
		padding-top: 0.5cqw;
	}
	.board-wrap :global(:where(coords.squares)) {
		bottom: 0;
		left: 0;
		text-transform: uppercase;
		text-align: right;
		flex-flow: column-reverse;
		height: 100%;
		width: 12.5%;
	}
	.board-wrap :global(:where(coords.squares.black)) {
		flex-flow: column;
	}
	.board-wrap :global(:where(coords.squares.left)) {
		text-align: left;
	}
	.board-wrap :global(:where(coords.squares coord)) {
		padding: 6% 4%;
	}
	.board-wrap :global(:where(coords.squares.rank2)) {
		transform: translateX(100%);
	}
	.board-wrap :global(:where(coords.squares.rank3)) {
		transform: translateX(200%);
	}
	.board-wrap :global(:where(coords.squares.rank4)) {
		transform: translateX(300%);
	}
	.board-wrap :global(:where(coords.squares.rank5)) {
		transform: translateX(400%);
	}
	.board-wrap :global(:where(coords.squares.rank6)) {
		transform: translateX(500%);
	}
	.board-wrap :global(:where(coords.squares.rank7)) {
		transform: translateX(600%);
	}
	.board-wrap :global(:where(coords.squares.rank8)) {
		transform: translateX(700%);
	}
	/* --- chessground.brown.css --- */
	.board-wrap :global(:where(cg-board)) {
		background-color: #c9d6ec;
		background-image: url('data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iVVRGLTgiIHN0YW5kYWxvbmU9Im5vIj8+CjxzdmcgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB4bWxuczp4PSJodHRwOi8vd3d3LnczLm9yZy8xOTk5L3hsaW5rIgogICAgIHZpZXdCb3g9IjAgMCA4IDgiIHNoYXBlLXJlbmRlcmluZz0iY3Jpc3BFZGdlcyI+CjxnIGlkPSJhIj4KICA8ZyBpZD0iYiI+CiAgICA8ZyBpZD0iYyI+CiAgICAgIDxnIGlkPSJkIj4KICAgICAgICA8cmVjdCB3aWR0aD0iMSIgaGVpZ2h0PSIxIiBpZD0iZSIgb3BhY2l0eT0iMCIvPgogICAgICAgIDx1c2UgeD0iMSIgeT0iMSIgaHJlZj0iI2UiIHg6aHJlZj0iI2UiLz4KICAgICAgICA8cmVjdCB5PSIxIiB3aWR0aD0iMSIgaGVpZ2h0PSIxIiBpZD0iZiIgZmlsbD0iIzJiNGE3YyIvPgogICAgICAgIDx1c2UgeD0iMSIgeT0iLTEiIGhyZWY9IiNmIiB4OmhyZWY9IiNmIi8+CiAgICAgIDwvZz4KICAgICAgPHVzZSB4PSIyIiBocmVmPSIjZCIgeDpocmVmPSIjZCIvPgogICAgPC9nPgogICAgPHVzZSB4PSI0IiBocmVmPSIjYyIgeDpocmVmPSIjYyIvPgogIDwvZz4KICA8dXNlIHk9IjIiIGhyZWY9IiNiIiB4OmhyZWY9IiNiIi8+CjwvZz4KPHVzZSB5PSI0IiBocmVmPSIjYSIgeDpocmVmPSIjYSIvPgo8L3N2Zz4K');
	}
	.board-wrap :global(:where(cg-board square.move-dest)) {
		background: radial-gradient(rgba(20, 85, 30, 0.5) 22%, #208530 0, rgba(0, 0, 0, 0.3) 0, rgba(0, 0, 0, 0) 0);
	}
	.board-wrap :global(:where(cg-board square.premove-dest)) {
		background: radial-gradient(rgba(20, 30, 85, 0.5) 22%, #203085 0, rgba(0, 0, 0, 0.3) 0, rgba(0, 0, 0, 0) 0);
	}
	.board-wrap :global(:where(cg-board square.oc.move-dest)) {
		background: radial-gradient(transparent 0%, transparent 80%, rgba(20, 85, 0, 0.3) 80%);
	}
	.board-wrap :global(:where(cg-board square.oc.premove-dest)) {
		background: radial-gradient(transparent 0%, transparent 80%, rgba(20, 30, 85, 0.2) 80%);
	}
	.board-wrap :global(:where(cg-board square.move-dest:hover)),
	.board-wrap :global(:where(cg-board square.move-dest.hover)) {
		background: rgba(20, 85, 30, 0.3);
	}
	.board-wrap :global(:where(cg-board square.premove-dest:hover)),
	.board-wrap :global(:where(cg-board square.premove-dest.hover)) {
		background: rgba(20, 30, 85, 0.2);
	}
	.board-wrap :global(:where(cg-board square.last-move)) {
		background-color: rgba(96, 165, 250, 0.45);
	}
	.board-wrap :global(:where(cg-board square.selected)) {
		background-color: rgba(43, 85, 156, 0.55);
	}
	.board-wrap :global(:where(cg-board square.check)) {
		background: radial-gradient(
		ellipse at center,
		rgba(255, 0, 0, 1) 0%,
		rgba(231, 0, 0, 1) 25%,
		rgba(169, 0, 0, 0) 89%,
		rgba(158, 0, 0, 0) 100%
		);
	}
	.board-wrap :global(:where(cg-board square.current-premove)) {
		background-color: rgba(43, 85, 156, 0.55);
	}
	/* Color each coord by the square it actually sits on so contrast holds on
	   both square colors. Chessground tags every <coord> coord-light /
	   coord-dark for its DEFAULT edge: files render on that edge so their tags
	   are correct, but ranks render on the opposite (left / a-file) edge — and
	   the a/h files are always opposite colors — so the rank tags are inverted
	   and must be swapped back. (coords.squares, if ever enabled, is per-square
	   and needs no swap.) */
	.board-wrap :global(:where(coords.files coord.coord-light)),
	.board-wrap :global(:where(coords.squares coord.coord-light)),
	.board-wrap :global(:where(coords.ranks coord.coord-dark)) {
		color: #0c141e;
	}
	.board-wrap :global(:where(coords.files coord.coord-dark)),
	.board-wrap :global(:where(coords.squares coord.coord-dark)),
	.board-wrap :global(:where(coords.ranks coord.coord-light)) {
		color: #eaf1ff;
	}
	/* --- chessground.cburnett.css --- */
	.board-wrap :global(:where(piece.pawn.white)) {
		background-image: url('data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI0NSIgaGVpZ2h0PSI0NSI+PHBhdGggZD0iTTIyLjUgOWMtMi4yMSAwLTQgMS43OS00IDQgMCAuODkuMjkgMS43MS43OCAyLjM4QzE3LjMzIDE2LjUgMTYgMTguNTkgMTYgMjFjMCAyLjAzLjk0IDMuODQgMi40MSA1LjAzLTMgMS4wNi03LjQxIDUuNTUtNy40MSAxMy40N2gyM2MwLTcuOTItNC40MS0xMi40MS03LjQxLTEzLjQ3IDEuNDctMS4xOSAyLjQxLTMgMi40MS01LjAzIDAtMi40MS0xLjMzLTQuNS0zLjI4LTUuNjIuNDktLjY3Ljc4LTEuNDkuNzgtMi4zOCAwLTIuMjEtMS43OS00LTQtNHoiIGZpbGw9IiNmZmYiIHN0cm9rZT0iIzAwMCIgc3Ryb2tlLXdpZHRoPSIxLjUiIHN0cm9rZS1saW5lY2FwPSJyb3VuZCIvPjwvc3ZnPg==');
	}
	.board-wrap :global(:where(piece.bishop.white)) {
		background-image: url('data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI0NSIgaGVpZ2h0PSI0NSI+PGcgZmlsbD0ibm9uZSIgZmlsbC1ydWxlPSJldmVub2RkIiBzdHJva2U9IiMwMDAiIHN0cm9rZS13aWR0aD0iMS41IiBzdHJva2UtbGluZWNhcD0icm91bmQiIHN0cm9rZS1saW5lam9pbj0icm91bmQiPjxnIGZpbGw9IiNmZmYiIHN0cm9rZS1saW5lY2FwPSJidXR0Ij48cGF0aCBkPSJNOSAzNmMzLjM5LS45NyAxMC4xMS40MyAxMy41LTIgMy4zOSAyLjQzIDEwLjExIDEuMDMgMTMuNSAyIDAgMCAxLjY1LjU0IDMgMi0uNjguOTctMS42NS45OS0zIC41LTMuMzktLjk3LTEwLjExLjQ2LTEzLjUtMS0zLjM5IDEuNDYtMTAuMTEuMDMtMTMuNSAxLTEuMzU0LjQ5LTIuMzIzLjQ3LTMtLjUgMS4zNTQtMS45NCAzLTIgMy0yeiIvPjxwYXRoIGQ9Ik0xNSAzMmMyLjUgMi41IDEyLjUgMi41IDE1IDAgLjUtMS41IDAtMiAwLTIgMC0yLjUtMi41LTQtMi41LTQgNS41LTEuNSA2LTExLjUtNS0xNS41LTExIDQtMTAuNSAxNC01IDE1LjUgMCAwLTIuNSAxLjUtMi41IDQgMCAwLS41LjUgMCAyeiIvPjxwYXRoIGQ9Ik0yNSA4YTIuNSAyLjUgMCAxIDEtNSAwIDIuNSAyLjUgMCAxIDEgNSAweiIvPjwvZz48cGF0aCBkPSJNMTcuNSAyNmgxME0xNSAzMGgxNW0tNy41LTE0LjV2NU0yMCAxOGg1IiBzdHJva2UtbGluZWpvaW49Im1pdGVyIi8+PC9nPjwvc3ZnPg==');
	}
	.board-wrap :global(:where(piece.knight.white)) {
		background-image: url('data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI0NSIgaGVpZ2h0PSI0NSI+PGcgZmlsbD0ibm9uZSIgZmlsbC1ydWxlPSJldmVub2RkIiBzdHJva2U9IiMwMDAiIHN0cm9rZS13aWR0aD0iMS41IiBzdHJva2UtbGluZWNhcD0icm91bmQiIHN0cm9rZS1saW5lam9pbj0icm91bmQiPjxwYXRoIGQ9Ik0yMiAxMGMxMC41IDEgMTYuNSA4IDE2IDI5SDE1YzAtOSAxMC02LjUgOC0yMSIgZmlsbD0iI2ZmZiIvPjxwYXRoIGQ9Ik0yNCAxOGMuMzggMi45MS01LjU1IDcuMzctOCA5LTMgMi0yLjgyIDQuMzQtNSA0LTEuMDQyLS45NCAxLjQxLTMuMDQgMC0zLTEgMCAuMTkgMS4yMy0xIDItMSAwLTQuMDAzIDEtNC00IDAtMiA2LTEyIDYtMTJzMS44OS0xLjkgMi0zLjVjLS43My0uOTk0LS41LTItLjUtMyAxLTEgMyAyLjUgMyAyLjVoMnMuNzgtMS45OTIgMi41LTNjMSAwIDEgMyAxIDMiIGZpbGw9IiNmZmYiLz48cGF0aCBkPSJNOS41IDI1LjVhLjUuNSAwIDEgMS0xIDAgLjUuNSAwIDEgMSAxIDB6bTUuNDMzLTkuNzVhLjUgMS41IDMwIDEgMS0uODY2LS41LjUgMS41IDMwIDEgMSAuODY2LjV6IiBmaWxsPSIjMDAwIi8+PC9nPjwvc3ZnPg==');
	}
	.board-wrap :global(:where(piece.rook.white)) {
		background-image: url('data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI0NSIgaGVpZ2h0PSI0NSI+PGcgZmlsbD0iI2ZmZiIgZmlsbC1ydWxlPSJldmVub2RkIiBzdHJva2U9IiMwMDAiIHN0cm9rZS13aWR0aD0iMS41IiBzdHJva2UtbGluZWNhcD0icm91bmQiIHN0cm9rZS1saW5lam9pbj0icm91bmQiPjxwYXRoIGQ9Ik05IDM5aDI3di0zSDl2M3ptMy0zdi00aDIxdjRIMTJ6bS0xLTIyVjloNHYyaDVWOWg1djJoNVY5aDR2NSIgc3Ryb2tlLWxpbmVjYXA9ImJ1dHQiLz48cGF0aCBkPSJNMzQgMTRsLTMgM0gxNGwtMy0zIi8+PHBhdGggZD0iTTMxIDE3djEyLjVIMTRWMTciIHN0cm9rZS1saW5lY2FwPSJidXR0IiBzdHJva2UtbGluZWpvaW49Im1pdGVyIi8+PHBhdGggZD0iTTMxIDI5LjVsMS41IDIuNWgtMjBsMS41LTIuNSIvPjxwYXRoIGQ9Ik0xMSAxNGgyMyIgZmlsbD0ibm9uZSIgc3Ryb2tlLWxpbmVqb2luPSJtaXRlciIvPjwvZz48L3N2Zz4=');
	}
	.board-wrap :global(:where(piece.queen.white)) {
		background-image: url('data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI0NSIgaGVpZ2h0PSI0NSI+PGcgZmlsbD0iI2ZmZiIgZmlsbC1ydWxlPSJldmVub2RkIiBzdHJva2U9IiMwMDAiIHN0cm9rZS13aWR0aD0iMS41IiBzdHJva2UtbGluZWNhcD0icm91bmQiIHN0cm9rZS1saW5lam9pbj0icm91bmQiPjxwYXRoIGQ9Ik04IDEyYTIgMiAwIDEgMS00IDAgMiAyIDAgMSAxIDQgMHptMTYuNS00LjVhMiAyIDAgMSAxLTQgMCAyIDIgMCAxIDEgNCAwek00MSAxMmEyIDIgMCAxIDEtNCAwIDIgMiAwIDEgMSA0IDB6TTE2IDguNWEyIDIgMCAxIDEtNCAwIDIgMiAwIDEgMSA0IDB6TTMzIDlhMiAyIDAgMSAxLTQgMCAyIDIgMCAxIDEgNCAweiIvPjxwYXRoIGQ9Ik05IDI2YzguNS0xLjUgMjEtMS41IDI3IDBsMi0xMi03IDExVjExbC01LjUgMTMuNS0zLTE1LTMgMTUtNS41LTE0VjI1TDcgMTRsMiAxMnoiIHN0cm9rZS1saW5lY2FwPSJidXR0Ii8+PHBhdGggZD0iTTkgMjZjMCAyIDEuNSAyIDIuNSA0IDEgMS41IDEgMSAuNSAzLjUtMS41IDEtMS41IDIuNS0xLjUgMi41LTEuNSAxLjUuNSAyLjUuNSAyLjUgNi41IDEgMTYuNSAxIDIzIDAgMCAwIDEuNS0xIDAtMi41IDAgMCAuNS0xLjUtMS0yLjUtLjUtMi41LS41LTIgLjUtMy41IDEtMiAyLjUtMiAyLjUtNC04LjUtMS41LTE4LjUtMS41LTI3IDB6IiBzdHJva2UtbGluZWNhcD0iYnV0dCIvPjxwYXRoIGQ9Ik0xMS41IDMwYzMuNS0xIDE4LjUtMSAyMiAwTTEyIDMzLjVjNi0xIDE1LTEgMjEgMCIgZmlsbD0ibm9uZSIvPjwvZz48L3N2Zz4=');
	}
	.board-wrap :global(:where(piece.king.white)) {
		background-image: url('data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI0NSIgaGVpZ2h0PSI0NSI+PGcgZmlsbD0ibm9uZSIgZmlsbC1ydWxlPSJldmVub2RkIiBzdHJva2U9IiMwMDAiIHN0cm9rZS13aWR0aD0iMS41IiBzdHJva2UtbGluZWNhcD0icm91bmQiIHN0cm9rZS1saW5lam9pbj0icm91bmQiPjxwYXRoIGQ9Ik0yMi41IDExLjYzVjZNMjAgOGg1IiBzdHJva2UtbGluZWpvaW49Im1pdGVyIi8+PHBhdGggZD0iTTIyLjUgMjVzNC41LTcuNSAzLTEwLjVjMCAwLTEtMi41LTMtMi41cy0zIDIuNS0zIDIuNWMtMS41IDMgMyAxMC41IDMgMTAuNSIgZmlsbD0iI2ZmZiIgc3Ryb2tlLWxpbmVjYXA9ImJ1dHQiIHN0cm9rZS1saW5lam9pbj0ibWl0ZXIiLz48cGF0aCBkPSJNMTEuNSAzN2M1LjUgMy41IDE1LjUgMy41IDIxIDB2LTdzOS00LjUgNi0xMC41Yy00LTYuNS0xMy41LTMuNS0xNiA0VjI3di0zLjVjLTMuNS03LjUtMTMtMTAuNS0xNi00LTMgNiA1IDEwIDUgMTBWMzd6IiBmaWxsPSIjZmZmIi8+PHBhdGggZD0iTTExLjUgMzBjNS41LTMgMTUuNS0zIDIxIDBtLTIxIDMuNWM1LjUtMyAxNS41LTMgMjEgMG0tMjEgMy41YzUuNS0zIDE1LjUtMyAyMSAwIi8+PC9nPjwvc3ZnPg==');
	}
	.board-wrap :global(:where(piece.pawn.black)) {
		background-image: url('data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI0NSIgaGVpZ2h0PSI0NSI+PHBhdGggZD0iTTIyLjUgOWMtMi4yMSAwLTQgMS43OS00IDQgMCAuODkuMjkgMS43MS43OCAyLjM4QzE3LjMzIDE2LjUgMTYgMTguNTkgMTYgMjFjMCAyLjAzLjk0IDMuODQgMi40MSA1LjAzLTMgMS4wNi03LjQxIDUuNTUtNy40MSAxMy40N2gyM2MwLTcuOTItNC40MS0xMi40MS03LjQxLTEzLjQ3IDEuNDctMS4xOSAyLjQxLTMgMi40MS01LjAzIDAtMi40MS0xLjMzLTQuNS0zLjI4LTUuNjIuNDktLjY3Ljc4LTEuNDkuNzgtMi4zOCAwLTIuMjEtMS43OS00LTQtNHoiIHN0cm9rZT0iIzAwMCIgc3Ryb2tlLXdpZHRoPSIxLjUiIHN0cm9rZS1saW5lY2FwPSJyb3VuZCIvPjwvc3ZnPg==');
	}
	.board-wrap :global(:where(piece.bishop.black)) {
		background-image: url('data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI0NSIgaGVpZ2h0PSI0NSI+PGcgZmlsbD0ibm9uZSIgZmlsbC1ydWxlPSJldmVub2RkIiBzdHJva2U9IiMwMDAiIHN0cm9rZS13aWR0aD0iMS41IiBzdHJva2UtbGluZWNhcD0icm91bmQiIHN0cm9rZS1saW5lam9pbj0icm91bmQiPjxnIGZpbGw9IiMwMDAiIHN0cm9rZS1saW5lY2FwPSJidXR0Ij48cGF0aCBkPSJNOSAzNmMzLjM5LS45NyAxMC4xMS40MyAxMy41LTIgMy4zOSAyLjQzIDEwLjExIDEuMDMgMTMuNSAyIDAgMCAxLjY1LjU0IDMgMi0uNjguOTctMS42NS45OS0zIC41LTMuMzktLjk3LTEwLjExLjQ2LTEzLjUtMS0zLjM5IDEuNDYtMTAuMTEuMDMtMTMuNSAxLTEuMzU0LjQ5LTIuMzIzLjQ3LTMtLjUgMS4zNTQtMS45NCAzLTIgMy0yeiIvPjxwYXRoIGQ9Ik0xNSAzMmMyLjUgMi41IDEyLjUgMi41IDE1IDAgLjUtMS41IDAtMiAwLTIgMC0yLjUtMi41LTQtMi41LTQgNS41LTEuNSA2LTExLjUtNS0xNS41LTExIDQtMTAuNSAxNC01IDE1LjUgMCAwLTIuNSAxLjUtMi41IDQgMCAwLS41LjUgMCAyeiIvPjxwYXRoIGQ9Ik0yNSA4YTIuNSAyLjUgMCAxIDEtNSAwIDIuNSAyLjUgMCAxIDEgNSAweiIvPjwvZz48cGF0aCBkPSJNMTcuNSAyNmgxME0xNSAzMGgxNW0tNy41LTE0LjV2NU0yMCAxOGg1IiBzdHJva2U9IiNlY2VjZWMiIHN0cm9rZS1saW5lam9pbj0ibWl0ZXIiLz48L2c+PC9zdmc+');
	}
	.board-wrap :global(:where(piece.knight.black)) {
		background-image: url('data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI0NSIgaGVpZ2h0PSI0NSI+PGcgZmlsbD0ibm9uZSIgZmlsbC1ydWxlPSJldmVub2RkIiBzdHJva2U9IiMwMDAiIHN0cm9rZS13aWR0aD0iMS41IiBzdHJva2UtbGluZWNhcD0icm91bmQiIHN0cm9rZS1saW5lam9pbj0icm91bmQiPjxwYXRoIGQ9Ik0yMiAxMGMxMC41IDEgMTYuNSA4IDE2IDI5SDE1YzAtOSAxMC02LjUgOC0yMSIgZmlsbD0iIzAwMCIvPjxwYXRoIGQ9Ik0yNCAxOGMuMzggMi45MS01LjU1IDcuMzctOCA5LTMgMi0yLjgyIDQuMzQtNSA0LTEuMDQyLS45NCAxLjQxLTMuMDQgMC0zLTEgMCAuMTkgMS4yMy0xIDItMSAwLTQuMDAzIDEtNC00IDAtMiA2LTEyIDYtMTJzMS44OS0xLjkgMi0zLjVjLS43My0uOTk0LS41LTItLjUtMyAxLTEgMyAyLjUgMyAyLjVoMnMuNzgtMS45OTIgMi41LTNjMSAwIDEgMyAxIDMiIGZpbGw9IiMwMDAiLz48cGF0aCBkPSJNOS41IDI1LjVhLjUuNSAwIDEgMS0xIDAgLjUuNSAwIDEgMSAxIDB6bTUuNDMzLTkuNzVhLjUgMS41IDMwIDEgMS0uODY2LS41LjUgMS41IDMwIDEgMSAuODY2LjV6IiBmaWxsPSIjZWNlY2VjIiBzdHJva2U9IiNlY2VjZWMiLz48cGF0aCBkPSJNMjQuNTUgMTAuNGwtLjQ1IDEuNDUuNS4xNWMzLjE1IDEgNS42NSAyLjQ5IDcuOSA2Ljc1UzM1Ljc1IDI5LjA2IDM1LjI1IDM5bC0uMDUuNWgyLjI1bC4wNS0uNWMuNS0xMC4wNi0uODgtMTYuODUtMy4yNS0yMS4zNC0yLjM3LTQuNDktNS43OS02LjY0LTkuMTktNy4xNmwtLjUxLS4xeiIgZmlsbD0iI2VjZWNlYyIgc3Ryb2tlPSJub25lIi8+PC9nPjwvc3ZnPg==');
	}
	.board-wrap :global(:where(piece.rook.black)) {
		background-image: url('data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI0NSIgaGVpZ2h0PSI0NSI+PGcgZmlsbC1ydWxlPSJldmVub2RkIiBzdHJva2U9IiMwMDAiIHN0cm9rZS13aWR0aD0iMS41IiBzdHJva2UtbGluZWNhcD0icm91bmQiIHN0cm9rZS1saW5lam9pbj0icm91bmQiPjxwYXRoIGQ9Ik05IDM5aDI3di0zSDl2M3ptMy41LTdsMS41LTIuNWgxN2wxLjUgMi41aC0yMHptLS41IDR2LTRoMjF2NEgxMnoiIHN0cm9rZS1saW5lY2FwPSJidXR0Ii8+PHBhdGggZD0iTTE0IDI5LjV2LTEzaDE3djEzSDE0eiIgc3Ryb2tlLWxpbmVjYXA9ImJ1dHQiIHN0cm9rZS1saW5lam9pbj0ibWl0ZXIiLz48cGF0aCBkPSJNMTQgMTYuNUwxMSAxNGgyM2wtMyAyLjVIMTR6TTExIDE0VjloNHYyaDVWOWg1djJoNVY5aDR2NUgxMXoiIHN0cm9rZS1saW5lY2FwPSJidXR0Ii8+PHBhdGggZD0iTTEyIDM1LjVoMjFtLTIwLTRoMTltLTE4LTJoMTdtLTE3LTEzaDE3TTExIDE0aDIzIiBmaWxsPSJub25lIiBzdHJva2U9IiNlY2VjZWMiIHN0cm9rZS13aWR0aD0iMSIgc3Ryb2tlLWxpbmVqb2luPSJtaXRlciIvPjwvZz48L3N2Zz4=');
	}
	.board-wrap :global(:where(piece.queen.black)) {
		background-image: url('data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI0NSIgaGVpZ2h0PSI0NSI+PGcgZmlsbC1ydWxlPSJldmVub2RkIiBzdHJva2U9IiMwMDAiIHN0cm9rZS13aWR0aD0iMS41IiBzdHJva2UtbGluZWNhcD0icm91bmQiIHN0cm9rZS1saW5lam9pbj0icm91bmQiPjxnIHN0cm9rZT0ibm9uZSI+PGNpcmNsZSBjeD0iNiIgY3k9IjEyIiByPSIyLjc1Ii8+PGNpcmNsZSBjeD0iMTQiIGN5PSI5IiByPSIyLjc1Ii8+PGNpcmNsZSBjeD0iMjIuNSIgY3k9IjgiIHI9IjIuNzUiLz48Y2lyY2xlIGN4PSIzMSIgY3k9IjkiIHI9IjIuNzUiLz48Y2lyY2xlIGN4PSIzOSIgY3k9IjEyIiByPSIyLjc1Ii8+PC9nPjxwYXRoIGQ9Ik05IDI2YzguNS0xLjUgMjEtMS41IDI3IDBsMi41LTEyLjVMMzEgMjVsLS4zLTE0LjEtNS4yIDEzLjYtMy0xNC41LTMgMTQuNS01LjItMTMuNkwxNCAyNSA2LjUgMTMuNSA5IDI2eiIgc3Ryb2tlLWxpbmVjYXA9ImJ1dHQiLz48cGF0aCBkPSJNOSAyNmMwIDIgMS41IDIgMi41IDQgMSAxLjUgMSAxIC41IDMuNS0xLjUgMS0xLjUgMi41LTEuNSAyLjUtMS41IDEuNS41IDIuNS41IDIuNSA2LjUgMSAxNi41IDEgMjMgMCAwIDAgMS41LTEgMC0yLjUgMCAwIC41LTEuNS0xLTIuNS0uNS0yLjUtLjUtMiAuNS0zLjUgMS0yIDIuNS0yIDIuNS00LTguNS0xLjUtMTguNS0xLjUtMjcgMHoiIHN0cm9rZS1saW5lY2FwPSJidXR0Ii8+PHBhdGggZD0iTTExIDM4LjVhMzUgMzUgMSAwIDAgMjMgMCIgZmlsbD0ibm9uZSIgc3Ryb2tlLWxpbmVjYXA9ImJ1dHQiLz48cGF0aCBkPSJNMTEgMjlhMzUgMzUgMSAwIDEgMjMgMG0tMjEuNSAyLjVoMjBtLTIxIDNhMzUgMzUgMSAwIDAgMjIgMG0tMjMgM2EzNSAzNSAxIDAgMCAyNCAwIiBmaWxsPSJub25lIiBzdHJva2U9IiNlY2VjZWMiLz48L2c+PC9zdmc+');
	}
	.board-wrap :global(:where(piece.king.black)) {
		background-image: url('data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI0NSIgaGVpZ2h0PSI0NSI+PGcgZmlsbD0ibm9uZSIgZmlsbC1ydWxlPSJldmVub2RkIiBzdHJva2U9IiMwMDAiIHN0cm9rZS13aWR0aD0iMS41IiBzdHJva2UtbGluZWNhcD0icm91bmQiIHN0cm9rZS1saW5lam9pbj0icm91bmQiPjxwYXRoIGQ9Ik0yMi41IDExLjYzVjYiIHN0cm9rZS1saW5lam9pbj0ibWl0ZXIiLz48cGF0aCBkPSJNMjIuNSAyNXM0LjUtNy41IDMtMTAuNWMwIDAtMS0yLjUtMy0yLjVzLTMgMi41LTMgMi41Yy0xLjUgMyAzIDEwLjUgMyAxMC41IiBmaWxsPSIjMDAwIiBzdHJva2UtbGluZWNhcD0iYnV0dCIgc3Ryb2tlLWxpbmVqb2luPSJtaXRlciIvPjxwYXRoIGQ9Ik0xMS41IDM3YzUuNSAzLjUgMTUuNSAzLjUgMjEgMHYtN3M5LTQuNSA2LTEwLjVjLTQtNi41LTEzLjUtMy41LTE2IDRWMjd2LTMuNWMtMy41LTcuNS0xMy0xMC41LTE2LTQtMyA2IDUgMTAgNSAxMFYzN3oiIGZpbGw9IiMwMDAiLz48cGF0aCBkPSJNMjAgOGg1IiBzdHJva2UtbGluZWpvaW49Im1pdGVyIi8+PHBhdGggZD0iTTMyIDI5LjVzOC41LTQgNi4wMy05LjY1QzM0LjE1IDE0IDI1IDE4IDIyLjUgMjQuNWwuMDEgMi4xLS4wMS0yLjFDMjAgMTggOS45MDYgMTQgNi45OTcgMTkuODVjLTIuNDk3IDUuNjUgNC44NTMgOSA0Ljg1MyA5IiBzdHJva2U9IiNlY2VjZWMiLz48cGF0aCBkPSJNMTEuNSAzMGM1LjUtMyAxNS41LTMgMjEgMG0tMjEgMy41YzUuNS0zIDE1LjUtMyAyMSAwbS0yMSAzLjVjNS41LTMgMTUuNS0zIDIxIDAiIHN0cm9rZT0iI2VjZWNlYyIvPjwvZz48L3N2Zz4=');
	}
</style>
