<script lang="ts">
import { emoji } from './emoji';

type State = 'start' | 'playing' | 'paused' | 'won' | 'lost';

let state: State = 'start'; 
let size = 20
let grid = createGrid();
let maxMatches = grid.length / 2;
let selected: number[] = [];
let matches: string[] = [];
let timerId: number | null = null;
let time = 20

function startTimer(){
  function countdown(){
    state !== 'paused' && (time -= 1)
  }
 timerId =  setInterval(countdown, 1000)
}

function createGrid(){
    let cards = new Set<string>()
    let maxSize = size /2

    while (cards.size < maxSize){
        const randomIndex = Math.floor(Math.random() * emoji.length)
        cards.add(emoji[randomIndex])
    }

    return shuffle([...cards, ...cards])
}

function shuffle<Items>(array: Items[]){
    return array.sort(() => Math.random() - 0.5)
}

function selectCard(cardIndex: number){
    selected = selected.concat(cardIndex)
}

function matchCards(){
   const [first, second] = selected

   if(grid[first] === grid[second]){
       matches = matches.concat(grid[first])
   }
  
   setTimeout(() => {
       selected = []
   }, 300)
}

function gameWon(){
    state = 'won'
}

$: selected.length === 2 && matchCards()
$: maxMatches === matches.length && gameWon()

$: console.log({state, selected, matches})



</script>


{#if state === 'start'}
    <h1>Memory Game</h1>
    <button on:click={() => state = 'playing'}>Start Game</button>
{/if}

{#if state === 'playing'}
<div class="matchers">
    {#each matches as card}
        <div>{card}</div>
    {/each}
</div>

    <div class="cards">
            {#each grid as card, cardIndex}

            {@const isSelected = selected.includes(cardIndex)}
            {@const isSelectedOrMatched = selected.includes(cardIndex) || matches.includes(card)}
            {@const match = matches.includes(card)}
              <button 
              class="card"
              class:selected = {isSelected}
              disabled={isSelectedOrMatched}
              on:click={() => selectCard(cardIndex)}
              >
                  <div class:match>{card}</div>
              </button>
                
            {/each}
    </div>
{/if}

{#if state === 'won'}
    <h2>You Won!</h2>
    <button on:click={() => state = 'playing'}>Play Again</button>
{/if}

{#if state === 'lost'}
    <h2>You Lost!</h2>
    <button on:click={() => state = 'playing'}>Try Again</button>
    
{/if}


<style>
    .cards {
        display: grid;
        grid-template-columns: repeat(5, 1fr);
        gap: 0.4rem;
    }

    .card {
        height: 140px;
        width: 140px;
        font-size: 4rem;
        background-color: var(--bg-2);

        &.selected{
            border: 4px solid var(--border);
        }

        & .match {
           transition: opacity 0.3s ease-out;
           opacity: 0.4;
        }
    }

    .matchers {
        display: flex;
        gap: 1rem;
        margin-block: 2rem;
        font-size: 3rem;
    }
</style>

