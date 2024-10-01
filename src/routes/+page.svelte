<script lang="ts">
    import { onMount } from "svelte";
    import refreshCwSvg from '$lib/assets/refresh-cw.svg';
    import { fade } from "svelte/transition";

    let wrapper: HTMLDivElement;

    const boxSizePx: number = 75;
    let numRows: number = 1;
    let numCols: number = 1;

    let currentPlayer: number = 0;
    let displayWinner: boolean = false;

    const updateGrid = () => {
        numRows = Math.floor(wrapper.clientHeight / boxSizePx);
        numCols = Math.floor(wrapper.clientWidth / boxSizePx);

        wrapper.style.setProperty('--rows', numRows as unknown as string);
        wrapper.style.setProperty('--cols', numCols as unknown as string);
    }

    onMount(() => {
        updateGrid();
        window.onresize = () => updateGrid();
    })

    const fadeOutTile = (tile: HTMLElement | null, delayMs: number) => {
        if (tile && tile.getAttribute('data-display') === '1') {
            setTimeout(() => {
                tile.style.opacity = '0';
                tile.style.cursor = 'default'
                tile.setAttribute('data-display', '0');
            }, delayMs);
        }
    }

    const handleTileClick = (index: number) => {
        const tile = document.getElementById(`tile-${index}`) as HTMLElement;
        if (tile.getAttribute('data-display') === '0') return;

        const tileCol = Math.floor(index % numCols);
        const tileRow = Math.floor(index / numCols);

        for (let i = tileRow; i > -1; i--) {
            for (let j = tileCol; j < numCols; j++) {
                const currTile = document.getElementById(`tile-${(i * numCols) + j}`)
                const distance = Math.floor(Math.hypot((tileRow - i), (tileCol - j)));
                fadeOutTile(currTile, distance * 20)
            }
        }

        displayWinner = index === (numRows - 1) * numCols;
        currentPlayer = currentPlayer === 0 ? 1 : 0;
    }

    const handleResetClick = () => {
        currentPlayer = 0;
        displayWinner = false;
        const tiles = document.querySelectorAll('.tile');
        for (const tile of tiles) {
            const htmlTile = tile as HTMLElement;
            htmlTile.style.opacity = '1';
            htmlTile.style.cursor = 'pointer';
            htmlTile.setAttribute('data-display', '1')
        }
    }
</script>

<main class={`h-screen w-screen py-10 flex flex-col justify-between items-center`}>
    <h1 class={`text-7xl`}>Chomp!</h1>
    <div class={`flex flex-col gap-4 justify-center items-center flex-grow w-full`}>
        <p class={`text-5xl my-2`}>Player {currentPlayer + 1}'s Turn!</p>
        <div bind:this={wrapper} class={`wrapper relative h-[20%] md:h-[75%] w-[80%] grid place-items-center border-4     border-neutral-800`} style={`grid-template-columns: repeat(var(--cols), 1fr); grid-template-rows: repeat(var(--rows), 1ft);`}>
            {#each Array(numRows * numCols) as item, index (index)}
                <button data-display="1" id={`tile-${index}`} class={`tile h-full w-full`} on:click={() => handleTileClick(index)}>
                    {#if (index === (numRows - 1) * numCols)}
                        <div class={`absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2`}>
                            <svg xmlns="http://www.w3.org/2000/svg" class="lucide lucide-skull stroke-neutral-100/40" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                                <path d="m12.5 17-.5-1-.5 1h1z"/><path d="M15 22a1 1 0 0 0 1-1v-1a2 2 0 0 0 1.56-3.25 8 8 0 1 0-11.12 0A2 2 0 0 0 8 20v1a1 1 0 0 0 1 1z"/>
                                <circle cx="15" cy="12" r="1"/>
                                <circle cx="9" cy="12" r="1"/>
                            </svg>
                        </div>
                    {/if}
                </button>
            {/each}
            {#if displayWinner}
                <p in:fade={{duration: 300}} class={`absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 text-3xl text-neutral-800`}>Player {currentPlayer === 0 ? 1 : 2} Wins!</p>
            {/if}
        </div>
        <button on:click={() => handleResetClick()} class={`flex flex-row items-center gap-2 px-8 py-4 text-4xl border-2 border-neutral-800 rounded-xl`}>
            <img src={refreshCwSvg} alt={`reset`}/>
            <p>Reset!</p>
        </button>
    </div>
</main>