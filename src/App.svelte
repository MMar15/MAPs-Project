<script>
  import { onMount } from 'svelte';
  import Chart from 'chart.js/auto';

  let chart;

  onMount(async () => {
    const data = await fetch('/data/USA_I_only_F0_8-30-2025.csv');
    const text = await data.text();

    const rows = text.trim().split('\n').map(row => row.split(','));
    const header = rows[0];
    const dataRow = rows.slice(1);
    const labels = dataRow.map(row => row[0]);

    const datasets = header.slice(1).map((name, index) => ({
      label: name,
      data: dataRow.map(row => Number(row[index + 1])),
      borderWidth: 2,
      fill: false
    }));

    const ctx = document.getElementById('myChart').getContext('2d');

    chart = new Chart(ctx, {
      type: 'line',
      data: { labels, datasets },
      options: {}
    });
  });
</script>

<div>
  <canvas id="myChart"></canvas>
</div>