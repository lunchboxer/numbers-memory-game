<script>
    import { fade } from "svelte/transition";
    import { cubicOut } from "svelte/easing";
    import Footer from "./lib/footer.svelte";

    import "./app.css";

    let grid = [];
    let scores = [0, 0];
    let currentPlayer = 0;
    let selectedSquares = [];
    let gameOver = false;
    let gameStarted = false;
    let visibleSquares = [];
    let gameWon = false;
    let flashedScore = undefined;

    let matchSound = new Audio("matched.mp3");
    let wrongSound = new Audio("wrong.mp3");
    let winSound = new Audio("win.mp3");

    const numbers = [
        { number: 1, word: "one" },
        { number: 2, word: "two" },
        { number: 3, word: "three" },
        { number: 4, word: "four" },
        { number: 5, word: "five" },
        { number: 6, word: "six" },
        { number: 7, word: "seven" },
        { number: 8, word: "eight" },
        { number: 9, word: "nine" },
        { number: 10, word: "ten" },
    ];

    function initializeGrid() {
        const selectedNumbers = numbers
            .sort(() => 0.5 - Math.random())
            .slice(0, 8);

        grid = selectedNumbers
            .flatMap((num) => [
                { value: num.number.toString(), matched: false, player: null },
                { value: num.word, matched: false, player: null },
            ])
            .sort(() => 0.5 - Math.random());
    }

    function handleSquareClick(index) {
        if (gameOver || grid[index].matched) return;

        if (selectedSquares.includes(index)) {
            selectedSquares = selectedSquares.filter((i) => i !== index);
        } else {
            selectedSquares = [...selectedSquares, index];
        }

        if (selectedSquares.length === 2) {
            const [first, second] = selectedSquares;
            const firstValue = grid[first].value;
            const secondValue = grid[second].value;

            if (
                numbers.find(
                    (n) =>
                        n.number.toString() === firstValue &&
                        n.word === secondValue,
                ) ||
                numbers.find(
                    (n) =>
                        n.word === firstValue &&
                        n.number.toString() === secondValue,
                )
            ) {
                grid[first].matched = true;
                grid[second].matched = true;
                grid[first].player = currentPlayer;
                grid[second].player = currentPlayer;
                scores[currentPlayer] += 20;
                flashScore(20);
                matchSound.play();
                checkForWin();
            } else {
                wrongSound.play();
                currentPlayer = (currentPlayer + 1) % 2;
            }
            setTimeout(() => {
                selectedSquares = [];
            }, 500);
            grid = [...grid];
        }
    }

    function flashScore(amount) {
        flashedScore = amount;
        setTimeout(() => {
            flashedScore = undefined;
        }, 400);
    }

    function checkForWin() {
        if (grid.every((square) => square.matched)) {
            gameWon = true;
            endGame();
        }
    }

    function endGame() {
        gameOver = true;
        if (gameWon) {
            winSound.play();
        }
    }

    function startGame() {
        gameStarted = true;
        gameOver = false;
        gameWon = false;
        scores = [0, 0];
        currentPlayer = 0;
        selectedSquares = [];
        initializeGrid();
        staggerSquareAppearance();
    }

    function staggerSquareAppearance() {
        const totalDuration = 300;
        const intervalDuration = totalDuration / grid.length;

        visibleSquares = [];
        const indexes = Array.from({ length: grid.length }, (_, i) => i);

        function showNextSquare() {
            if (indexes.length > 0) {
                const randomIndex = Math.floor(Math.random() * indexes.length);
                const squareIndex = indexes.splice(randomIndex, 1)[0];
                visibleSquares = [...visibleSquares, squareIndex];
                setTimeout(showNextSquare, intervalDuration);
            }
        }

        showNextSquare();
    }
</script>

<main>
    <div class="game-info">
        <h1 class="title">Numbers Memory Game</h1>
        <p>Player 1: {scores[0]} | Player 2: {scores[1]}</p>
        <p>Current Player: {currentPlayer + 1}</p>
    </div>
    {#if !gameStarted}
        <button class="start-button" on:click={startGame}>Start Game</button>
    {:else}
        <div class="grid" class:game-over={gameOver}>
            {#each grid as square, i}
                {#if visibleSquares.includes(i)}
                    <button
                        class="square"
                        class:selected={selectedSquares.includes(i)}
                        class:matched={square.matched}
                        class:player1={square.player === 0}
                        class:player2={square.player === 1}
                        on:click={() => handleSquareClick(i)}
                        transition:fade={{ duration: 500, easing: cubicOut }}
                    >
                        {square.matched ? "" : square.value}
                    </button>
                {/if}
            {/each}
        </div>

        {#if flashedScore}
            <div class="flashed-score" transition:fade={{ duration: 300 }}>
                +{flashedScore}
            </div>
        {/if}

        {#if gameOver}
            <div class="game-over-content" transition:fade={{ duration: 300 }}>
                <h2>🎉 Game Over!</h2>
                <h3>Final Scores</h3>
                <p>Player 1: {scores[0]}</p>
                <p>Player 2: {scores[1]}</p>
                {#if scores[0] === scores[1]}
                    <p>It's a tie!</p>
                {:else}
                    <p>Winner: Player {scores[0] > scores[1] ? 1 : 2}</p>
                {/if}
                <button class="play-again-button" on:click={startGame}>
                    Play Again
                </button>
            </div>
        {/if}
    {/if}
</main>
<Footer />

<style>
    main {
        display: flex;
        flex-direction: column;
        justify-content: space-between;
        align-items: center;
        max-width: 600px;
        padding: 0 0.5rem;
        margin: 0 auto;
        position: relative;
        min-height: calc(100vh - 3.8rem);
    }

    .game-info {
        display: flex;
        flex-direction: column;
        padding: 1rem;
        justify-content: center;
        align-items: center;
        width: 100%;
        margin-bottom: 20px;
    }

    .grid {
        display: grid;
        grid-template-columns: repeat(4, 1fr);
        gap: 10px;
        width: 100%;
    }

    .grid.game-over {
        opacity: 0.5;
        pointer-events: none;
        transition: opacity 0.3s;
    }
    .title {
        color: #6200ea;
    }

    .square {
        aspect-ratio: 1;
        font-size: clamp(16px, 3vw, 28px);
        font-family: inherit;
        border: 1px solid #333;
        background-color: var(--bg-color);
        border-radius: 5px;
        color: var(--bg-color);
        cursor: pointer;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        transition: opacity 0.5s;
        padding: 0;
    }
    @media (max-width: 600px) {
        .game-info {
            flex-wrap: wrap;
        }
        .title {
            margin: 0 0 0.5rem;
            padding: 0;
            width: 100%;
            order: -1;
            clear: both;
        }
    }

    .square:hover:not(:disabled) {
        border: 2px solid #fff;
    }

    .square.selected {
        background-color: #f6f6f6;
        border: 2px solid #6200ea;
        box-shadow: 0 3px 6px rgba(0, 0, 0, 0.2);
    }

    .square.matched {
        opacity: 0.7;
        cursor: default;
        border: none;
        box-shadow: none;
    }

    .square.player1 {
        background-color: #ffcccb;
    }

    .square.player2 {
        background-color: #add8e6;
    }

    .game-over-content {
        position: absolute;
        top: 50%;
        transform: translateY(-50%);
        background-color: var(--bg-color);
        padding: 20px;
        border: 1px solid #ccc;
        border-radius: 10px;
        text-align: center;
        z-index: 10;
    }
    .game-over-content h3 {
        margin: 2rem 0 1rem;
    }
    .game-over-content h2 {
        margin: 1rem 0 2rem;
    }

    .flashed-score {
        position: absolute;
        top: 50%;
        transform: translateY(-50%);
        color: #6200ea;
        font-size: 2rem;
    }

    .start-button,
    .play-again-button {
        font-size: 1rem;
        padding: 1rem 2rem;
        margin: 1rem;
        background-color: #6200ea;
        color: white;
        border: none;
        border-radius: 5px;
        cursor: pointer;
        transition: background-color 0.3s;
    }

    .start-button:hover,
    .play-again-button:hover {
        background-color: #b388ff;
    }
</style>

