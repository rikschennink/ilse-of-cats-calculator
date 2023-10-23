<script>
  import { browser } from "$app/environment";
  import "./styles.css";

  const gameModes = [
    ["standard", "Standard game"],
    ["family", "Family game"],
    ["beasts", "Beasts game"],
    ["events", "Events game"],
    ["beasts,events", "Beasts & Events game"],
  ];

  let gameMode = gameModes[0][0];

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

  const ALL_GAME_MODES = ["standard", "family", "events", "beasts"];

  /**
   * @type { { key:string, label:string, calc?:(v:number) => number, mode: string[] }[]}
   */
  const scoreCategories = [
    {
      key: "blue-families",
      label: "Blue",
      mode: ALL_GAME_MODES,
    },
    {
      key: "green-families",
      label: "Green",
      mode: ALL_GAME_MODES,
    },
    {
      key: "orange-families",
      label: "Orange",
      mode: ALL_GAME_MODES,
    },
    {
      key: "purple-families",
      label: "Purple",
      mode: ALL_GAME_MODES,
    },
    {
      key: "red-families",
      label: "Red",
      mode: ALL_GAME_MODES,
    },
    {
      key: "rare-treasures",
      label: "Rare",
      mode: ALL_GAME_MODES.filter((mode) => mode !== "family"),
    },
    {
      key: "private-lessons",
      label: "Lessons",
      mode: ALL_GAME_MODES,
    },
    {
      key: "public-lessons",
      label: "Public",
      mode: ALL_GAME_MODES.filter((mode) => mode !== "family"),
    },
    {
      key: "events",
      label: "Events",
      mode: ["events"],
    },
    {
      key: "beasts",
      label: "Beasts",
      mode: ["beasts"],
    },
    {
      key: "rats",
      label: "#Rats",
      calc: (v) => v * -1,
      mode: ALL_GAME_MODES,
    },
    {
      key: "rooms",
      label: "#Rooms",
      calc: (v) => v * -5,
      mode: ALL_GAME_MODES,
    },
  ];

  $: activeScoreCategories = scoreCategories.filter((category) => {
    const modes = gameMode.split(",");
    return modes.some((mode) => category.mode.includes(mode));

    // if () {

    // }
    // if (category.mode.includes(gameMode)) {
    // 	return category;
    // }
  });

  // calculates the current scores
  $: scores = activePlayers.map((player) => {
    // loop over values
    return player.values.reduce((totalScore, curr, index) => {
      let score = curr;

      if (typeof activeScoreCategories[index].calc === "function") {
        score = /** @type { (v:number) => number } */ (
          activeScoreCategories[index].calc
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

<!--

<fieldset>
	<input type="radio" value="8">
	<input type="radio" value="11">
	<input type="radio" value="15">
	<input type="radio" value="20">
	<input type="radio" value="25">
	<input type="radio" value="30">
	<input type="radio" value="🖊️">
</fieldset>

-->

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

    <select bind:value={gameMode}>
      {#each gameModes as [value, label] (value)}
        <option {value}>{label}</option>
      {/each}
    </select>

    <button on:click={handleReset}>Reset</button>
  </div>

  <table>
    {#if playerCountInt > 1}
      <thead>
        <tr>
          <th />
          {#each activePlayers as player}
            <th>P{player.name}</th>
          {/each}
        </tr>
      </thead>
    {/if}
    <tbody>
      {#each activeScoreCategories as scoreCategorie, i (scoreCategorie.key)}
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
