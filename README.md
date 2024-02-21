# Interactive Visualization

## The graph
<iframe src="https://wys6299.github.io/Interactive_Visualization_Project" width="100%" height="500px"></iframe>



## Design Rationale:

In designing our visualization, we aimed to represent the relationship between GDP, CO2 emissions, and death rates due to indoor air pollution across different countries and years. We chose a scatter plot as our primary visualization technique to effectively display this multivariate data. The x-axis represents GDP, the y-axis represents death rates, and the size of the circles represents population size. Additionally, we used color encoding to distinguish between continents, providing another layer of information.

We chose logarithmic scaling for the x-axis to accommodate the wide range of GDP values across countries while maintaining clarity in the visualization. For the y-axis, linear scaling was appropriate as death rates are typically represented in a linear scale. We used square root scaling for circle radius to prevent large population sizes from dominating the visualization.

To enable interactivity, we added a slider component that allows users to select specific years. This allows for dynamic exploration of the data over time, providing insights into trends and patterns.

## Development Process:

We spent significant time importing and processing the data to ensure it was in a suitable format for visualization. This involved parsing the CSV file, converting date fields, and preparing the data for display.
