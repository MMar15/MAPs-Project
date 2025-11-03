<script>
  import { onMount } from 'svelte';
  import Chart from 'chart.js/auto';

  let chart;
  let headers = [];
  let datasets = [];
  let selectedData = '';

  onMount(async () => {
    const data = await fetch('/data/USA_I_only_F0_8-30-2025.csv');
    const text = await data.text();

    const rows = text.trim().split('\n').map(row => row.split(','));
    headers = rows[0];
    const dataRow = rows.slice(1);
    const labels = dataRow.map(row => row[0]);

    datasets = headers.slice(1).map((name, index) => ({
      label: name,
      data: dataRow.map(row => Number(row[index + 1])),
      borderWidth: 2,
      fill: true
    }));

    selectedData = headers[1];

    const ctx = document.getElementById('myChart').getContext('2d');

    chart = new Chart(ctx, {
      type: 'line',
      data: { labels, datasets },
      options: {}
    });
    refresh();
  });

  function refresh() {
    const dataset = datasets.find(d => d.label === selectedData);
    chart.data.datasets = [dataset];
    chart.update();
  };
</script>

<select bind:value={selectedData} on:change={refresh}>
  {#each headers.slice(1) as column}
    <option value={column}>{column}</option>
  {/each}
</select>

<div>
  <canvas id="myChart"></canvas>
</div>