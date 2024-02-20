<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import Graph from './graph.svelte';

  let data = [];
  let tempData = [];

  onMount(async () => {
    const res = await fetch('gdp_co2_death.csv');
    const text = await res.text();
    tempData = d3.csvParse(text, d3.autoType);

    // Parse date fields and create a new 'date' variable
    data = tempData.map(d => ({
      ...d,
      date: new Date(d.Year, 0) // Assuming 'Year' is the name of your date field
    }));
  });
</script>

<main>
  <h1>Global gdp_co2_death Trends</h1>
  <Graph {data} />
</main>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Nunito:wght@300;400;700&display=swap');

  :root {
    --color-bg: #ffffff;
    --color-outline: #c2c2c2;
    --color-shadow: hsl(0, 0%, 0%, 0.1);
    --color-text: #3f4252;
    --color-bg-1: hsla(0, 0%, 0%, 0.2);
    --color-shadow-1: hsl(0, 0%, 96%);
  }

  *,
  *::before,
  *::after {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  main {
    text-align: center;
    font-family: 'Nunito', sans-serif;
    font-weight: 300;
    line-height: 2;
    font-size: 24px;
    color: var(--color-text);
  }

  h1 {
    font-size: 2em;
    font-weight: 300;
    line-height: 2;
  }
</style>