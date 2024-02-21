<!-- Your main component -->
<script lang="ts">
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import Slider from './Slider.svelte';
  import CheckboxFilter from './CheckboxFilter.svelte';

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
  let gx;
  let gy;

  const margin = { top: 20, right: 40, bottom: 65, left: 40 };
  const width = 960;
  const height = 560;

  let selectedContinents = [];

  const dataAt = year => data.filter(d => d.Year === year);

  const bisect = d3.bisector((d) => d.dataAt).center;
  let tooltipPt = null;
  function onPointerMove(event) {
    const i = bisect(data, x.invert(d3.pointer(event)[0]));
    tooltipPt = data[i];
    console.log(tooltipPt);
  }

  function onPointerLeave(event) {
    tooltipPt = null;
  }

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
          .attr("transform", `translate(0,${height - margin.bottom + 15})`)
          .call(d3.axisBottom(x).ticks(width / 80, ","))
          .call(g => g.select(".domain").remove())
          .call(g => g.append("text")
              .attr("x", width)
              .attr("y", margin.bottom - 25)
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
      d3.select(svg)
        .on('pointerenter pointermove', onPointerMove)
        .on('pointerleave', onPointerLeave);

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

<div>
<CheckboxFilter continents={Array.from(new Set(data.map(d => d.Continent)))} bind:selectedContinents />
</div>

<div><svg id='chart'
  bind:this={svg}
  {width}
  {height}
  viewBox="0 0 {width} {height}"
  style="max-width: 400%; height: auto;"/></div>

  <!-- {#if tooltipPt}
    <g transform="translate({x(tooltipPt.date)},{y(tooltipPt.value)})">
      <text font-weight="bold">{tooltipPt.value}</text>
    </g>
  {/if}
   -->
<Slider bind:selectedYear />


<style>
/* Add your styles here */
</style>