<script>
  import { browser } from "$app/environment";
  import "./styles.css";

  const defaultPlayerCount = "4";
  let playerCount = browser
    ? localStorage.getItem("playerCount") || defaultPlayerCount
    : defaultPlayerCount;
  $: playerCountInt = parseInt(playerCount, 10);

  $: if (playerCount && browser) {
    localStorage.setItem("playerCount", playerCount);
  }

  /**
   * @type { { name: string, values:number[] }[] }
   */
  let players = [];

  $: activePlayers = players.slice(0, playerCountInt);

  const reset = () => {
    players = [
      {
        name: "1",
        values: [],
      },
      {
        name: "2",
        values: [],
      },
      {
        name: "3",
        values: [],
      },
      {
        name: "4",
        values: [],
      },
      {
        name: "5",
        values: [],
      },
      {
        name: "6",
        values: [],
      },
    ];
  };

  /**
   * @type { { key:string, label:string, calc?:(v:number) => number }[]}
   */
  const scoreCategories = [
    {
      key: "blue-families",
      label: "Blue",
    },
    {
      key: "green-families",
      label: "Green",
    },
    {
      key: "orange-families",
      label: "Orange",
    },
    {
      key: "purple-families",
      label: "Purple",
    },
    {
      key: "red-families",
      label: "Red",
    },
    {
      key: "rare-treasures",
      label: "Rare",
    },
    {
      key: "private-lessons",
      label: "Lessons",
    },
    {
      key: "public-lessons",
      label: "Public",
    },
    {
      key: "rats",
      label: "#Rats",
      calc: (v) => v * -1,
    },
    {
      key: "rooms",
      label: "#Rooms",
      calc: (v) => v * -5,
    },
  ];

  // calculates the current scores
  $: scores = activePlayers.map((player) => {
    // loop over values
    return player.values.reduce((totalScore, curr, index) => {
      let score = curr;

      if (typeof scoreCategories[index].calc === "function") {
        score = /** @type { (v:number) => number } */ (
          scoreCategories[index].calc
        )(score);
      }

      return totalScore + score;
    }, 0);
  });

  // reset the scoreboard
  const handleReset = () => {
    if (!confirm("Are you sure you want to reset the scores?")) return;
    reset();
  };

  // set initial scores
  reset();
</script>

<svelte:head>
  <title>Isle of Cats</title>
  <meta name="description" content="Isle of Cats score app" />
</svelte:head>

<div class="app">
  <div class="controls">
    <select bind:value={playerCount}>
      <option value="1">Players 1</option>
      <option value="2">Players 2</option>
      <option value="3">Players 3</option>
      <option value="4">Players 4</option>
      <option value="5">Players 5</option>
      <option value="6">Players 6</option>
    </select>

    <button on:click={handleReset}>Reset</button>
  </div>

  <table>
    {#if playerCountInt > 1}
      <thead>
        <tr>
          <th />
          {#each activePlayers as player}
            <th>{player.name}</th>
          {/each}
        </tr>
      </thead>
    {/if}
    <tbody>
      {#each scoreCategories as scoreCategorie, i (scoreCategorie.key)}
        <tr class={scoreCategorie.key}>
          <th>{scoreCategorie.label}</th>
          {#each activePlayers as player, j}
            <td
              ><input
                type="number"
                min="0"
                step="1"
                bind:value={activePlayers[j].values[i]}
              /></td
            >
          {/each}
        </tr>
      {/each}
    </tbody>
    <tfoot>
      <tr>
        <th>Total</th>
        {#each scores as score}
          <td>{score}</td>
        {/each}
      </tr>
    </tfoot>
  </table>
</div>
