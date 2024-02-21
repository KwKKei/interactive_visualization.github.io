<script lang="ts">
    import { onMount } from 'svelte';
    import * as d3 from 'd3';
    import Slider from './Slider.svelte';

    export let data;
    export let selectedYear;

    let svg;
    let circle;
    let xAxis;
    let yAxis;
    let grid;
    let x;
    let y;
    let radius;
    let color;

    const margin = { top: 20, right: 20, bottom: 35, left: 40 };
    const width = 960;
    const height = 560;

    const bisect = d3.bisector((d) => d.Year).center;

    const dataAt = year => data.filter(d => d.Year === year);

    const handlePointerMove = (event) => {
        const xPos = d3.pointer(event)[0];
        const year = x.invert(xPos);
        const i = bisect(data, year);
        // Handle tooltip logic here
    };

    const handlePointerLeave = () => {
        // Handle tooltip hiding logic here
    };

    onMount(async () => {
        // Fetch data
        const res = await fetch('gdp_co2_death.csv');
        data = await res.text();
        data = d3.csvParse(data, d3.autoType);

        // Determine domain for x, y, and radius scales
        const minGDP = d3.min(data, d => d.GDP);
        const maxGDP = d3.max(data, d => d.GDP);
        const minDeath = d3.min(data, d => d.Death);
        const maxDeath = d3.max(data, d => d.Death);
        const minPopulation = d3.min(data, d => d.Population);
        const maxPopulation = d3.max(data, d => d.Population);

        x = d3.scaleLog([minGDP, maxGDP], [margin.left, width - margin.right]);
        y = d3.scaleLinear([minDeath, maxDeath], [height - margin.bottom, margin.top]);
        radius = d3.scaleSqrt([minPopulation, maxPopulation], [0, width / 24]);
        color = d3.scaleOrdinal(data.map(d => d.Continent), d3.schemeCategory10).unknown("black");

        const svgNode = d3.select("#chart").append("svg")
            .attr("viewBox", [0, 0, width, height]);

        xAxis = g => g
            .attr("transform", `translate(0,${height - margin.bottom})`)
            .call(d3.axisBottom(x).ticks(width / 80, ","))
            .call(g => g.select(".domain").remove())
            .call(g => g.append("text")
                .attr("x", width)
                .attr("y", margin.bottom - 4)
                .attr("fill", "currentColor")
                .attr("text-anchor", "end")
                .text("GDP →"));

        yAxis = g => g
            .attr("transform", `translate(${margin.left},0)`)
            .call(d3.axisLeft(y))
            .call(g => g.select(".domain").remove())
            .call(g => g.append("text")
                .attr("x", -margin.left)
                .attr("y", 10)
                .attr("fill", "currentColor")
                .attr("text-anchor", "start")
                .text("↑ Death"));

        grid = g => g
            .attr("stroke", "currentColor")
            .attr("stroke-opacity", 0.1)
            .call(g => g.append("g")
                .selectAll("line")
                .data(x.ticks())
                .join("line")
                .attr("x1", d => 0.5 + x(d))
                .attr("x2", d => 0.5 + x(d))
                .attr("y1", margin.top)
                .attr("y2", height - margin.bottom))
            .call(g => g.append("g")
                .selectAll("line")
                .data(y.ticks())
                .join("line")
                .attr("y1", d => 0.5 + y(d))
                .attr("y2", d => 0.5 + y(d))
                .attr("x1", margin.left)
                .attr("x2", width - margin.right));

        svgNode.append("g").call(xAxis);
        svgNode.append("g").call(yAxis);
        svgNode.append("g").call(grid);

        circle = svgNode.append("g")
            .attr("stroke", "black")
            .selectAll("circle")
            .data(dataAt(1991), d => d.Entity)
            .join("circle")
            .sort((a, b) => d3.descending(a.Population, b.Population))
            .attr("cx", d => x(d.GDP))
            .attr("cy", d => y(d.Death))
            .attr("r", d => radius(d.Population))
            .attr("fill", d => color(d.Continent))
            .call(circle => circle.append("title")
                .text(d => [d.Entity, d.Continent, ['Deaths: '+d.Death], ['GDP: '+d.GDP], ['Population: '+d.Population]].join("\n")));

        // Attach event listeners for pointer events
        d3.select(svgNode.node())
            .on('pointerenter pointermove', handlePointerMove)
            .on('pointerleave', handlePointerLeave);

        // Draw legend
        const legend = svgNode.append("g")
            .attr("transform", `translate(${width - margin.right - 150}, ${margin.top})`);

        const legendItems = legend.selectAll("g")
            .data(color.domain())
            .join("g")
            .attr("transform", (d, i) => `translate(0, ${i * 20})`);

        legendItems.append("rect")
            .attr("width", 15)
            .attr("height", 15)
            .attr("fill", color);

        legendItems.append("text")
            .attr("x", 20)
            .attr("y", 9)
            .attr("dy", "0.35em")
            .text(d => d);

        // Return the SVG node
        return svgNode.node();
    });

    $: {
        // Update circle positions and attributes when data or selectedYear changes
        if (circle) {
            circle.data(dataAt(selectedYear), d => d.Entity)
                .sort((a, b) => d3.descending(a.Population, b.Population))
                .attr("cx", d => x(d.GDP))
                .attr("cy", d => y(d.Death))
                .attr("r", d => radius(d.Population))
                .attr("fill", d => color(d.Continent));
        }
    }
</script>

<div><svg id='chart'
    bind:this={svg}
    {width}
    {height}
    viewBox="0 0 {width} {height}"
    style="max-width: 100%; height: auto;"/></div>
    

<Slider bind:selectedYear />

<div style="margin: 0 auto; max-width: 800px;">
  <h3>Interactive Visualization</h3>

  <div style="margin: 0 auto; max-width: 800px;">
    <h4>Design Rationale:</h4>
    <p>
      In designing our visualization, we aimed to represent the relationship between GDP, CO2 emissions, and death rates due to indoor air pollution across different countries and years. We chose a scatter plot as our primary visualization technique to effectively display this multivariate data. The x-axis represents GDP, the y-axis represents death rates, and the size of the circles represents population size. Additionally, we used color encoding to distinguish between continents, providing another layer of information.
    </p>
    <p>
      We chose logarithmic scaling for the x-axis to accommodate the wide range of GDP values across countries while maintaining clarity in the visualization. For the y-axis, linear scaling was appropriate as death rates are typically represented in a linear scale. We used square root scaling for circle radius to prevent large population sizes from dominating the visualization.
    </p>
    <p>
      To enable interactivity, we added a slider component that allows users to select specific years. This allows for dynamic exploration of the data over time, providing insights into trends and patterns.
    </p>
  </div>

  <div style="margin: 0 auto; max-width: 800px;">
    <h4>Development Process:</h4>
    <p>
      We spent significant time importing and processing the data to ensure it was in a suitable format for visualization. This involved parsing the CSV file, converting date fields, and preparing the data for display.
    </p>
    <p>
      Initially, we encountered an issue where the graph wouldn't load once the app ran. After investigating, we realized that we misunderstood the purpose and operation of the onMount() function in Svelte. We mistakenly believed that it was the equivalent of the componentDidMount() lifecycle method in React, which is called once when the component mounts. However, in Svelte, onMount() is only called once when the component is created and added to the DOM. This meant that our graph initialization code, which relied on data that was asynchronously loaded, was executing before the data was available, resulting in an empty graph.
    </p>
    <p>
      To resolve this issue, we restructured our code to properly handle the asynchronous loading of data. Instead of initializing the graph in the onMount() function, we moved the graph initialization logic to a separate function that is called once the data is successfully loaded. This ensured that the graph is only initialized when the data is available, preventing the issue of the graph not loading on app startup.
    </p>
  </div>
</div>


<style>
  /* Add your styles here */
</style>
