<script>
	// @ts-nocheck

	import { onMount } from 'svelte';
	import { fly, scale, fade } from 'svelte/transition';
	import { quintOut, elasticOut } from 'svelte/easing';

	// 0 = empty
	let puzzle = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 0];
	let isSolved = false;
	let moveCount = 0;

	function getValidMoves(emptyIndex) {
		const row = Math.floor(emptyIndex / 4);
		const col = emptyIndex % 4;
		const moves = [];

		if (row > 0) moves.push(emptyIndex - 4);
		if (row < 3) moves.push(emptyIndex + 4);
		if (col > 0) moves.push(emptyIndex - 1);
		if (col < 3) moves.push(emptyIndex + 1);

		return moves;
	}

	function checkSolved() {
		const solved = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 0];
		isSolved = puzzle.every((v, i) => v === solved[i]);
	}

	function shufflePuzzle() {
		const shuffled = [...puzzle];

		// many valid moves from solved => always solvable
		for (let i = 0; i < 1000; i++) {
			const emptyIndex = shuffled.indexOf(0);
			const validMoves = getValidMoves(emptyIndex);
			const pick = validMoves[Math.floor(Math.random() * validMoves.length)];
			[shuffled[emptyIndex], shuffled[pick]] = [shuffled[pick], shuffled[emptyIndex]];
		}

		puzzle = shuffled;
		isSolved = false;
		moveCount = 0;
	}

	function handleTileClick(index) {
		if (isSolved) return;

		const emptyIndex = puzzle.indexOf(0);
		const validMoves = getValidMoves(emptyIndex);

		if (!validMoves.includes(index)) return;

		const next = [...puzzle];
		[next[emptyIndex], next[index]] = [next[index], next[emptyIndex]];
		puzzle = next;
		moveCount += 1;
		checkSolved();
	}

	function resetPuzzle() {
		puzzle = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 0];
		isSolved = true;
		moveCount = 0;
	}

	onMount(shufflePuzzle);
</script>

<svelte:head>
	<title>15 Puzzle Game</title>
	<meta name="description" content="Play the classic 15 puzzle sliding game" />
</svelte:head>

<div class="app">
	<div class="container">
		<section class="stats" in:fly={{ y: -20, duration: 500, delay: 200 }}>
			<div class="moves">
				<span class="movesLabel">Moves:</span>
				<span class="movesValue">{moveCount}</span>
			</div>

			{#if isSolved}
				<div class="solved" in:scale={{ duration: 500, easing: elasticOut }}>🎉 Puzzle Solved!</div>
			{/if}
		</section>

		<section class="boardWrap" in:scale={{ duration: 600, delay: 300 }}>
			<div class="card">
				<div class="board" aria-label="15 puzzle board">
					{#each puzzle as tile, index (index)}
						<button
							class="tile {tile === 0 ? 'empty' : ''}"
							onclick={() => handleTileClick(index)}
							disabled={isSolved}
							aria-label={tile === 0 ? 'Empty space' : `Tile ${tile}`}
							in:fly={{
								x: Math.random() * 40 - 20,
								y: Math.random() * 40 - 20,
								duration: 400,
								delay: index * 50,
								easing: quintOut
							}}
						>
							{tile === 0 ? '' : tile}
						</button>
					{/each}
				</div>
			</div>
		</section>

		<section class="controls" in:fly={{ y: 20, duration: 500, delay: 600 }}>
			<button class="btn btnPrimary" onclick={shufflePuzzle}>New Game</button>
			<button class="btn btnOutline" onclick={resetPuzzle}>Show Solution</button>
		</section>

		<section class="instructions" in:fade={{ duration: 600, delay: 800 }}>
			<div class="card cardSoft">
				<h2>How to Play</h2>
				<ul>
					<li><span class="dot">•</span> Click a tile next to the empty space to move it</li>
					<li><span class="dot">•</span> Arrange tiles 1–15 with the empty space bottom-right</li>
					<li><span class="dot">•</span> Try to solve it in as few moves as possible</li>
				</ul>
			</div>
		</section>
	</div>
</div>

<style>
	:global(html, body) {
		height: 100%;
		margin: 0;
		font-family:
			system-ui,
			-apple-system,
			Segoe UI,
			Roboto,
			Helvetica,
			Arial;
		color: #1f2937;
	}

	.app {
		min-height: 100vh;
		padding: 16px;
		display: flex;
		justify-content: center;
		background: linear-gradient(135deg, #eff6ff 0%, #eef2ff 45%, #faf5ff 100%);
	}

	.container {
		width: 100%;
		max-width: 680px;
	}

	.header {
		text-align: center;
		margin: 10px 0 24px;
	}

	.header h1 {
		margin: 0 0 6px;
		font-size: 2.4rem;
		font-weight: 800;
		letter-spacing: -0.02em;
	}

	.header p {
		margin: 0;
		color: #4b5563;
		line-height: 1.4;
	}

	.stats {
		display: flex;
		justify-content: center;
		gap: 12px;
		align-items: center;
		margin: 0 0 18px;
		flex-wrap: wrap;
	}

	.moves {
		display: inline-flex;
		gap: 8px;
		align-items: baseline;
		padding: 10px 14px;
		border-radius: 12px;
		background: rgba(255, 255, 255, 0.78);
		box-shadow: 0 1px 8px rgba(15, 23, 42, 0.08);
		backdrop-filter: blur(8px);
	}

	.movesLabel {
		font-size: 0.9rem;
		color: #4b5563;
		font-weight: 600;
	}

	.movesValue {
		font-size: 1.15rem;
		font-weight: 800;
		color: #2563eb;
	}

	.solved {
		padding: 10px 14px;
		border-radius: 12px;
		background: #dcfce7;
		color: #166534;
		box-shadow: 0 1px 8px rgba(15, 23, 42, 0.08);
		font-weight: 800;
	}

	.boardWrap {
		display: flex;
		justify-content: center;
	}

	.card {
		width: 100%;
		max-width: 420px;
		background: rgba(255, 255, 255, 0.9);
		border-radius: 18px;
		box-shadow: 0 10px 30px rgba(15, 23, 42, 0.12);
		backdrop-filter: blur(10px);
		padding: 18px;
	}

	.board {
		display: grid;
		grid-template-columns: repeat(4, 1fr);
		gap: 10px;
	}

	.tile {
		aspect-ratio: 1 / 1;
		border-radius: 14px;
		border: 2px solid rgba(147, 197, 253, 0.9);
		background: linear-gradient(135deg, #3b82f6 0%, #2563eb 100%);
		color: #ffffff;
		font-size: 1.35rem;
		font-weight: 900;
		cursor: pointer;
		box-shadow: 0 6px 14px rgba(37, 99, 235, 0.25);
		transition:
			transform 150ms ease,
			box-shadow 150ms ease,
			filter 150ms ease;
		user-select: none;
	}

	.tile:hover {
		transform: scale(1.05);
		box-shadow: 0 10px 18px rgba(37, 99, 235, 0.35);
		filter: brightness(1.03);
	}

	.tile:active {
		transform: scale(0.96);
	}

	.tile:disabled {
		cursor: default;
	}

	.tile.empty {
		background: #f3f4f6;
		border-color: rgba(229, 231, 235, 1);
		box-shadow: none;
	}

	.controls {
		display: flex;
		justify-content: center;
		gap: 12px;
		margin: 18px 0 0;
		flex-wrap: wrap;
	}

	.btn {
		border-radius: 12px;
		padding: 12px 18px;
		font-size: 1rem;
		font-weight: 800;
		cursor: pointer;
		border: 0;
		transition:
			transform 120ms ease,
			box-shadow 120ms ease,
			filter 120ms ease;
	}

	.btn:active {
		transform: scale(0.98);
	}

	.btnPrimary {
		background: #2563eb;
		color: #fff;
		box-shadow: 0 10px 18px rgba(37, 99, 235, 0.25);
	}

	.btnPrimary:hover {
		filter: brightness(1.05);
	}

	.btnOutline {
		background: #ffffff;
		color: #2563eb;
		border: 2px solid rgba(37, 99, 235, 0.4);
		box-shadow: 0 10px 18px rgba(15, 23, 42, 0.08);
	}

	.btnOutline:hover {
		filter: brightness(1.02);
	}

	.instructions {
		margin-top: 22px;
		display: flex;
		justify-content: center;
	}

	.cardSoft {
		background: rgba(255, 255, 255, 0.8);
		box-shadow: 0 10px 30px rgba(15, 23, 42, 0.08);
		max-width: 520px;
	}

	.instructions h2 {
		margin: 0 0 10px;
		font-size: 1.15rem;
	}

	.instructions ul {
		margin: 0;
		padding: 0;
		list-style: none;
		display: grid;
		gap: 10px;
		color: #4b5563;
		font-size: 0.95rem;
		line-height: 1.35;
	}

	.instructions li {
		display: flex;
		align-items: flex-start;
	}

	.dot {
		margin-right: 10px;
		color: #2563eb;
		font-weight: 900;
		line-height: 1.2;
	}

	@media (max-width: 420px) {
		.card {
			padding: 14px;
		}
		.board {
			gap: 8px;
		}
		.tile {
			font-size: 1.2rem;
			border-radius: 12px;
		}
	}
</style>
