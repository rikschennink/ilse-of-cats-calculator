<script>
  import { browser } from "$app/environment";
  import { writable } from "svelte/store";
  import "./styles.css";

  const AWARDS = ["🥇", "🥈", "🥉"];

  const ALL_GAME_MODES = ["standard", "family", "events", "beasts"];

  const GAME_MODES = [
    ["standard", "Standard game"],
    ["family", "Family game"],
    ["beasts", "Beasts game"],
    ["events", "Events game"],
    ["beasts,events", "Beasts & Events game"],
  ];

  /** @type { [string,string][] } */
  const AVATARS = [
    "Blue Hissnipper", // blue
    "Crystal Garmin", // green
    "Orange Mhoxxite", // orange
    "Teruvian", // red
    "Starry Vandermil", // purple
    "Oshax", // white
  ].map((cat) => [cat.toLowerCase().replaceAll(" ", "-"), cat]);

  const PLAYERS = ["P1", "P2", "P3", "P4", "P5", "P6"];

  const FAMILY_PRESETS = [0, 8, 11, 15, 20, 25, 30, 35, 40, 45, 50, 60, 65, 70];

  /**
   *
   * @param {string} value
   * @returns {number}
   */
  const calcLessonTotal = (value) => {
    if (!value) return 0;

    const lines = value
      .split(/,| |\n/g)
      .map((line) => line.trim())
      .map((line) => parseInt(line, 10))
      .filter((line) => !isNaN(line));

    return lines.reduce((prev, curr) => {
      return prev + curr;
    }, 0);
  };

  /**
   * @type { { key:string, label:string, mode: string[], min:number|null, max:number|null, step: number, presets?:number[], type?: string }[]}
   */
  const SCORE_CATEGORIES = [
    {
      key: "blue-families",
      label: "Blue",
      mode: ALL_GAME_MODES,
      min: 0,
      max: null,
      step: 1,
      presets: FAMILY_PRESETS,
    },
    {
      key: "green-families",
      label: "Green",
      mode: ALL_GAME_MODES,
      min: 0,
      max: null,
      step: 1,
      presets: FAMILY_PRESETS,
    },
    {
      key: "orange-families",
      label: "Orange",
      mode: ALL_GAME_MODES,
      min: 0,
      max: null,
      step: 1,
      presets: FAMILY_PRESETS,
    },
    {
      key: "purple-families",
      label: "Purple",
      mode: ALL_GAME_MODES,
      min: 0,
      max: null,
      step: 1,
      presets: FAMILY_PRESETS,
    },
    {
      key: "red-families",
      label: "Red",
      min: 0,
      max: null,
      step: 1,
      mode: ALL_GAME_MODES,
      presets: FAMILY_PRESETS,
    },
    {
      key: "rare-treasures",
      label: "Rare",
      min: 0,
      max: null,
      step: 3,
      mode: ALL_GAME_MODES.filter((mode) => mode !== "family"),
      presets: [0, 3, 6, 9, 12, 15],
    },
    {
      key: "private-lessons",
      label: "Lessons",
      min: 0,
      max: null,
      step: 1,
      mode: ALL_GAME_MODES,
      type: "textarea",
    },
    {
      key: "public-lessons",
      label: "Public",
      min: null,
      max: null,
      step: 1,
      mode: ALL_GAME_MODES.filter((mode) => mode !== "family"),
      type: "textarea",
    },
    {
      key: "events",
      label: "Events",
      min: 0,
      max: null,
      step: 1,
      mode: ["events"],
    },
    {
      key: "beasts",
      label: "Beasts",
      mode: ["beasts"],
      min: 0,
      max: null,
      step: 5,
      presets: [0, 5, 10, 15, 20, 25],
    },
    {
      key: "rats",
      min: -35,
      max: 0,
      step: 1,
      label: "Rats",
      mode: ALL_GAME_MODES,
      presets: [0, -1, -2, -3, -4, -5, -6, -7, -8, -9, -10],
    },
    {
      key: "rooms",
      min: -35,
      step: 5,
      max: 0,
      label: "Rooms",
      mode: ALL_GAME_MODES,
      presets: [0, -5, -10, -15, -20, -25, -30, -35],
    },
  ];

  const reset = () => {
    const playerCount = $state ? $state.playerCount : 4;
    const gameMode = $state ? $state.gameMode : "standard";
    $state = {
      gameMode,
      playerCount,
      players: PLAYERS.reduce((obj, key) => {
        // empty player score object
        obj[key] = {
          avatar: null,
          scores: {},
        };
        return obj;
      }, /** @type { { [key:string]: any } }*/ ({})),
    };

    // reset player and UI focus
    activePlayer = "P1";
    activeCategory = "avatar";
  };

  /**
   *
   * @param {string} str
   */
  const getLineCount = (str) => Math.max(1, (str || "").split("\n").length);

  const hasScores = () => {
    return Object.values($state.players).some(
      (player) => Object.keys(player.scores).length
    );
  };

  // changing players resets the game
  /** @type {number} */
  let prevPlayerCount;
  $: activePlayerCount = $state && $state.playerCount;
  $: if (prevPlayerCount !== activePlayerCount) {
    if (prevPlayerCount) reset();
    prevPlayerCount = activePlayerCount;
  }

  // changing game mode resets the game
  /** @type {string} */
  let prevGameMode;
  $: activeGameMode = $state && $state.gameMode;
  $: if (prevGameMode !== activeGameMode && hasScores()) {
    if (prevGameMode) reset();
    prevGameMode = activeGameMode;
  }

  $: availableAvatars =
    $state &&
    AVATARS.filter(
      ([key]) =>
        !Object.values($state.players).some((player) => player.avatar === key)
    ).map(([key]) => key);

  // read current game state
  const state = writable();
  let activePlayer = "";
  let activeCategory = "";

  /**
   *
   * @param { any } state
   */
  const isValidState = (state) => {
    return (
      typeof state.playerCount === "number" &&
      typeof state.gameMode === "string" &&
      typeof state.players === "object" &&
      Object.values(state.players).length === PLAYERS.length &&
      Object.values(state.players).every(
        (value) => typeof value.scores === "object"
      )
    );
  };
  if (browser) {
    try {
      const stateStr = localStorage.getItem("STATE");
      if (stateStr) {
        const storedState = JSON.parse(stateStr);
        if (isValidState(storedState)) {
          $state = storedState;
        } else {
          reset();
        }
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

  $: scores = (activePlayers || []).map((player) => {
    /**
     * @type { { [key:string]:number } }
     */
    const playerScores = $state.players[player].scores;

    let playerScore = 0;
    Object.entries(playerScores).forEach(([key, value]) => {
      const scoreCategorie = SCORE_CATEGORIES.find(
        (category) => category.key === key
      );
      if (scoreCategorie && scoreCategorie.type === "textarea") {
        playerScore += calcLessonTotal(`${value}`);
      } else {
        playerScore += value;
      }
    });

    return [player, playerScore];
  });

  $: ranking = [...scores].sort((a, b) => {
    if (a[1] < b[1]) return 1;
    if (a[1] > b[1]) return -1;
    return 0;
  });

  // reset the scoreboard
  const handleReset = () => {
    if (!confirm("Are you sure you want to reset the scores?")) return;
    reset();
  };

  /**
   * @param {string} player
   * @param {string} scoreCategory
   */
  const handleFocus = (player, scoreCategory) => {
    activePlayer = player;
    activeCategory = scoreCategory;
  };

  /**
   *
   * @param { string } player
   * @returns { number }
   */
  const getNextPlayerIndex = (player) => {
    const currentPlayerIndex = activePlayers.indexOf(player);
    return currentPlayerIndex + 1 === activePlayers.length
      ? 0
      : currentPlayerIndex + 1;
  };

  /**
   * @param {string} player
   * @param {string} scoreCategory
   */
  const handleFocusNext = (player, scoreCategory) => {
    const nextPlayerIndex = getNextPlayerIndex(player);

    if (nextPlayerIndex === 0) {
      const currentScoreCategoryIndex = activeScoreCategories.findIndex(
        (category) => category.key === scoreCategory
      );
      const nextScoreCategoryIndex =
        currentScoreCategoryIndex + 1 === activeScoreCategories.length
          ? -1
          : currentScoreCategoryIndex + 1;

      if (nextScoreCategoryIndex === -1) {
        // stop!
        activeCategory = "";
        activePlayer = "";
        return;
      }

      activeCategory = activeScoreCategories[nextScoreCategoryIndex].key;
    }

    activePlayer = activePlayers[nextPlayerIndex];

    // focus the field
    const fieldId = `${activePlayer}-${activeCategory}`;
    document.getElementById(fieldId)?.focus();
  };
</script>

<svelte:head>
  <title>Meow Meter</title>
  <meta
    name="description"
    content="Meow Meter, an Isle of Cats Score Calculator"
  />
  <meta name="theme-color" content="#F7ECDC" />
  <meta name="mobile-web-app-capable" content="yes" />
  <meta name="apple-touch-startup-image" content="splash.png" />
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

    <div class="table-wrapper">
      <table>
        {#if $state.playerCount > 1}
          <thead>
            <tr class="presets avatars">
              <td colspan={$state.playerCount + 1}>
                {#each activePlayers as player}
                  <div
                    data-active={`${
                      activePlayer === player && activeCategory === "avatar"
                        ? "true"
                        : "false"
                    }`}
                  >
                    {#each AVATARS as [key, label]}
                      <span class="preset avatar">
                        <input
                          type="radio"
                          value={key}
                          disabled={!availableAvatars.includes(key) &&
                            key !== $state.players[player].avatar}
                          bind:group={$state.players[player].avatar}
                          on:click={() => {
                            const nextPlayerIndex = getNextPlayerIndex(player);

                            // done! move to score categories
                            if (nextPlayerIndex === 0) {
                              activeCategory = "";
                              activePlayer = "P1";
                              return;
                            }

                            // select next player
                            activePlayer = activePlayers[nextPlayerIndex];
                          }}
                          name={`avatar_${player}`}
                          id={`avatar_${player}_${key}`}
                        />
                        <label for={`avatar_${player}_${key}`}
                          ><img
                            src={`/avatar/${key}.png`}
                            alt={label}
                            draggable="false"
                          /></label
                        >
                      </span>
                    {/each}
                  </div>
                {/each}
              </td>
            </tr>
            <tr data-avatar-input={activeCategory === "avatar"}>
              <th />
              {#each activePlayers as player, i}
                <th data-active={activePlayer === player ? "true" : "false"}>
                  <input
                    type="radio"
                    bind:group={activePlayer}
                    on:click={() => {
                      // hide
                      if (
                        activePlayer === player &&
                        activeCategory === "avatar"
                      ) {
                        activeCategory = "";
                      }
                      // show
                      else {
                        activeCategory = "avatar";
                        activePlayer = player;
                      }
                    }}
                    value={player}
                    id={player}
                    name="avatars"
                  />
                  <label for={player}>
                    {#if $state.players[player].avatar}
                      <img
                        src={`/avatar/${$state.players[player].avatar}.png`}
                        alt={player}
                        draggable="false"
                      />
                    {:else}
                      <img
                        src={`/avatar/${AVATARS[i][0]}.png`}
                        alt={AVATARS[i][1]}
                        class="avatar-placeholder"
                        draggable="false"
                      />
                    {/if}
                  </label>
                </th>
              {/each}
            </tr>
          </thead>
        {/if}
        <tbody>
          {#each activeScoreCategories as { key, label, min, max, step, presets, type } (key)}
            <tr
              class={`${key} scores`}
              data-active={activeCategory === key ? "true" : "false"}
            >
              <th>{label}</th>
              {#each activePlayers as player}
                <td data-active={activePlayer === player ? "true" : "false"}>
                  {#if type === "textarea"}
                    <textarea
                      placeholder={`0\n0`}
                      id={`${player}-${key}`}
                      style={`--lines: ${
                        getLineCount($state.players[player].scores[key]) + 1
                      }`}
                      on:keypress={(e) => {
                        if (!/[0-9]|Enter|Backspace/.test(e.key))
                          e.preventDefault();
                      }}
                      on:focus={() => handleFocus(player, key)}
                      bind:value={$state.players[player].scores[key]}
                    />
                  {:else}
                    <input
                      type="number"
                      {min}
                      {max}
                      {step}
                      placeholder="0"
                      id={`${player}-${key}`}
                      on:focus={() => handleFocus(player, key)}
                      bind:value={$state.players[player].scores[key]}
                    />
                  {/if}
                </td>
              {/each}
            </tr>
            {#if presets}
              <tr
                class={`${key} presets`}
                data-active={activeCategory === key ? "true" : "false"}
              >
                <td colspan={$state.playerCount + 1}>
                  {#each activePlayers as player}
                    <div
                      data-active={`${
                        activePlayer === player && activeCategory === key
                          ? "true"
                          : "false"
                      }`}
                    >
                      {#each presets as preset, k}
                        <span class="preset score">
                          <input
                            type="radio"
                            value={preset}
                            bind:group={$state.players[player].scores[key]}
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
            <th />
            {#each activePlayers as player, i}
              <td data-active={activePlayer === player ? "true" : "false"}>
                {scores[i][1]}
                {#if scores.some(([, score]) => score !== 0)}
                  <span class="award"
                    >{AWARDS[ranking.findIndex(([p]) => p === player)] ||
                      ""}</span
                  >
                {/if}
              </td>
            {/each}
          </tr>
        </tfoot>
      </table>
    </div>
  </div>
{/if}
