<script>
	import { onMount } from 'svelte';
	import Cell from './Cell.svelte'
	import Timer from './Timer.svelte'
	import { STATE } from './constants.js'


	let board = [];
	let rows = 9;
	let cols = 9;
	let totalMines = 10;
	let mines = 10;
	// let isGameOver = false;
	// let hasGameStarted = false;
	let minesPositionList = [];
	let isMouseDown = false;
	let gameState = STATE.WAITING

	let timer = 0
	let gameResult = ''


	onMount(() => {
		initBoard()
	})

	const addToTimer = () => {timer += 100}


	let timerInterval

	$: {
		if (gameState === STATE.PLAYING) {
			timerInterval = setInterval(addToTimer, 100)
		} else {
			clearInterval(timerInterval)
		}
	}


	const rng = (min, max) => {
		return Math.floor(Math.random() * (max - min)) + min;
	}

	const resetGame = () => {
		timer = 0
		board = []
		// isGameOver = true
		// hasGameStarted = false
		gameState = STATE.WAITING
		minesPositionList = []
		gameResult = ''
		initBoard()
		console.log('resetting')
	}

	const endGame = () => {
		// isGameOver = true
		// hasGameStarted = false
		gameState = STATE.FINISHED
	}

	const initBoard = () => {
		let _board = []
		for(let col = 0; col < cols; col++) {
			let currentRow = []
			for(let row = 0; row < rows; row++) {
				currentRow.push({
					hasBomb: false,
					isRevealed: false,
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
			addMineCounterToBorderingCells(position.row, position.col)
		})
		
		// hasGameStarted = true
		// isGameOver = false
		gameState = STATE.PLAYING
		revealCell(row, col)
		
	}

	const triggerCellReveal = (row, col) => {
		revealCell(row, col)
		let gameState = checkForVictory()
		if (gameState){
			gameResult = 'YOU WIN'
			endGame()
		}
	}

	const revealCell = (row, col) => {
		if (gameState !== STATE.FINISHED && !board[row][col].isRevealed) {
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

	const flagCell = (row, col) => {
		if(!board[row][col].isRevealed) {
			board[row][col].isFlagged = !board[row][col].isFlagged
		}
	}

	const	checkForVictory = () => {
		for(let col = 0; col < cols; col++) {
			for(let row = 0; row < rows; row++) {
				let currentCell = board[row][col]
				if(!currentCell.hasBomb && !currentCell.isRevealed && !currentCell.isFlagged) {
					return false
				}
			}
		}
		return true
	}

	const loseGame = () => {
		endGame()
		minesPositionList.forEach(mine => {
			board[mine.row][mine.col].isRevealed = true
		})
		gameResult = 'YOU LOSE'
	}

	

	
</script>

<svelte:window on:mousedown={() => isMouseDown = true} on:mouseup={() => isMouseDown = false} />

<div class="board-wrapper">
	<div class="controls">
		<h2>{gameResult}</h2>
		<button on:click={resetGame}>Reset</button>
		<Timer ms={timer} />
	</div>
	<div class="board">
		{#each board as row, rowIndex}
			{#each row as cell, cellIndex}
				<Cell
					row={rowIndex}
					col={cellIndex}
					width={`${100 / rows}%`}
					height={`${100 / cols}%`}
					gameState={gameState}
					isFlagged={cell.isFlagged}
					isMouseDown={isMouseDown}
					hasBomb={cell.hasBomb}
					isRevealed={cell.isRevealed}
					number={cell.number}
					on:revealInitialCell={(e) => revealInitialCell(e.detail.rowIndex, e.detail.colIndex)}
					on:revealCell={(e) => triggerCellReveal(e.detail.rowIndex, e.detail.colIndex)}
					on:flagCell={(e) => flagCell(e.detail.rowIndex, e.detail.colIndex)}
				/>
			{/each}
		{/each}
	</div>

</div>

<style>
	:global(*), :global(*::before), :global(*::after) {
	box-sizing: border-box;
	}

	:global(html), :global(body) {
		width: 100%;
		height: 100%;
	}

	:global(body) {
		display: flex;
		justify-content: center;
		align-items: center;
		background: #E8E8E8;
	}

		.board-wrapper {
			display: flex;
			flex-direction: column;
		}

		.controls {
			width: 100%;
			height: 60px;
			margin-bottom: 20px;
			background: #FAFAFA;
			border-radius: 5px;
			box-shadow: 0 3px 6px rgba(0,0,0,0.16), 0 3px 6px rgba(0,0,0,0.23);
			display: flex;
			justify-content: space-between;
			align-items: center;
			padding: 5px 25px;
		}

		.board {
		margin: auto;
		display: flex;
		flex-wrap: wrap;
		border: 1px solid #808080;
		box-shadow: 0 14px 28px rgba(0,0,0,0.25), 0 10px 10px rgba(0,0,0,0.22);
		width: 500px;
		height: 500px;
		padding: 1px;
	}
	

</style>
