<script lang="ts">
    let tiles: number[] = Array.from({ length: 16 }, (_, i) => i + 1);
    let draggedTile: number | null = null;

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

    // Gestion des événements tactiles
    function handleTouchStart(event: TouchEvent, tile: number): void {
        draggedTile = tile;
        if (event.target instanceof HTMLElement) {
            event.target.style.opacity = '0.5';
        }
    }

    function handleTouchMove(event: TouchEvent): void {
        event.preventDefault(); // Empêche le scrolling lors du déplacement
    }

    function handleTouchEnd(event: TouchEvent, tile: number): void {
        if (draggedTile === null) return;

        swapTiles(tile);
        if (event.target instanceof HTMLElement) {
            event.target.style.opacity = '1';
        }
    }

    function swapTiles(targetTile: number): void {
        if (draggedTile === null) return;

        const draggedIndex = tiles.indexOf(draggedTile);
        const droppedIndex = tiles.indexOf(targetTile);
        [tiles[draggedIndex], tiles[droppedIndex]] = [tiles[droppedIndex], tiles[draggedIndex]];

        draggedTile = null; // Reset après l'échange
    }
</script>

<style>
    .grid-container {
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100svh;
        background-color: #f5f5f5;
        padding: 20px;
    }

    .grid {
        display: grid;
        grid-template-columns: repeat(4, 1fr);
        grid-template-rows: repeat(4, 1fr);
        gap: 10px;
        width: 100%;
        max-width: 400px; /* Limite max sur grands écrans */
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
        aspect-ratio: 1; /* Carrés automatiques */
    }

    .tile:hover {
        transform: scale(1.05);
    }

    .tile:active {
        background-color: #388e3c;
        cursor: grabbing;
    }

    /* Responsive */
    @media (max-width: 600px) {
        .grid {
            max-width: 100%;
        }

        .tile {
            font-size: calc(1rem + 5vw);
        }
    }
</style>

<div class="grid-container">
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
</div>
