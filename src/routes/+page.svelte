<script lang="ts">
    import { onMount } from 'svelte';

    // 1. Define types 
    interface MatchaItem {
        name: string;
        cost: number;
        yield: number;
        unlocked: boolean;
        art: string;
    }

    // 2. State variables 
    let matchaPowder = $state(0);
    let zenCoins = $state(0);
    let selected = $state<MatchaItem | null>(null);
    let totalCoinsEarned = $state(0);
    // click upgrade
    let clickPower = $state(1);
    let clickUpgradeCost = $state(10);
    
    // upgrade
    let autoWhisker= $state(0);
    let autoCost = $state(25);
    const powder_per_whisker = 1; //each whisker makes 1g/sec
    let powderPerSec = $derived(autoWhisker * powder_per_whisker)

    // options menu 
let showOptions = $state(false);
let darkMode = $state(true);
let timeSpeed = $state(1);

    // 3. The Menu State Array 
    let items = $state<MatchaItem[]> ([
        {
            name: "Matcha Latte",
            cost: 15,
            yield: 45,
            unlocked: false,
            art: String.raw`
      /
  .--/___.
 /  /     \
 |  ICED  |
 |        |    ta-da!
 | MATCHA |
 |  LATTE |
 \________/`
        },
        {
            name: "Matcha Frappe",
            cost: 40,
            yield: 150,
            unlocked: false,
            art: String.raw`
      /
  .--/___.
(~~~~~~~~~~)
 |  ICED  |
 | MATCHA |
 |        |     ta-da!
 | FRAPPE |
 \________/`
        },
        {
            name: "Uji Matcha Parfait",
            cost: 120,
            yield: 600,
            unlocked: false,
            art: String.raw`
    ( )
  (~~~~)
 (~~~~~~)
  \   /     ta-da! a parfait!
   \ /
    v` 
        }
    ]);

    // 4. Dynamic Viewport
const WHISK = String.raw`
       \ | /
        |||
      [=====]`
const bowlArt = String.raw`
    ~ ~ ~
-------------
\~~~~~~~~~~~/
 \_________/

  a tea bowl`;

const counterArt = String.raw`

    -   -    -
  (_)  (_)  (_)
=================
|   MATCHA      |
|   COUNTER     |
=================`;

const cafeArt = String.raw`
    --------------
   /   MATCHA     \
  /     CAFE       \
 |   []  ||   []   |
 |_______||________|
 `;

 const fieldArt = String.raw`
    ~  *  ~  *  ~  *  ~
   ^^^   ^^^   ^^^    ^^^
  ^^^^^^^^^^^^^^^^^^^^^^^^^
   ^^^   ^^^   ^^^    ^^^
        a matcha field`;

const plantationArt = String.raw`
        /\
       /  \    /\
      /^^^^\  /  \
     /^^^^^^\/^^^^\
    /              \ Matcha Plantation
`;
const STAGES = [bowlArt, counterArt, cafeArt, fieldArt, plantationArt]
let currentArt = $derived(selected ? selected.art : WHISK); 
let stage = $derived(Math.min(Math.floor(Math.log10(totalCoinsEarned+ 1)), STAGES.length - 1));

    // 5. Game loop
    onMount(() => {
        const timer = setInterval(() => {
            matchaPowder += powderPerSec * timeSpeed * 0.1;
            for (const item of items) {
                if(!item.unlocked && matchaPowder >= item.cost) item.unlocked = true;
            }
        }, 100);
        return () => clearInterval(timer);
    });

    // Game Functions
function WhiskMatcha() {
    matchaPowder += clickPower;
}

function make(item: MatchaItem) {
    if (matchaPowder < item.cost) return;
    matchaPowder -= item.cost;
    zenCoins += item.yield;
    totalCoinsEarned += item.yield;
    selected = item;
}

function buyAutoWhisker() {
    if (zenCoins < autoCost) return;
    zenCoins -= autoCost;
    autoWhisker += 1;
    autoCost = Math.ceil(autoCost * 1.15);
}

function buyClickUpgrade(){
    if (zenCoins < clickUpgradeCost) return;
    zenCoins -= clickUpgradeCost;
    clickPower += 1;
    clickUpgradeCost = Math.ceil(clickUpgradeCost * 1.5);
}

function resetAll() {
    matchaPowder = 0;
    zenCoins = 0;
    selected = null;
    clickPower = 1;
    clickUpgradeCost = 10;
    autoWhisker = 0;
    autoCost = 25;
    timeSpeed = 1;
    totalCoinsEarned = 0;
    for (const item of items) item.unlocked = false;
}
</script>

<!-- YOUR CANVAS (The Interface Layout) -->
<main class="terminal" class:light={!darkMode}>
    <button class="options-toggle" onclick={() => (showOptions = !showOptions)} > Options </button>

    {#if showOptions}
        <section class="options">
            
            <label>
                <input type="checkbox" bind:checked={darkMode} />
                Dark mode
            </label>
            <label>
                Time speed: {timeSpeed}x
                <input type="range" min="0.25" max="4" step="0.25" bind:value={timeSpeed} />
            </label>
            <button onclick={resetAll}>Reset All</button>
            <button onclick={() => (showOptions = false)}>Close</button>
        </section>
        {/if}
<h1> Welcome to The Matcha Hub! </h1>

<p> Your matcha business:</p>
<pre>{STAGES[stage]}</pre>
<p> ---------------------</p>
  <pre>{currentArt}</pre>
  <section class="stats">
    <p>Matcha Powder: {Math.floor(matchaPowder)}g</p>
    <p>Zen Coins: {zenCoins}</p>
    <p>Rate: {powderPerSec * timeSpeed}g/sec</p>
  </section>

  <section class="actions">
    <!-- TODO: Add buttons that invoke your custom interaction logic -->
     <button onclick={WhiskMatcha}> Whisk Matcha Powder (+{clickPower}g) </button>
     <button onclick={buyClickUpgrade} disabled={zenCoins < clickUpgradeCost}> Upgrade Whisk (Costs {clickUpgradeCost} coins | +1g per click)</button>
     <button onclick={buyAutoWhisker} disabled={ zenCoins < autoCost}>
        Buy Auto-Whisker (Costs {autoCost} coins | owned : {autoWhisker})
     </button>
    {#each items as item (item.name)}
        {#if item.unlocked}
            <button onclick={()=>make(item)} disabled={matchaPowder < item.cost}>
            Make {item.name} (Costs {item.cost}g | yields {item.yield})
        </button>
        {/if}
     {/each}
  </section>
</main>

<style>
  .terminal {
    font-family: monospace;
    background: #516e51;
    color: #a3b899;
    padding: 2rem;
  }
  .terminal.light{
    background: #bddbbf;
    color: #2f3d2f;
  }
  button {
    font-family: monospace;
    border-radius: 5px;
    padding: 8px;
    cursor: pointer;
    background: #c3a9ba
  }
  .options {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    gap: 8px;
    margin: 1rem 0;
  }
  .options input{
    accent-color: #c3a9ba;
  }
</style>
