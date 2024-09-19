<script lang="ts">
    let tiles: number[] = Array.from({ length: 16 }, (_, i) => i + 1);
    let draggedTile: number | null = null;
    let touchTile: HTMLElement | null = null;
    let winMessage: string = "";
    let timer: number = 0;
    let timerInterval: NodeJS.Timeout | null = null;
    let showRetryButton: boolean = false;

    // Sommes des lignes, colonnes, diagonales et carrés
    let rowSums = [0, 0, 0, 0];
    let colSums = [0, 0, 0, 0];
    let diagSums = [0, 0];
    let squareSums = [0, 0, 0, 0, 0];

// Démarrer le timer au démarrage
    startTimer();
    // Appel de la fonction pour initialiser les sommes au démarrage
    checkSums();

    function handleDragStart(event: DragEvent, tile: number): void {
        draggedTile = tile;
        if (event.target instanceof HTMLElement) {
            event.target.style.opacity = '0.5';
        }
    }

    function handleDragOver(event: DragEvent): void {
        event.preventDefault();
    }

    function handleDrop(event: DragEvent, tile: number): void {
        event.preventDefault();
        if (draggedTile === null) return;

        swapTiles(tile);
        if (event.target instanceof HTMLElement) {
            event.target.style.opacity = '1';
        }
    }

    function handleTouchStart(event: TouchEvent, tile: number): void {
        draggedTile = tile;
        touchTile = event.target as HTMLElement;
        touchTile.style.opacity = '0.5';
    }

    function handleTouchMove(event: TouchEvent): void {
        event.preventDefault(); // Empêche le scrolling lors du déplacement
    }

    function handleTouchEnd(event: TouchEvent, tile: number): void {
        const elementUnderTouch = document.elementFromPoint(
            event.changedTouches[0].clientX,
            event.changedTouches[0].clientY
        ) as HTMLElement;

        if (touchTile && elementUnderTouch && elementUnderTouch !== touchTile) {
            const targetTile = elementUnderTouch.innerText;
            if (targetTile) {
                const targetTileNumber = parseInt(targetTile);
                swapTiles(targetTileNumber);
            }
        }

        touchTile.style.opacity = '1';
        touchTile = null;
        draggedTile = null;
    }

    function swapTiles(targetTile: number): void {
        if (draggedTile === null) return;

        const draggedIndex = tiles.indexOf(draggedTile);
        const droppedIndex = tiles.indexOf(targetTile);
        [tiles[draggedIndex], tiles[droppedIndex]] = [tiles[droppedIndex], tiles[draggedIndex]];

        draggedTile = null; // Reset après l'échange

        checkSums(); // Vérifie les sommes après l'échange
    }
   function startTimer(): void {
        timer = 0;
        timerInterval = setInterval(() => {
            timer++;
        }, 1000);
    }

    function stopTimer(): void {
        if (timerInterval) {
            clearInterval(timerInterval);
        }
    }

    function checkSums(): void {
        // Réinitialise les sommes
        rowSums = [0, 0, 0, 0];
        colSums = [0, 0, 0, 0];
        diagSums = [0, 0];
        squareSums = [0, 0, 0, 0, 0];

        // Calcul des lignes et colonnes
        for (let i = 0; i < 4; i++) {
            for (let j = 0; j < 4; j++) {
                rowSums[i] += tiles[i * 4 + j];
                colSums[j] += tiles[i * 4 + j];
            }
        }

        // Calcul des diagonales
        diagSums[0] = tiles[0] + tiles[5] + tiles[10] + tiles[15];
        diagSums[1] = tiles[3] + tiles[6] + tiles[9] + tiles[12];

        // Calcul des carrés
        squareSums[0] = tiles[0] + tiles[1] + tiles[4] + tiles[5]; // Carré haut-gauche
        squareSums[1] = tiles[2] + tiles[3] + tiles[6] + tiles[7]; // Carré haut-droit
        squareSums[2] = tiles[8] + tiles[9] + tiles[12] + tiles[13]; // Carré bas-gauche
        squareSums[3] = tiles[10] + tiles[11] + tiles[14] + tiles[15]; // Carré bas-droit
        squareSums[4] = tiles[5] + tiles[6] + tiles[9] + tiles[10]; // Carré central

        // Vérifie si toutes les sommes sont égales à 34
        if (rowSums.every(sum => sum === 34) && 
            colSums.every(sum => sum === 34) &&
            diagSums.every(sum => sum === 34) &&
            squareSums.every(sum => sum === 34)) {
            winMessage = "C'est gagné !";
            stopTimer(); // Arrête le timer lorsque le jeu est gagné
            showRetryButton = true; // Affiche le bouton Retry
        } else {
            winMessage = ""; // Réinitialise si ce n'est pas encore gagné
            showRetryButton = false; // Cache le bouton Retry
        }
    }
    function resetGame(): void {
        tiles = Array.from({ length: 16 }, (_, i) => i + 1); // Réinitialiser les tuiles
        startTimer(); // Redémarrer le timer
        checkSums(); // Recalculer les sommes
        showRetryButton = false; // Cacher le bouton Retry pendant le jeu
    }
</script>

<style>
    .grid-container {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        height: 100svh;
        background-color: #f5f5f5;
        padding: 20px;
        box-sizing: border-box; /* Ensure padding doesn't affect height */
    }

    .grid {
        display: grid;
        grid-template-columns: repeat(4, 1fr);
        grid-template-rows: repeat(4, 1fr);
        gap: 10px;
        width: 100%;
        max-width: 400px;
        height: auto;
    }

    .tile {
        display: flex;
        justify-content: center;
        align-items: center;
        background-color: #4caf50;
        color: white;
        font-size: calc(1rem + 1vw);
        border-radius: 5px;
        cursor: grab;
        user-select: none;
        transition: transform 0.2s, font-size 0.2s;
        aspect-ratio: 1;
    }

    .tile:hover {
        transform: scale(1.05);
    }

    .tile:active {
        background-color: #388e3c;
        cursor: grabbing;
    }

    .message {
        position: absolute;
        width: 100%;
        height: 100%;
        background-color: rgba(0, 0, 0, 0.493); 
        font-size: 6rem; /* Adjusted size for better mobile view */
        color: #ff5722;
        text-shadow: 5px 10px 0 #9e391a;
        font-weight: 800;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        text-align: center;
        gap: 20px;
        padding: 0 10px; /* Added padding for smaller screens */
        box-sizing: border-box; /* Ensure padding doesn't affect layout */
    }

    .timer {
        font-size: 2rem; /* Adjusted size for better mobile view */
        color: #00ff0d;
        text-shadow: 3px 4px 0 #388e3c;
    }

    .sums-container {
        display: grid;
        grid-template-columns: repeat(2, 1fr); /* Two columns layout */
        gap: 10px;
        width: 100%;
        max-width: 600px;
        margin-top: 20px;
    }

    .sum-block {
        width: 100%;
        margin: 0;
        padding: 10px;
        border: 1px solid #ccc;
        border-radius: 5px;
        background-color: #e0e0e0;
        text-align: center;
        box-sizing: border-box; /* Ensure padding doesn't affect width */
    }

    .retry-button {
        margin-top: 20px;
        padding: 10px 20px;
        font-size: 1.2rem;
        color: #fff;
        background-color: #4caf50;
        border: none;
        border-radius: 5px;
        cursor: pointer;
    }

    .retry-button:hover {
        background-color: #388e3c;
    }

    /* Responsive */
    @media (max-width: 600px) {
        .grid {
            max-width: 100%;
        }

        .tile {
            font-size: calc(1rem + 5vw);
        }

        .message {
            font-size: 3rem;
            padding: 0 5px; /* Reduced padding for smaller screens */
            text-shadow: 3px 5px 0 #9e391a;
        }

        .timer {
            font-size: 1.5rem;
            text-shadow: 2px 3px 0 #388e3c;
        }

        .sums-container {
        
            font-size: 0.9rem; /* Adjusted size for better mobile view */
        }

        .sum-block {
            padding: 5px;
            font-size: 0.9rem; /* Adjusted size for better mobile view */
        }

        .retry-button {
            font-size: 1rem;
            padding: 8px 16px; /* Adjusted padding for smaller screens */
        }
    }
    h1 {
        margin-bottom: 20px;
        color: #00ff0d;
        text-shadow: 2px 2px 0 #388e3c;
    }
    h5 {
        color :#4caf50;
        text-align: center;
        margin-bottom: 20px;
    }
</style>


<div class="grid-container"> 
    <h1>Tile Swap 34</h1>
    <h5>Faites en sorte que toutes les ligne, colonnes, diagonales, et carré (1,2,5,6/3,4,7,8/9,10,13,14/11,12,15,16/6,7,10,11)</h5>
    <div class="grid">
        {#each tiles as tile (tile)}
            <div
                class="tile"
                draggable="true"
                on:dragstart={(event) => handleDragStart(event, tile)}
                on:dragover={handleDragOver}
                on:drop={(event) => handleDrop(event, tile)}
                on:touchstart={(event) => handleTouchStart(event, tile)}
                on:touchmove={handleTouchMove}
                on:touchend={(event) => handleTouchEnd(event, tile)}>
                {tile}
            </div>
        {/each}
    </div>
    {#if winMessage}
    <div class="message">{winMessage}<br>
        <div class="timer">Temps écoulé: {Math.floor(timer / 60)}:{(timer % 60).toString().padStart(2, '0')}</div>
        {#if showRetryButton}
        <button class="retry-button" on:click={resetGame}>Retry</button>
    {/if}
    </div>
   
    {/if}
    <div class="sums-container">
        <div class="sum-block">
            <strong>Lignes:</strong>
            {#each rowSums as sum, i}
                <div>Ligne {i + 1}: {sum}</div>
            {/each}
        </div>
        <div class="sum-block">
            <strong>Colonnes:</strong>
            {#each colSums as sum, i}
                <div>Colonne {i + 1}: {sum}</div>
            {/each}
        </div>
        <div class="sum-block">
            <strong>Diagonales:</strong>
            <div>Diagonale 1: {diagSums[0]}</div>
            <div>Diagonale 2: {diagSums[1]}</div>
        </div>
        <div class="sum-block">
            <strong>Carrés:</strong>
            {#each squareSums as sum, i}
                <div>Carré {i + 1}: {sum}</div>
            {/each}
        </div>
    </div>
   
        <button class="retry-button" on:click={resetGame}>Retry</button>
</div>
