<script lang="ts">
    let board = Array(9).fill(null).map(() => Array(9).fill(null));
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
            return;
        }
        if (moves.length === 81) {
            alert('It\'s a draw!');
            return;
        }
        moves.push([row, col]);
        isBlackTurn = !isBlackTurn;
    }

    function restartGame() {
        // confirm
        if (!confirm('Are you sure you want to restart the game?')) {
            return;
        }
        board = Array(9).fill(null).map(() => Array(9).fill(null));
        moves = [];
        isBlackTurn = true;
    }

    function undoMove() {
        if (moves.length === 0) return;
        const [row, col] = moves.pop();
        board[row][col] = null;
        isBlackTurn = !isBlackTurn;
    }

    function checkConsecutiveStones() {
        const size = 9;

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
        grid-template-columns: repeat(9, 40px);
        grid-template-rows: repeat(9, 40px);
        grid-gap: 0px;
        background-color: #DCB35C; /* Brown background color */
        padding: 5px; /* Add padding for the grid lines */
        position: relative; /* Add position relative to the board */
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
        width: 40px;
        height: 40px;
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
    }

    .black-stone {
        background-color: #000;
    }

    .white-stone {
        background-color: #fff;
        border: 1px solid #000;
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
        margin-bottom: 20px; /* Adjust the margin as needed */
    }
</style>

<div class="container">
    <div class="title-container">
        <h1>Connect Five</h1>
        <p>{isBlackTurn ? 'Black' : 'White'}'s turn</p>
    </div>
    <div class="board-container">
        <div class="board">
            {#each board as row, rowIndex}
                {#each row as cell, colIndex}
                    <div class="cell" on:click={() => handleClick(rowIndex, colIndex)}>
                        {#if colIndex < 9}
                            <div class="line vertical-line"></div>
                        {/if}
                        {#if rowIndex < 9}
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
            <button on:click={undoMove}>Undo Move</button>
        </div>
        <div>
            <label>
                Blind Mode
                <input type="checkbox" bind:checked={isBlindMode} />
            </label>
        </div>
    </div>
</div>
