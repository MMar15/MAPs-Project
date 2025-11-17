<script>
  import { onMount } from 'svelte';
  import Chart from 'chart.js/auto';
  import converter from "us-state-converter";

  let chart;
  let headers = [];
  let datasets = [];
  let labels = [];
  let USHeaders = [];
  let USDatasets = [];
  let USLabels = [];
  let selectedData = 'View All';
  let selectedArea = 'America';
  let stateSets = {};

  async function getData(location) {
    const data = await fetch(location);
    const text = await data.text();
    const rows = text.trim().split('\n').map(row => row.split(','));
    const headers = rows[0];
    const dataRows = rows.slice(1);
    const labels = dataRows.map(row => row[0]);

    const datasets = headers.slice(1).map((name, i) => ({
      label: name,
      data: dataRows.map(row => Number(row[i + 1])),
      borderWidth: 2,
      fill: true,
    }));
    return { headers, labels, datasets };
  };

  onMount(async () => {
    const USA = await getData('/data/USA_I_only_F0_8-30-2025.csv');
    USHeaders = USA.headers;
    USDatasets = USA.datasets;
    USLabels = USA.labels;
    const states = await getData('/data/state_F0_8-30-2025.csv');
    headers = states.headers;
    datasets = states.datasets;
    labels = states.labels;
    stateSets = splitStates(states.headers);

    const ctx = document.getElementById('myChart').getContext('2d');

    chart = new Chart(ctx, {
      type: 'line',
      data: { labels: USLabels, datasets: USDatasets },
      options: { 
        responsive: true,
        maintainAspectRatio: false,      
      }
    });
  });

  async function refresh() {
    if (!chart) return;
    let currentDatasets = [];
    let currentLabels = [];

    if (selectedArea === 'America') {
      currentDatasets = USDatasets;
      currentLabels = USLabels;
    } else if (stateSets[selectedArea]) {
      const columns = stateSets[selectedArea];
      currentDatasets = datasets.filter(d => columns.includes(d.label));
      currentLabels = labels;
    }

    chart.data.labels = currentLabels;
    if (selectedData === 'View All') {
      chart.data.datasets = currentDatasets;
    } else {
      chart.data.datasets = currentDatasets.filter(d => d.label === selectedData);
    }
    chart.update();
  };

  function splitStates(headers) {
    const stateHeaders = headers.slice(1);
    const states = {};
    
    stateHeaders.forEach(name => {
      if (name === "ID ") return;
      const dataset = datasets.find(d => d.label === name);
      if (!dataset) return;
      const zero = dataset.data.every(z => z === 0);
      if (zero) return;

      const acro = name.slice(0, 2).toUpperCase();
      const unacro = converter(acro);
      const fullName = unacro ? unacro.name : acro;
      
      if (!states[fullName]) {
        states[fullName] = [];
      }
      states[fullName].push(name);
    });
    
    console.log("States grouped: ", states);
    return states;
  }
</script>

<select bind:value={selectedArea} on:change={() => { selectedData = 'View All'; refresh(); }}>
  <option value="America">America</option>
  {#each Object.keys(stateSets) as state}
    <option value={state}>{state}</option>
  {/each}
</select>

<select bind:value={selectedData} on:change={refresh}>
  <option value="View All">View All</option>
  {#if selectedArea === 'America'}
    {#each USHeaders.slice(1) as column}
      <option value={column}>{column}</option>
    {/each}
  {:else if stateSets[selectedArea]}
    {#each stateSets[selectedArea] as column}
      <option value={column}>{column}</option>
    {/each}
  {/if}
</select>

<div style="width: 100%; height: 100%;">
  <canvas id="myChart"></canvas>
</div>