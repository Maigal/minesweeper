<script>
  import { createEventDispatcher } from 'svelte';
  import { STATE } from './constants.js'
  const dispatch = createEventDispatcher();

  export let row;
  export let col;
  export let width;
  export let height;
  export let isMouseDown;
  export let hasBomb;
  export let isRevealed;
  export let number;
  export let gameState;
  export let isFlagged;

  let hovering = false;

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

  function enter() {
		hovering = true;
	}

	function leave() {
		hovering = false;
	}

  function handleCellClick(e) {
    if(e.which === 1) {
      if (!isFlagged && gameState !== STATE.FINISHED) {
        if (gameState === STATE.PLAYING) {
          revealCell()
        } else {
          revealInitialCell()
        }
      }
    } else if (e.which === 3) {
      flagCell()
    }
    
  }

  function revealCell() {
    dispatch('revealCell', {
      rowIndex: row,
      colIndex: col
    })
  }

  function revealInitialCell() {
    dispatch('revealInitialCell', {
      rowIndex: row,
      colIndex: col
    })
  }

  function flagCell() {
    dispatch('flagCell', {
      rowIndex: row,
      colIndex: col
    })
  }
</script>



<div 
    class="cell" 
    style="width: {width}; height: {height}"
    class:revealed="{isRevealed}"
    class:hovering={!isRevealed && hovering && isMouseDown}
    on:contextmenu|preventDefault
    on:mouseup={handleCellClick}
    on:mouseenter={enter} on:mouseleave={leave}
  >
  {#if !hasBomb && gameState === STATE.FINISHED && isFlagged}
    <div class="cell-bomb">✔</div>
  {:else if isFlagged}
    <div class="cell-flag">✔</div>
  {:else if hasBomb && isRevealed}
    <div class="cell-bomb">X</div>
  {:else if isRevealed && number > 0}
    <p class="cell-amount" style="color: {numberColors[number]}">{number}</p>
  {/if}

  </div>


<style>
  .cell {
		border: solid #808080;
		border-width: 0 1px 1px 0;
		cursor: pointer;
		display: flex;
		justify-content: center;
		align-items: center;
		background: #c0c0c0;
		box-shadow: inset 2px 2px 2px 0px #F7F7F7, inset -2px -2px 2px 0px #7D7D7D;
    margin: 0;
    padding: 0;
    user-select: none;
	}

	.cell:hover {
		background: #dddcdc;
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

  .cell-flag {
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
		background: #42aa42
  }

  .hovering {
    box-shadow: inset 5px 5px 2px 0px rgba(0,0,0,0.2) !important;
  }
</style>