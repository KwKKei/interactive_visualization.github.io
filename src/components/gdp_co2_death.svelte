<!-- <script lang="ts">
  
  import * as d3 from 'd3';
  import {Scrubber} from "./scrubber.svelte"

  export let data;

  const width = 928;
  const height = 600;
  const marginTop = 20;
  const marginRight = 30;
  const marginBottom = 30;
  const marginLeft = 40;

  $: chart = {
    svg = d3.create("svg")
        .attr("viewBox", [0, 0, width, height]);

    svg.append("g")
        .call(xAxis);

    svg.append("g")
        .call(yAxis);

    svg.append("g")
        .call(grid);

    const circle = svg.append("g")
        .attr("stroke", "black")
      .selectAll("circle")
      .data(dataAt(1800), d => d.name)
      .join("circle")
        .sort((a, b) => d3.descending(a.population, b.population))
        .attr("cx", d => x(d.income))
        .attr("cy", d => y(d.lifeExpectancy))
        .attr("r", d => radius(d.population))
        .attr("fill", d => color(d.region))
        .call(circle => circle.append("title")
          .text(d => [d.name, d.region].join("\n")));

    return Object.assign(svg.node(), {
      update(data) {
        circle.data(data, d => d.name)
            .sort((a, b) => d3.descending(a.population, b.population))
            .attr("cx", d => x(d.income))
            .attr("cy", d => y(d.lifeExpectancy))
            .attr("r", d => radius(d.population));
      }
    });
  }

  $: update = chart.update(currentData)

  $: currentData = dataAt(date)

$: x = d3
    .scaleLog([200, 1e5], [margin.left, width - margin.right]);

  $: y = d3
    .scaleLinear([14, 86], [height - margin.bottom, margin.top]);
  
  $: radius = d3.scaleSqrt([0, 5e8], [0, width / 24]);

  $: max = d3.max(data, (d) => Math.abs(d.value));

  // Create a symmetric diverging color scale.
  $: color = d3
    .scaleOrdinal(data.map(d => d.region), d3.schemeCategory10)
    .unknown("black");

  $: xAxis = g => g
    .attr("transform", `translate(0,${height - margin.bottom})`)
    .call(d3.axisBottom(x).ticks(width / 80, ","))
    .call(g => g.select(".domain").remove())
    .call(g => g.append("text")
        .attr("x", width)
        .attr("y", margin.bottom - 4)
        .attr("fill", "currentColor")
        .attr("text-anchor", "end")
        .text("Income per capita (dollars) →"))
  
  $: yAxis = g => g
    .attr("transform", `translate(${margin.left},0)`)
    .call(d3.axisLeft(y))
    .call(g => g.select(".domain").remove())
    .call(g => g.append("text")
        .attr("x", -margin.left)
        .attr("y", 10)
        .attr("fill", "currentColor")
        .attr("text-anchor", "start")
        .text("↑ Life expectancy (years)"))

  $: grid = g => g
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
</script>

<div class="gdp_co2_death-plot">
  <svg
    bind:this={svg}
    {width}
    {height}
    viewBox="0 0 {width} {height}"
    style="max-width: 100%; height: auto;"
  >
    x-axis
    <g bind:this={gx} transform="translate(0,{height - marginBottom})" />
    y-axis
    <g bind:this={gy} transform="translate({marginLeft},0)">
      <text
        x="5"
        y={marginTop}
        dy="0.32em"
        fill="#000"
        font-weight="bold"
        text-anchor="start"
      >
        Anomaly (°C)
      </text>
    </g>

    points
    <g stroke="#000" stroke-opacity="0.2">
      {#each data as d, i}
        <circle
          key={i}
          cx={x(d.date)}
          cy={y(d.value)}
          fill={color(d.value)}
          r="2.5"
        />
      {/each}
    </g>

    tooltip
    {#if tooltipPt}
      <g transform="translate({x(tooltipPt.date)},{y(tooltipPt.value)})">
        <text font-weight="bold">{tooltipPt.value}</text>
      </g>
    {/if}
  </svg>
</div>

<!--
Sources:
https://observablehq.com/@d3/global-temperature-trends?intent=fork
https://observablehq.com/@d3/line-with-tooltip/2?intent=fork
https://www.nytimes.com/interactive/2017/01/18/science/earth/2016-hottest-year-on-record.html
--> -->
