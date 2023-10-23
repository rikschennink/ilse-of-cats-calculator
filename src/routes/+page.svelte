<script>
  import { browser } from "$app/environment";
  import { writable } from "svelte/store";
  import "./styles.css";

  const ALL_GAME_MODES = ["standard", "family", "events", "beasts"];

  const GAME_MODES = [
    ["standard", "Standard game"],
    ["family", "Family game"],
    ["beasts", "Beasts game"],
    ["events", "Events game"],
    ["beasts,events", "Beasts & Events game"],
  ];

  const PLAYERS = ["P1", "P2", "P3", "P4", "P5", "P6"];

  const FAMILY_PRESETS = [0, 8, 11, 15, 20, 25];

  /**
   * @type { { key:string, label:string, mode: string[], presets?:number[] }[]}
   */
  const SCORE_CATEGORIES = [
    {
      key: "blue-families",
      label: "Blue",
      mode: ALL_GAME_MODES,
      presets: FAMILY_PRESETS,
    },
    {
      key: "green-families",
      label: "Green",
      mode: ALL_GAME_MODES,
      presets: FAMILY_PRESETS,
    },
    {
      key: "orange-families",
      label: "Orange",
      mode: ALL_GAME_MODES,
      presets: FAMILY_PRESETS,
    },
    {
      key: "purple-families",
      label: "Purple",
      mode: ALL_GAME_MODES,
      presets: FAMILY_PRESETS,
    },
    {
      key: "red-families",
      label: "Red",
      mode: ALL_GAME_MODES,
      presets: FAMILY_PRESETS,
    },
    {
      key: "rare-treasures",
      label: "Rare",
      mode: ALL_GAME_MODES.filter((mode) => mode !== "family"),
      presets: [0, 3, 6, 9, 12, 15],
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
      presets: [0, 5, 10, 15, 20, 25],
    },
    {
      key: "rats",
      label: "Rats",
      mode: ALL_GAME_MODES,
      presets: [0, -1, -2, -3, -4, -5],
    },
    {
      key: "rooms",
      label: "Rooms",
      mode: ALL_GAME_MODES,
      presets: [0, -5, -10, -15, -20],
    },
  ];

  const reset = () => {
    const playerCount = $state ? $state.playerCount : 4;
    const gameMode = $state ? $state.gameMode : "standard";
    $state = {
      gameMode,
      playerCount,
      players: PLAYERS.reduce((players, key) => {
        // empty player score object
        players[key] = {};
        return players;
      }, {}),
    };
  };

  // read current game state
  const state = writable();

  if (browser) {
    try {
      const stateStr = localStorage.getItem("STATE");
      if (stateStr) {
        $state = JSON.parse(stateStr);
      } else {
        reset();
      }
    } catch (err) {
      console.error(err);
    }
  }

  $: if ($state && browser) {
    try {
      localStorage.setItem("STATE", JSON.stringify($state));
    } catch (err) {
      console.error(err);
    }
  }

  $: activeScoreCategories = $state
    ? SCORE_CATEGORIES.filter((category) => {
        /** @type {string[]} */
        const modes = $state.gameMode.split(",");
        return modes.some((mode) => category.mode.includes(mode));
      })
    : [];

  // calculates the current scores
  $: activePlayers = $state
    ? Object.keys($state.players).slice(0, $state.playerCount)
    : [];

  $: scores = activePlayers.map((player) => {
    /**
     * @type { { [key:string]:number } }
     */
    const playerScores = $state.players[player];
    let playerScore = 0;
    Object.entries(playerScores).forEach(([key, value]) => {
      const scoreCategorie = SCORE_CATEGORIES.find(
        (category) => category.key === key
      );
      if (scoreCategorie && scoreCategorie.calc) {
        playerScore += scoreCategorie.calc(value);
      } else {
        playerScore += value;
      }
    });

    return playerScore;
  });

  // reset the scoreboard
  const handleReset = () => {
    if (!confirm("Are you sure you want to reset the scores?")) return;
    reset();
  };

  let focussedPlayer = "";
  let focussedScoreCategory = "";
  /**
   *
   * @param {string} player
   * @param {string} scoreCategory
   */
  const handleFocus = (player, scoreCategory) => {
    focussedPlayer = player;
    focussedScoreCategory = scoreCategory;
  };

  const handleFocusNext = (player, scoreCategory) => {
    const currentPlayerIndex = activePlayers.indexOf(player);
    const nextPlayerIndex =
      currentPlayerIndex + 1 === activePlayers.length
        ? 0
        : currentPlayerIndex + 1;

    if (nextPlayerIndex === 0) {
      const currentScoreCategoryIndex = activeScoreCategories.findIndex(
        (category) => category.key === scoreCategory
      );
      const nextScoreCategoryIndex =
        currentScoreCategoryIndex + 1 === activeScoreCategories.length
          ? 0
          : currentScoreCategoryIndex + 1;

      focussedScoreCategory = activeScoreCategories[nextScoreCategoryIndex].key;
    }

    focussedPlayer = activePlayers[nextPlayerIndex];

    // focus the field
    const fieldId = `${focussedPlayer}-${focussedScoreCategory}`;
    document.getElementById(fieldId)?.focus();
  };

  $: console.log(focussedPlayer, focussedScoreCategory);
</script>

<svelte:head>
  <title>Isle of Cats score app</title>
  <meta name="description" content="Isle of Cats score app" />
</svelte:head>

{#if browser}
  <div class="app">
    <div class="controls">
      <select bind:value={$state.playerCount}>
        {#each PLAYERS as _, i}
          <option value={i + 1}>Players {i + 1}</option>
        {/each}
      </select>

      <select bind:value={$state.gameMode}>
        {#each GAME_MODES as [value, label] (value)}
          <option {value}>{label}</option>
        {/each}
      </select>

      <button on:click={handleReset}>Reset</button>
    </div>

    <table>
      {#if $state.playerCount > 1}
        <thead>
          <tr>
            <th />
            {#each activePlayers as player}
              <th>{player}</th>
            {/each}
          </tr>
        </thead>
      {/if}
      <tbody>
        {#each activeScoreCategories as { key, label, presets } (key)}
          <tr class={key}>
            <th>{label}</th>
            {#each activePlayers as player}
              <td
                ><input
                  type="number"
                  min="0"
                  step="1"
                  placeholder="0"
                  id={`${player}-${key}`}
                  on:focus={() => handleFocus(player, key)}
                  bind:value={$state.players[player][key]}
                />
                <div
                  class="preset-pointer"
                  data-active={`${
                    focussedPlayer === player && focussedScoreCategory === key
                      ? "true"
                      : "false"
                  }`}
                />
              </td>
            {/each}
          </tr>
          {#if presets}
            <tr
              class="presets"
              data-active={focussedScoreCategory === key ? "true" : "false"}
            >
              <th />
              <td colspan={$state.playerCount}>
                {#each activePlayers as player}
                  <div
                    data-active={`${
                      focussedPlayer === player && focussedScoreCategory === key
                        ? "true"
                        : "false"
                    }`}
                  >
                    {#each presets as preset, k}
                      <span class="preset">
                        <input
                          type="radio"
                          value={preset}
                          bind:group={$state.players[player][key]}
                          on:click={() => {
                            setTimeout(() => {
                              handleFocusNext(player, key);
                            }, 100);
                          }}
                          name={`${key}_${player}`}
                          id={`${key}_${player}_${k}`}
                        />
                        <label for={`${key}_${player}_${k}`}>{preset}</label>
                      </span>
                    {/each}
                  </div>
                {/each}
              </td>
            </tr>
          {/if}
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
{/if}
