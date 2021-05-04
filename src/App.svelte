<script>
	const ROWS = 6;
	const COLUMNS = 7;
	let board = generateBlankBoard();

	let turn = 1;
	$: player = turn % 2 == 1 ? 1 : 2;

	let gameOver = false;
	let draw = false;

	let firstY;
	let currentCol;

	function generateBlankBoard() {
		return [...Array(ROWS)].map(() => [...Array(COLUMNS)].map(() => null));
	}
	function resetGame() {
		board = generateBlankBoard();
		gameOver = false;
		draw = false;
		turn = 1;
	}

	function getFirstAvailableCellY(col) {
		if (board[0][col]) return 0;

		let i = ROWS - 1;
		while (i >= 0) {
			if (!board[i][col]) {
				return i;
			}
			i--;
		}
	}
	function dropPiece(col) {
		if (board[0][col] || gameOver || draw) return;

		let curPlayer = 'p' + player;
		let y = getFirstAvailableCellY(col);
		board[y][col] = curPlayer;
	}

	function dropRandomPiece() {
		if (gameOver) return;
		let col = Math.floor(Math.random() * COLUMNS);
		if (board[0][col] && turn < 42) {
			return dropRandomPiece();
		} else {
			let randomPieceFirstY = getFirstAvailableCellY(col);
			dropPiece(col);
			if (isWinningMove(col, randomPieceFirstY)) {
				return (gameOver = true);
			}
			turn++;
		}
	}

	function isWinningMove(x, y) {
		// This checks if the move of the current player would win them the game based on the x and y position of the last move
		let p = 'p' + player;

		// HORIZONTAL
		let row = board[y]; // We know the row the move is in and only have to check if there are 4 pieces of the same color in a row in this row
		for (let i = 0; i < COLUMNS - 3; i++) {
			// This only goes to COLUMNS - 3 as a new row cannot start in the last 3 tiles of a row
			// because this checks every tile in the row and sees if there are 3 more pieces of the same player going out from that tile
			if (row[i] == p && row[i + 1] == p && row[i + 2] == p && row[i + 3] == p) return true;
		}
		// VERTICAL
		for (let i = 0; i < ROWS - 3; i++) {
			// This is pretty much the same as horizontal, the column the move is in is known
			// so this checks every tile in the column and sees if there are 4 pieces of the same color in a row
			if (board[i][x] == p && board[i + 1][x] == p && board[i + 2][x] == p && board[i + 3][x] == p) return true;
		}

		// DIAGONAL
		// This differs from the horizontal and vertical checking because it's checking every cell and is not just limited to the
		// row/column the move was places in, and so is probably really inefficient

		// This starts at the top (so again only to ROWS-3), looks at each cell and goes diagonally down (in both directions) from that cell
		// It again checks if there are 4 pieces of the same color in a diagonal row, going out from said cell
		for (let i = 0; i < ROWS - 3; i++) {
			for (let j = 0; j < COLUMNS; j++) {
				if (board[i][j] != p) continue; // a cell can be completely skipped if itself isn't the color of the current player.

				// LEFT -> RIGHT
				if (board[i + 1][j - 1] == p && board[i + 2][j - 2] == p && board[i + 3][j - 3] == p) {
					return true;
				}

				// RIGHT -> LEFT
				if (board[i + 1][j + 1] == p && board[i + 2][j + 2] == p && board[i + 3][j + 3] == p) {
					return true;
				}
			}
		}
		return false;
	}
</script>

<svelte:head><title>Svelte Connect 4</title></svelte:head>
<main>
	{#if gameOver}
		<div class="game-over-div">
			<h1>Game Over</h1>
			<p><span class={'player-info' + player}>Player {player}</span> has won in turn {turn}!</p>
			<button class="new-game-btn" on:click={resetGame}>New Game</button>
		</div>
	{:else if draw}
		<div class="game-over-div">
			<h1>Draw</h1>
			<p>Neither player could connect 4 in time.</p>
			<button class="new-game-btn" on:click={resetGame}>New Game</button>
		</div>
	{/if}

	<h1 class="heading">Svelte Connect 4</h1>
	<div class="center">
		<div id="board-div">
			{#each board as row, y}
				<div class="row">
					{#each row as cell, x}
						<div
							class="cell-background"
							on:click={e => {
								if (board[0][x] || gameOver || draw) return;
								firstY = getFirstAvailableCellY(x);
								dropPiece(x);

								if (isWinningMove(x, firstY)) {
									return (gameOver = true);
								}
								firstY = getFirstAvailableCellY(x);
								console.log(`Turn: ${turn} | Player: ${player}\nNew piece at x = ${x}, y = ${y}\n--`);
								turn++;
								if (turn == ROWS * COLUMNS + 1) {
									draw = true;
								}
							}}
							on:mouseover={e => {
								currentCol = x;
								firstY = getFirstAvailableCellY(currentCol);
							}}
							on:mouseleave={e => {
								firstY = undefined;
								currentCol = undefined;
							}}
						>
							<div
								class={`cell ${cell} ${
									firstY == y && currentCol == x ? 'valid' : null
								} p${player}-indicator`}
							/>
						</div>
					{/each}
				</div>
			{/each}
		</div>
		<div class="turn-info">
			<h1>Turn: {turn} (<span class={'player-info' + player}>Player {player}</span>)</h1>
		</div>
		<div class="controls">
			<button class="reset-btn" on:click={resetGame}>Reset Game</button>
			<button on:click={dropRandomPiece}>Random</button>
		</div>
	</div>
</main>

<style>
	@import url('https://fonts.googleapis.com/css2?family=Poppins&display=swap');
	:root {
		--player-1-color: #f44336;
		--player-1-indicator: #f443363d;
		--player-2-color: #ffeb3b;
		--player-2-indicator: #fff1763f;
	}
	:global(*) {
		font-family: 'Poppins', sans-serif;
		box-sizing: border-box;
		margin: 0;
		padding: 0;
	}
	h1,
	p {
		color: white;
	}
	main {
		height: 100vh;
		background-color: #181818;
	}
	button {
		padding: 1em;
		border: none;
		border-radius: 7px;
		color: white;
		background-color: rgb(223, 30, 30);
		font-size: 1.05rem;
	}
	button:hover {
		cursor: pointer;
	}
	.heading {
		height: 7vh;
		background-color: rgb(223, 30, 30);
		color: white;
		display: flex;
		justify-content: center;
		align-items: center;
	}

	.center {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}
	#board-div {
		background-color: #07090a;
		margin: 1.5em;
		border-radius: 15px;
		padding: 0.2em;
	}
	.row {
		display: flex;
	}
	.cell {
		background: #2b2a2a;
		color: white;
		width: 75px;
		height: 75px;
		border-radius: 50%;
		margin: 1px;
		transition: 0.2s ease;
	}
	.cell.valid.p1-indicator {
		background-color: var(--player-1-indicator);
	}
	.cell.valid.p2-indicator {
		background-color: var(--player-2-indicator);
	}
	.p1 {
		background-color: var(--player-1-color) !important;
	}
	.p2 {
		background-color: var(--player-2-color) !important;
	}

	.turn-info {
		text-align: center;
	}
	.player-info1 {
		color: var(--player-1-color);
	}
	.player-info2 {
		color: var(--player-2-color);
	}
	.controls {
		margin: 1em 0 1em 0;
	}

	.game-over-div {
		z-index: 2;
		position: absolute;
		padding: 0.3em;
		width: 50vh;
		border: 2px solid #eeeeee;
		border-radius: 20px;
		background-color: black;
		color: white;
		top: 25%;
		left: 82.5%;
		transform: translate(-50%, -50%);
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}
	.new-game-btn {
		margin-top: 7px;
		padding: 0.6em;
		font-size: 1.1rem;
	}
</style>
