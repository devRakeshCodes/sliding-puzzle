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
	<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover" />
</svelte:head>

<div class="app">
	<div class="topFade" aria-hidden="true"></div>

	<header class="hud" in:fade={{ duration: 450 }}>
		<div class="brand">
			<div class="logo" aria-hidden="true">15</div>
			<div class="brandText">
				<h1>Sliding Puzzle</h1>
				<p>Arrange 1–15</p>
			</div>
		</div>

		<div class="hudRight">
			<div class="pill" aria-label="Move counter">
				<span class="pillLabel">Moves</span>
				<span class="pillValue">{moveCount}</span>
			</div>

			<div class="hudButtons">
				<button class="iconBtn" onclick={shufflePuzzle} aria-label="New game">↻</button>
				<button class="iconBtn ghost" onclick={resetPuzzle} aria-label="Show solution">✓</button>
			</div>
		</div>
	</header>

	{#if isSolved}
		<div class="toast" in:fade>
			<div class="toastEmoji">🎉</div>
			<div class="toastText">
				<div class="toastTitle">You solved it!</div>
				<div class="toastSub">Nice moves. Tap ↻ to play again.</div>
			</div>
		</div>
	{/if}

	<main class="stage">
		<section class="boardShell" in:scale={{ duration: 520, delay: 120 }}>
			<div class="boardHeader">
				<div class="hint">Swipe-style taps • No zoom</div>
				<div class="difficulty">4×4</div>
			</div>

			<div class="board" aria-label="15 puzzle board">
				{#each puzzle as tile, index (index)}
					<button
						class="tile {tile === 0 ? 'empty' : ''}"
						onclick={() => handleTileClick(index)}
						disabled={isSolved}
						aria-label={tile === 0 ? 'Empty space' : `Tile ${tile}`}
						in:fly={{
							x: Math.random() * 36 - 18,
							y: Math.random() * 36 - 18,
							duration: 360,
							delay: index * 40,
							easing: quintOut
						}}
					>
						{#if tile !== 0}
							<span class="tileInner">
								<span class="tileNumber">{tile}</span>
							</span>
						{/if}
					</button>
				{/each}
			</div>

			<div class="bottomBar" in:fly={{ y: 16, duration: 420, delay: 250 }}>
				<button class="btn primary" onclick={shufflePuzzle}>New Game</button>
				<button class="btn secondary" onclick={resetPuzzle}>Solve</button>
			</div>
		</section>

		<section class="miniHelp" in:fade={{ duration: 450, delay: 300 }}>
			<div class="miniCard">
				<div class="miniTitle">How to play</div>
				<div class="miniRow">
					<span class="dot">•</span>
					<span>Tap a tile next to the empty space</span>
				</div>
				<div class="miniRow">
					<span class="dot">•</span>
					<span>Order tiles 1–15, empty in bottom-right</span>
				</div>
			</div>
		</section>
	</main>
</div>

<style>
	/* --- Mobile-game baseline --- */
	/* Prevent annoying tap highlight + double-tap zoom-ish behavior */
	button {
		-webkit-tap-highlight-color: transparent;
		touch-action: manipulation;
	}

	.app {
		min-height: 100svh;
		padding: calc(14px + env(safe-area-inset-top)) calc(14px + env(safe-area-inset-right))
			calc(18px + env(safe-area-inset-bottom)) calc(14px + env(safe-area-inset-left));
		background:
			radial-gradient(1200px 700px at 20% -10%, rgba(241, 191, 99, 0.45), transparent 60%),
			radial-gradient(900px 600px at 90% 10%, rgba(236, 219, 72, 0.35), transparent 55%),
			radial-gradient(800px 600px at 55% 120%, rgba(34, 197, 94, 0.22), transparent 55%),
			linear-gradient(180deg, #737dad 0%, #404968 60%, #010b3b 100%);
		display: flex;
		flex-direction: column;
		gap: 14px;
		position: relative;
	}

	.topFade {
		position: absolute;
		inset: 0 0 auto 0;
		height: 140px;
		background: linear-gradient(180deg, rgba(0, 0, 0, 0.45), transparent);
		pointer-events: none;
	}

	/* --- HUD --- */
	.hud {
		display: flex;
		align-items: center;
		justify-content: space-between;
		gap: 12px;
	}

	.brand {
		display: flex;
		align-items: center;
		gap: 10px;
		min-width: 0;
	}

	.logo {
		width: 44px;
		height: 44px;
		border-radius: 14px;
		display: grid;
		place-items: center;
		font-weight: 900;
		letter-spacing: -0.04em;
		color: rgba(255, 255, 255, 0.95);
		background:
			linear-gradient(135deg, rgba(59, 130, 246, 0.95), rgba(99, 102, 241, 0.85)),
			radial-gradient(120px 80px at 30% 20%, rgba(255, 255, 255, 0.25), transparent 55%);
		box-shadow:
			0 10px 24px rgba(0, 0, 0, 0.35),
			inset 0 1px 0 rgba(255, 255, 255, 0.18);
	}

	.brandText {
		min-width: 0;
	}

	.brandText h1 {
		margin: 0;
		font-size: 1.05rem;
		font-weight: 900;
		letter-spacing: -0.02em;
		line-height: 1.1;
		white-space: nowrap;
		overflow: hidden;
		text-overflow: ellipsis;
	}

	.brandText p {
		margin: 2px 0 0;
		font-size: 0.82rem;
		color: rgba(255, 255, 255, 0.7);
	}

	.hudRight {
		display: flex;
		align-items: center;
		gap: 10px;
	}

	.pill {
		display: inline-flex;
		align-items: baseline;
		gap: 8px;
		padding: 10px 12px;
		border-radius: 999px;
		background: rgba(255, 255, 255, 0.08);
		border: 1px solid rgba(255, 255, 255, 0.12);
		backdrop-filter: blur(10px);
		box-shadow: 0 12px 28px rgba(0, 0, 0, 0.25);
	}

	.pillLabel {
		font-size: 0.78rem;
		color: rgba(255, 255, 255, 0.7);
		font-weight: 700;
	}

	.pillValue {
		font-size: 1rem;
		font-weight: 900;
	}

	.hudButtons {
		display: flex;
		gap: 8px;
	}

	.iconBtn {
		width: 44px;
		height: 44px;
		border-radius: 14px;
		border: 1px solid rgba(255, 255, 255, 0.14);
		background: rgba(255, 255, 255, 0.08);
		color: rgba(255, 255, 255, 0.92);
		font-weight: 900;
		font-size: 1.05rem;
		box-shadow:
			0 12px 28px rgba(0, 0, 0, 0.25),
			inset 0 1px 0 rgba(255, 255, 255, 0.14);
		transition:
			transform 120ms ease,
			filter 120ms ease;
	}

	.iconBtn:active {
		transform: translateY(1px) scale(0.98);
	}

	.iconBtn:hover {
		filter: brightness(1.06);
	}

	.iconBtn.ghost {
		background: rgba(255, 255, 255, 0.05);
	}

	/* --- Toast win banner --- */
	.toast {
		display: flex;
		gap: 10px;
		align-items: center;
		padding: 12px 14px;
		border-radius: 18px;
		background:
			linear-gradient(135deg, rgba(34, 197, 94, 0.22), rgba(59, 130, 246, 0.16)),
			rgba(255, 255, 255, 0.06);
		border: 1px solid rgba(255, 255, 255, 0.14);
		backdrop-filter: blur(12px);
		box-shadow: 0 18px 40px rgba(0, 0, 0, 0.35);
	}

	.toastEmoji {
		width: 40px;
		height: 40px;
		border-radius: 14px;
		display: grid;
		place-items: center;
		background: rgba(255, 255, 255, 0.08);
		border: 1px solid rgba(255, 255, 255, 0.14);
	}

	.toastTitle {
		font-weight: 900;
		letter-spacing: -0.01em;
	}

	.toastSub {
		margin-top: 2px;
		font-size: 0.86rem;
		color: rgba(255, 255, 255, 0.72);
	}

	/* --- Stage --- */
	.stage {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 14px;
	}

	.boardShell {
		width: min(94vw, 520px);
		border-radius: 26px;
		padding: 14px;
		background: rgba(255, 255, 255, 0.06);
		border: 1px solid rgba(255, 255, 255, 0.12);
		backdrop-filter: blur(14px);
		box-shadow:
			0 26px 70px rgba(0, 0, 0, 0.45),
			inset 0 1px 0 rgba(255, 255, 255, 0.12);
	}

	.boardHeader {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 4px 6px 10px;
	}

	.hint {
		font-size: 0.8rem;
		color: rgba(255, 255, 255, 0.65);
	}

	.difficulty {
		font-size: 0.8rem;
		font-weight: 800;
		padding: 6px 10px;
		border-radius: 999px;
		background: rgba(255, 255, 255, 0.08);
		border: 1px solid rgba(255, 255, 255, 0.12);
	}

	.board {
		display: grid;
		grid-template-columns: repeat(4, 1fr);
		gap: 10px;
		padding: 10px;
		border-radius: 22px;
		background:
			radial-gradient(220px 160px at 25% 20%, rgba(255, 255, 255, 0.18), transparent 60%),
			rgba(255, 255, 255, 0.08);
		border: 1px solid rgba(255, 255, 255, 0.1);
	}

	/* Big, tappable tiles (mobile game feel) */
	.tile {
		aspect-ratio: 1 / 1;
		border-radius: 18px;
		border: 1px solid rgba(255, 255, 255, 0.14);
		background:
			linear-gradient(135deg, rgba(1, 59, 153, 0.95), rgba(9, 11, 133, 0.82)),
			radial-gradient(120px 90px at 30% 20%, rgba(255, 255, 255, 0.25), transparent 55%);
		box-shadow:
			0 14px 26px rgba(0, 0, 0, 0.35),
			inset 0 1px 0 rgba(255, 255, 255, 0.16);
		color: rgba(255, 255, 255, 0.96);
		cursor: pointer;
		user-select: none;
		transition:
			transform 120ms ease,
			filter 120ms ease,
			box-shadow 120ms ease;
		position: relative;
		overflow: hidden;
	}

	.tile::after {
		content: '';
		position: absolute;
		inset: -40% -20% auto -30%;
		height: 80%;
		transform: rotate(-12deg);
		background: linear-gradient(180deg, rgba(255, 255, 255, 0.22), transparent);
		opacity: 0.65;
		pointer-events: none;
	}

	.tile:hover {
		filter: brightness(1.05);
	}

	.tile:active {
		transform: translateY(1px) scale(0.985);
		box-shadow:
			0 10px 18px rgba(0, 0, 0, 0.32),
			inset 0 1px 0 rgba(255, 255, 255, 0.14);
	}

	.tile:focus-visible {
		outline: 3px solid rgba(255, 255, 255, 0.55);
		outline-offset: 3px;
	}

	.tile.empty {
		background: rgba(255, 255, 255, 0.06);
		border: 1px dashed rgba(255, 255, 255, 0.18);
		box-shadow: none;
		cursor: default;
	}

	.tile.empty::after {
		display: none;
	}

	.tileInner {
		position: relative;
		z-index: 1;
		width: 100%;
		height: 100%;
		display: grid;
		place-items: center;
	}

	.tileNumber {
		font-size: clamp(1.25rem, 3.8vw, 1.8rem);
		font-weight: 950;
		letter-spacing: -0.03em;
		text-shadow: 0 10px 20px rgba(0, 0, 0, 0.25);
	}

	/* Bottom bar controls */
	.bottomBar {
		display: flex;
		gap: 10px;
		padding: 12px 6px 4px;
	}

	.btn {
		flex: 1;
		height: 50px;
		border-radius: 16px;
		border: 1px solid rgba(255, 255, 255, 0.14);
		font-weight: 900;
		letter-spacing: -0.01em;
		color: rgba(255, 255, 255, 0.92);
		background: rgba(255, 255, 255, 0.08);
		box-shadow:
			0 16px 34px rgba(0, 0, 0, 0.28),
			inset 0 1px 0 rgba(255, 255, 255, 0.14);
		transition:
			transform 120ms ease,
			filter 120ms ease;
	}

	.btn:active {
		transform: translateY(1px) scale(0.99);
	}

	.btn:hover {
		filter: brightness(1.06);
	}

	.btn.primary {
		background: linear-gradient(135deg, rgba(34, 197, 94, 0.55), rgba(59, 130, 246, 0.45));
	}

	.btn.secondary {
		background: rgba(212, 149, 11, 0.82);
	}

	/* Help card */
	.miniHelp {
		width: min(94vw, 520px);
	}

	.miniCard {
		border-radius: 20px;
		padding: 14px 14px;
		background: rgba(255, 255, 255, 0.06);
		border: 1px solid rgba(255, 255, 255, 0.12);
		backdrop-filter: blur(14px);
		box-shadow: 0 18px 40px rgba(0, 0, 0, 0.3);
	}

	.miniTitle {
		font-weight: 900;
		margin-bottom: 8px;
		letter-spacing: -0.01em;
	}

	.miniRow {
		display: flex;
		gap: 10px;
		align-items: flex-start;
		color: rgba(255, 255, 255, 0.74);
		font-size: 0.92rem;
		line-height: 1.3;
	}

	.dot {
		color: rgba(255, 255, 255, 0.85);
		font-weight: 900;
	}

	/* Slightly larger spacing on bigger screens */
	@media (min-width: 480px) {
		.board {
			gap: 12px;
			padding: 12px;
		}
		.boardShell {
			padding: 16px;
		}
	}

	/* --- Desktop polish: center the "mobile game" and soften the background --- */
	@media (min-width: 900px) {
		:global(body) {
			/* neutral dark desktop backdrop */
			background:
				radial-gradient(900px 600px at 50% 20%, rgba(255, 255, 255, 0.06), transparent 60%),
				linear-gradient(180deg, #070a12 0%, #050714 100%);
		}

		.app {
			/* remove the loud mobile wallpaper on desktop */
			background: transparent;
			min-height: 100vh;
			padding: 32px;
			align-items: center;
			justify-content: center;
		}

		.stage {
			width: 100%;
			max-width: 520px;
		}

		.hud,
		.toast,
		.miniHelp {
			width: 520px;
		}

		.boardShell {
			width: 520px;
			border-radius: 32px;
			padding: 18px;
			background: rgba(255, 255, 255, 0.07);
			border: 1px solid rgba(255, 255, 255, 0.14);
			box-shadow:
				0 40px 110px rgba(0, 0, 0, 0.55),
				inset 0 1px 0 rgba(255, 255, 255, 0.12);
			position: relative;
		}

		.boardShell::before {
			content: '';
			display: block;
			height: 10px;
			width: 120px;
			margin: 2px auto 12px;
			border-radius: 999px;
			background: rgba(255, 255, 255, 0.12);
		}
	}
</style>
