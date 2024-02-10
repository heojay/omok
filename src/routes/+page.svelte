<script lang="ts">
    import { GomokuSolution } from '@algorithm.ts/gomoku';

    const LINES = 11;

    let board = Array(LINES)
        .fill(null)
        .map(() => Array(LINES).fill(null));
    let isBlackTurn = true;
    let isBlindMode = false;
    let isOnePlayerMode = null;
    let isGameFinished = true;
    let moves = [];

    const solution = new GomokuSolution({
        MAX_ROW: LINES,
        MAX_COL: LINES
    });
    solution.init([]);

    function handleClick(row: number, col: number) {
        if (board[row][col] !== null || isGameFinished) return;
        if (isBlackTurn) {
            board[row][col] = 0;
        } else {
            board[row][col] = 1;
        }
        if (checkConsecutiveStones(row, col)) {
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
        solution.forward(row, col, isBlackTurn ? 0 : 1);
        isBlackTurn = !isBlackTurn;

        if (isOnePlayerMode && !isBlackTurn) {
            const [row, col] = solution.minimaxSearch(isBlackTurn ? 1 : 0);
            handleClick(row, col);
        }
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
        board = Array(LINES)
            .fill(null)
            .map(() => Array(LINES).fill(null));
        moves = [];
        isBlackTurn = true;
        isGameFinished = false;
        solution.init([]);
    }

    function undoMove() {
        if (moves.length === 0) return;
        const [row, col] = moves.pop();
        solution.revert(row, col);
        board[row][col] = null;
        isBlackTurn = !isBlackTurn;

        if (isOnePlayerMode && !isBlackTurn) {
            undoMove();
        }
    }

    function checkConsecutiveStones(row, col) {
        const size = LINES;
        const stone = board[row][col];

        // Check horizontally
        let count = 0;
        for (let i = -4; i <= 4; i++) {
            const c = col + i;
            if (c < 0 || c >= size) continue;
            count = board[row][c] === stone ? count + 1 : 0;
            if (count === 5) return true;
        }

        // Check vertically
        count = 0;
        for (let i = -4; i <= 4; i++) {
            const r = row + i;
            if (r < 0 || r >= size) continue;
            count = board[r][col] === stone ? count + 1 : 0;
            if (count === 5) return true;
        }

        // Check diagonally (top-left to bottom-right)
        count = 0;
        for (let i = -4; i <= 4; i++) {
            const r = row + i;
            const c = col + i;
            if (r < 0 || r >= size || c < 0 || c >= size) continue;
            count = board[r][c] === stone ? count + 1 : 0;
            if (count === 5) return true;
        }

        // Check diagonally (top-right to bottom-left)
        count = 0;
        for (let i = -4; i <= 4; i++) {
            const r = row + i;
            const c = col - i;
            if (r < 0 || r >= size || c < 0 || c >= size) continue;
            count = board[r][c] === stone ? count + 1 : 0;
            if (count === 5) return true;
        }

        return false;
    }

    function selectMode(isOnePlayer: boolean) {
        isOnePlayerMode = isOnePlayer;
        isGameFinished = false; // Hide modal and start new game
    }
</script>

<div class="modal" style="display: {isOnePlayerMode === null ? 'block' : 'none'}">
    <div class="modal-content">
        <h2>Select Game Mode</h2>
        <button on:click={() => selectMode(true)}>One Player</button>
        <button on:click={() => selectMode(false)}>Two Players</button>
    </div>
</div>

<div style="--lines:{LINES};">
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
                        <div
                            class="cell"
                            role="button"
                            tabindex="0"
                            on:click={() => handleClick(rowIndex, colIndex)}
                            on:keydown={(event) => handleKeyDown(event, rowIndex, colIndex)}
                        >
                            {#if colIndex < LINES}
                                <div class="line vertical-line"></div>
                            {/if}
                            {#if rowIndex < LINES}
                                <div class="line horizontal-line"></div>
                            {/if}
                            {#if isBlindMode && cell !== null}
                                <div class="stone purple-stone"></div>
                            {:else if cell === 0}
                                <div class="stone black-stone"></div>
                            {:else if cell === 1}
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
        background-color: #dcb35c; /* Brown background color */
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
        background: radial-gradient(
            circle at 20% 20%,
            rgba(255, 255, 255, 0.8),
            rgba(255, 255, 255, 0) 70%
        );
        /* Add box-shadow for depth */
        box-shadow: 0 0 5px rgba(0, 0, 0, 0.5);
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

    .modal {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background-color: rgba(0, 0, 0, 0.5); /* Semi-transparent background */
        z-index: 999; /* Ensure modal is above other content */
        display: none; /* Initially hidden */
    }

    .modal-content {
        background-color: #fefefe;
        margin: 20% auto; /* Center modal vertically and horizontally */
        padding: 20px;
        border: 1px solid #888;
        width: 50%; /* Adjust width as needed */
        max-width: 300px; /* Set max-width for responsiveness */
        text-align: center;
    }
</style>
