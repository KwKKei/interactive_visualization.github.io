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
  <h3>Discover How Our World's Wealth Affects the Air We Breathe and the Lives We Lead</h3>
  <div style="margin: 0 auto; max-width: 950px;">
    <p style="text-align:left; font-size:23px; line-height:100%">
      Deaths that are from all causes attributed to household air pollution from solid fuels per 100,000 people, in both sexes aged age-standardized
    </p>
  </div>
  <br>
  <Graph {data} />

  <div style="margin: 0 auto; max-width: 1000px;">
    <div style="margin: 0 auto; max-width: auto;">
      <h4 style="text-align:left;">Design Rationale:</h4>
      <p style="text-align:left;">
        In designing our visualization, we aimed to represent the relationship between GDP, CO2 emissions, and death rates due to indoor air pollution across different countries and years. We chose a scatter plot as our primary visualization technique to effectively display this multivariate data. The x-axis represents GDP, the y-axis represents death rates, and the size of the circles represents population size. Additionally, we used color encoding to distinguish between continents, providing another layer of information.
      </p>
      <br>
      <p style="text-align:left;">
        We chose logarithmic scaling for the x-axis to accommodate the wide range of GDP values across countries while maintaining clarity in the visualization. For the y-axis, linear scaling was appropriate as death rates are typically represented in a linear scale. We used square root scaling for circle radius to prevent large population sizes from dominating the visualization.
      </p>
      <br>
      <p style="text-align:left;">
        To enable interactivity, we added a slider component that allows users to select specific years. This allows for dynamic exploration of the data over time, providing insights into trends and patterns.
      </p>
    </div>
    <br>
    <div style="margin: 0 auto; max-width: auto;">
      <h4 style="text-align:left;">Development Process:</h4>
      <p style="text-align:left;">
        We spent significant time importing and processing the data to ensure it was in a suitable format for visualization. This involved parsing the CSV file, converting date fields, and preparing the data for display.
      </p>
      <br>
      <p style="text-align:left;">
        Initially, we encountered an issue where the graph wouldn't load once the app ran. Our investigation led us to a crucial misunderstanding of the `onMount()` function in Svelte. We incorrectly assumed that using `onMount()` for initializing our graph, alongside data loaded asynchronously, would suffice. However, we didn't account for the timing issues related to the asynchronous data loading process. This oversight resulted in the graph attempting to render before the data was fully available, leading to an empty visualization.

        Further complicating the issue, we introduced `afterUpdate()` in an attempt to refresh the graph once the data had updated. Unfortunately, this created a conflict with our initialization logic in `onMount()`. The `afterUpdate()` function was meant to reinitialize the graph with new data, but its execution interfered with the proper initialization process. The graph's data dependency meant that it needed a complete, successful data load before any rendering attempts, something our initial setup with `onMount()` and `afterUpdate()` failed to coordinate effectively.

        The breakthrough came when we decided to remove `afterUpdate()` from our graph initialization logic. This action eliminated the conflict and allowed the `onMount()` function to correctly wait for the asynchronous data to load before attempting to render the graph. By simplifying our approach and ensuring that the graph initialization logic only ran after the data was fully prepared, we resolved the issue. The graph now loads correctly upon the app's startup, showcasing the intended visualization without any timing or data synchronization issues.
      </p>
      <br>
      <p style="text-align:left;">
        To resolve this issue, we restructured our code to properly handle the asynchronous loading of data. Instead of initializing the graph in the onMount() function, we moved the graph initialization logic to a separate function that is called once the data is successfully loaded. This ensured that the graph is only initialized when the data is available, preventing the issue of the graph not loading on app startup.
      </p>
    </div>
  </div>
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
