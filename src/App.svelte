<script>
	import { onMount } from 'svelte';
	let board = [];
	let rows = 9;
	let cols = 9;
	let totalMines = 10;
	let mines = 10;
	let isGameOver = false;
	let hasGameStarted = false;
	let minesPositionList = [];
	let isMouseDown = false;

	let numberColors = {
		0: "black",
		1: "blue",
		2: "green",
		3: "red",
		4: "purple",
		5: "orange",
		6: "teal",
		7: "brown",
		8: "black"
	}


	onMount(() => {
		initBoard()
	})

	const rng = (min, max) => {
		return Math.floor(Math.random() * (max - min)) + min;
	}

	const initBoard = () => {
		let _board = []
		for(let col = 0; col < cols; col++) {
			let currentRow = []
			for(let row = 0; row < rows; row++) {
				currentRow.push({
					hasBomb: false,
					isRevelead: false,
					isFlagged: false,
					number: 0
				})
			}
			_board.push(currentRow)
		}
		console.table(board)
		board = _board
	}

	const addMineCounterToBorderingCells = (row, col) => {
		for(let xOffset = -1; xOffset <= 1; xOffset++) {
			for(let yOffset = -1; yOffset <= 1; yOffset++) {
				if (board[row + xOffset] && board[row + xOffset][col + yOffset] && !board[row + xOffset][col + yOffset].hasBomb) {
					board[row + xOffset][col + yOffset].number ++
				}
			}
		}
	}

	const revealInitialCell = (row, col) => {
		let minesToPlace = totalMines;
		
		while(minesToPlace > 0) {
			let _row = rng(0, rows)
			let _col = rng(0, cols)
			if (!(_row === row && _col === col) && !board[_row][_col].hasBomb) {
				board[_row][_col].hasBomb = true
				minesPositionList.push({row: _row, col: _col})
				
				minesToPlace--
			}
		}

		minesPositionList.forEach(position => {
					console.log('pos ', position)
			addMineCounterToBorderingCells(position.row, position.col)
		})
		
		
		revealCell(row, col)
		hasGameStarted = true
	}

	const revealCell = (row, col) => {
		if (!isGameOver && !board[row][col].isRevealed) {
			board[row][col].isRevealed = true
			if (board[row][col].hasBomb) {
				loseGame()
			} 
			else if (board[row][col].number === 0) {
				for(let xOffset = -1; xOffset <= 1; xOffset++) {
					for(let yOffset = -1; yOffset <= 1; yOffset++) {
						if (board[row + xOffset] && board[row + xOffset][col + yOffset] && !(xOffset === 0 && yOffset === 0)) {	
							revealCell(row + xOffset,col + yOffset)
						}
					}
				}
			}
			
		}
	}

	const loseGame = () => {
		isGameOver = true
		minesPositionList.forEach(mine => {
			board[mine.row][mine.col].isRevealed = true
		})
	}

	
</script>

<svelte:window on:mousedown={() => isMouseDown = true} on:mouseup={() => isMouseDown = false} />

<div class="board">
	{#each board as row, rowIndex}
		{#each row as cell, cellIndex}
			<div 
				class="cell" 
				style="width: {`${100 / rows}%`}; height: {`${100 / cols}%`};"
				class:revealed="{cell.isRevealed}"
				on:click={() => hasGameStarted ? revealCell(rowIndex, cellIndex) : revealInitialCell(rowIndex, cellIndex)}
				on:mouseover={(e) => console.log(e, this)}
			>
			{#if cell.hasBomb && cell.isRevealed}
				<div class="cell-bomb"> X</div>
			{:else if cell.isRevealed && cell.number > 0}
				<p class="cell-amount" style="color: {numberColors[cell.number]}">{cell.number}</p>
			{/if}

			</div>
		{/each}
	{/each}
</div>

<style>
	*, *::before, *::after {
	box-sizing: border-box;
	}

	:global(html), :global(body) {
		width: 100%;
		height: 100%;
	}
	.board {
		margin: auto;
		display: flex;
		flex-wrap: wrap;
		border: solid #808080;
		border-width: 1px 0 0 1px;
		width: 500px;
		height: 500px;
	}

	.cell {
		border: solid #808080;
		border-width: 0 1px 1px 0;
		cursor: pointer;
		display: flex;
		justify-content: center;
		align-items: center;
		background: #c0c0c0;
		box-shadow: inset 2px 2px 2px 0px #F7F7F7, inset -2px -2px 2px 0px #7D7D7D;
	}

	.cell:hover {
		background: lightblue;
	}


	.cell.revealed {
		background: #dfdede;
		box-shadow: none;
	}

	.cell-amount {
		font-weight: 600;
		font-size: 24px;
		margin: 0;
		padding: 0;
	}

	.cell-bomb {
		font-weight: 600;
		font-size: 24px;
		margin: 0;
		padding: 0;
		color: #FAFAFA;
		width: 100%;
		height: 100%;
		display: flex;
		justify-content: center;
		align-items: center;
		background: rgb(124, 56, 56)
	}


	

</style>
