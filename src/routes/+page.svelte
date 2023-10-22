<script>
  import "./styles.css";

  /**
   * @type { { name: string, values:number[] }[] }
   */
  let players = [
    {
      name: "A",
      values: [],
    },
    {
      name: "B",
      values: [],
    },
    {
      name: "C",
      values: [],
    },
    {
      name: "D",
      values: [],
    },
    {
      name: "E",
      values: [],
    },
    {
      name: "F",
      values: [],
    },
  ];

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
      label: "# Rats",
      calc: (v) => v * -1,
    },
    {
      key: "rooms",
      label: "# Rooms",
      calc: (v) => v * -5,
    },
  ];

  $: scores = players.map((player) => {
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
</script>

<svelte:head>
  <title>Isle of Cats</title>
  <meta name="description" content="Isle of Cats score app" />
</svelte:head>

<div class="app">
  <table>
    <thead>
      <tr>
        <th />
        {#each players as player}
          <th>{player.name}</th>
        {/each}
      </tr>
    </thead>
    <tbody>
      {#each scoreCategories as scoreCategorie, i (scoreCategorie.key)}
        <tr>
          <th>{scoreCategorie.label}</th>
          {#each players as player, j}
            <td
              ><input
                type="number"
                min="0"
                step="1"
                bind:value={players[j].values[i]}
              /></td
            >
          {/each}
        </tr>
      {/each}
    </tbody>
    <tfoot>
      <tr>
        <th>Score</th>
        {#each scores as score}
          <td>{score}</td>
        {/each}
      </tr>
    </tfoot>
  </table>
</div>
