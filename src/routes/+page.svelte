<script lang="ts">
    const LINES = 11;

    let board = Array(LINES).fill(null).map(() => Array(LINES).fill(null));
    let isBlackTurn = true;
    let isBlindMode = false;
    let isGameFinished = false;
    let moves = [];

    function handleClick(row: number, col: number) {
        if (board[row][col] !== null || isGameFinished) return;
        if (isBlackTurn) {
            board[row][col] = 'B';
        } else {
            board[row][col] = 'W';
        }
        if (checkConsecutiveStones()) {
            alert(`${isBlackTurn ? 'Black' : 'White'} wins!`);
            isGameFinished = true;
            isBlindMode = false;
            return;
        }
        if (moves.length >= LINES * LINES - 1) {
            alert(`It's a draw!`);
            isGameFinished = true;
            isBlindMode = false;
            return;
        }
        moves.push([row, col]);
        isBlackTurn = !isBlackTurn;
    }

    function handleKeyDown(event: KeyboardEvent, row: number, col: number) {
        if (event.key === 'Enter') {
            handleClick(row, col);
        }
    }

    function restartGame() {
        // confirm
        if (!isGameFinished) {
            if (!confirm('Are you sure you want to restart the game?')) {
                return;
            }
        }
        board = Array(LINES).fill(null).map(() => Array(LINES).fill(null));
        moves = [];
        isBlackTurn = true;
        isGameFinished = false;
    }

    function undoMove() {
        if (moves.length === 0) return;
        const [row, col] = moves.pop();
        board[row][col] = null;
        isBlackTurn = !isBlackTurn;
    }

    function checkConsecutiveStones() {
        const size = LINES;

        // Check horizontally
        for (let row = 0; row < size; row++) {
            for (let col = 0; col <= size - 5; col++) {
                if (
                    board[row][col] !== null &&
                    board[row][col] === board[row][col + 1] &&
                    board[row][col] === board[row][col + 2] &&
                    board[row][col] === board[row][col + 3] &&
                    board[row][col] === board[row][col + 4]
                ) {
                    return true;
                }
            }
        }

        // Check vertically
        for (let col = 0; col < size; col++) {
            for (let row = 0; row <= size - 5; row++) {
                if (
                    board[row][col] !== null &&
                    board[row][col] === board[row + 1][col] &&
                    board[row][col] === board[row + 2][col] &&
                    board[row][col] === board[row + 3][col] &&
                    board[row][col] === board[row + 4][col]
                ) {
                    return true;
                }
            }
        }

        // Check diagonally (top-left to bottom-right)
        for (let row = 0; row <= size - 5; row++) {
            for (let col = 0; col <= size - 5; col++) {
                if (
                    board[row][col] !== null &&
                    board[row][col] === board[row + 1][col + 1] &&
                    board[row][col] === board[row + 2][col + 2] &&
                    board[row][col] === board[row + 3][col + 3] &&
                    board[row][col] === board[row + 4][col + 4]
                ) {
                    return true;
                }
            }
        }

        // Check diagonally (top-right to bottom-left)
        for (let row = 0; row <= size - 5; row++) {
            for (let col = size - 1; col >= 4; col--) {
                if (
                    board[row][col] !== null &&
                    board[row][col] === board[row + 1][col - 1] &&
                    board[row][col] === board[row + 2][col - 2] &&
                    board[row][col] === board[row + 3][col - 3] &&
                    board[row][col] === board[row + 4][col - 4]
                ) {
                    return true;
                }
            }
        }

        return false;
    }
</script>

<style>
    .container {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        height: 100vh; /* Make the container take up the full viewport height */
    }

    .title-container {
        text-align: center;
    }

    .board-container {
        margin-bottom: 20px; /* Adjust the margin as needed */
    }

    .board {
        display: grid;
        grid-template-columns: repeat(var(--lines), calc(100% / var(--lines)));
        grid-template-rows: repeat(var(--lines), calc(100% / var(--lines)));
        grid-gap: 0;
        background-color: #DCB35C; /* Brown background color */
        padding: 5px; /* Add padding for the grid lines */
        position: relative; /* Add position relative to the board */
        width: 90vw; /* Set max width for the board */
        max-width: 500px;
        height: 90vw; /* Set max height for the board */
        max-height: 500px;
    }

    .line {
        position: absolute;
        background-color: #000;
    }

    .horizontal-line {
        height: 2px;
        width: 100%;
        top: 50%;
        transform: translateY(-50%);
    }

    .vertical-line {
        height: 100%;
        width: 2px;
        left: 50%;
        transform: translateX(-50%);
    }

    .cell {
        width: 100%;
        height: 100%;
        display: flex;
        align-items: center;
        justify-content: center;
        cursor: pointer;
        font-size: 1.2em;
        background-color: rgba(0, 0, 0, 0);
        position: relative; /* Add position relative to the cell */
    }

    .stone {
        border-radius: 50%;
        width: 80%;
        height: 80%;
        position: absolute;
        /* Add gradient for shiny effect */
        background: radial-gradient(circle at 20% 20%, rgba(255,255,255,0.8), rgba(255,255,255,0) 70%);
        /* Add box-shadow for depth */
        box-shadow: 0 0 5px rgba(0,0,0,0.5);
        /* Add transition for smooth hover effect */
        transition: transform 0.2s ease;
    }

    .black-stone {
        background-color: #000;
    }

    .white-stone {
        background-color: #fbfbfb;
    }

    .purple-stone {
        background-color: #800080;
    }

    .button-container {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        gap: 10px;
        margin-bottom: 7em; /* Adjust the margin as needed */
    }
</style>

<div style='--lines:{LINES};'>
    <div class="container">
        <div>
            <label>
                Blind Mode
                <input type="checkbox" bind:checked={isBlindMode} />
            </label>
        </div>

        <div class="title-container">
            <h1>Connect Five</h1>
            <p>{isBlackTurn ? '⚫ Black' : '⚪ White'}'s turn {isBlackTurn ? '⚫' : '⚪'}</p>
        </div>
        <div class="board-container">
            <div class="board">
                {#each board as row, rowIndex}
                    {#each row as cell, colIndex}
                        <div class="cell" role="button" tabindex="0" on:click={() => handleClick(rowIndex, colIndex)} on:keydown={(event) => handleKeyDown(event, rowIndex, colIndex)}>
                            {#if colIndex < LINES}
                                <div class="line vertical-line"></div>
                            {/if}
                            {#if rowIndex < LINES}
                                <div class="line horizontal-line"></div>
                            {/if}
                            {#if isBlindMode && cell !== null}
                                <div class="stone purple-stone"></div>
                            {:else if cell === 'B'}
                                <div class="stone black-stone"></div>
                            {:else if cell === 'W'}
                                <div class="stone white-stone"></div>
                            {/if}
                        </div>
                    {/each}
                {/each}
            </div>
        </div>
        <div class="button-container">
            <div>
                <button on:click={restartGame}>Restart Game</button>
                <button on:click={undoMove} disabled={isGameFinished}>Undo Move</button>
            </div>
        </div>
    </div>
</div>
