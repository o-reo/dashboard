<script>
  export let data = [];
  export let columns = {};
  export let sortkey;
  export let threshold = "";
  export let colorCellFunction = null;

  let displayData = [];
  let sortedData = [];
  let reload = false;
  let search = "";
  let sortKey = sortkey;
  let reverse = true;

  // init function initiates values.
  function init() {
    reload = true;
    sortTable(sortKey);
  }

  // sortTable function sorts the display array by key
  // If second time clicked on the same key, then reverses the array.
  function sortTable(key) {
    if (key === null || data.length === 0) {
      return;
    }
    // If reload (websocket)
    if (reload === true && sortKey) {
      reload = false;
      if (reverse === false) {
        data.sort((a, b) => (a[sortKey] > b[sortKey] ? 1 : -1));
      } else if (reverse === true) {
        data.sort((a, b) => (a[sortKey] > b[sortKey] ? 1 : -1));
        data = data.reverse();
      }
      // If clicked
    } else {
      if (sortKey === key) {
        reverse = !reverse;
        data = data.reverse();
      } else {
        reverse = false;
        data = data.sort((a, b) => (a[key] > b[key] ? 1 : -1));
      }
    }

    sortedData = data;
    sortKey = key;
  }

  // searchTable filters display array if one of element's value
  // contains searchkey (string in string). If searchKey is empty
  // sets display array to default.
  function searchTable(searchKey) {
    if (searchKey === "") {
      displayData = sortedData;
      return;
    }
    displayData = sortedData.filter(elem => {
      for (let key in elem) {
        if (typeof elem[key] !== "string") {
          continue;
        }
        const value = elem[key].toLowerCase();
        if (value.includes(searchKey.toLowerCase())) {
          return true;
        }
      }
      return false;
    });
  }

  function colorCell(xp) {
    return colorCellFunction(xp);
  }

  $: if (data) {
    init();
  }

  $: if (search) {
    searchTable(search);
  }

  $: if (sortedData) {
    searchTable(search);
  }
</script>

<style>
  table {
    border-collapse: collapse;
    width: 100%;
    position: relative;
  }

  .rowlink::before {
    content: "";
    display: block;
    position: absolute;
    left: 0;
    width: 100%;
    height: 1.5em; /* don't forget to set the height! */
  }
</style>

<div class="control" />

<div class="field is-horizontal">
  <div class="field-label is-normal">
    <label class="label">Search</label>
  </div>
  <div class="field-body">
    <div class="field">
      <p class="control">
        <input
          bind:value={search}
          class="input"
          type="text"
          placeholder="search..." />
      </p>
    </div>
  </div>
</div>

<table class="table">
  <thead>
    <tr>
      <th>№</th>
      {#each Object.keys(columns) as key (key)}
        <th
          on:click={() => {
            sortTable(key);
          }}>
          {columns[key]}
        </th>
      {/each}
    </tr>
  </thead>
  <tbody>
    {#each displayData as element, index (element.id)}
      <tr style="background:{colorCell(element[threshold])}">
        <td>
          {#if '_url_' in element}
            <a class="rowlink" href={element._url_}>{index + 1}</a>
          {:else}{index + 1}{/if}
        </td>
        {#each Object.keys(columns) as key (key)}
          <td>{element[key]}</td>
        {/each}
      </tr>
    {:else}
      <tr>
        <td colspan="6">Loading...</td>
      </tr>
    {/each}
  </tbody>
</table>
