# Chapel Hill Arrest Data Visualizer

### An Interactive Police Arrest Map & Hotspot Visualizer in Python
**Tech Stack:** Python 3 | Pandas | Folium (Leaflet.js) | Google Sheets (CSV) | Jupyter / Google Colab

---

## Overview

**Chapel Hill Arrest Data Visualizer** is an interactive map tool built in Python using Pandas and Folium. The application pulls public arrest records from the Town of Chapel Hill, cleans and standardizes the dataset, and visualizes arrest patterns across the town through clickable map markers and density heatmaps.

Built as a foundational first Python project, this tool was created to explore core data analysis workflows: reading online spreadsheet data via CSV URLs, cleaning real-world inconsistencies, fixing coordinate anomalies, and rendering geospatial data on interactive web maps.

---

## Key Features

- **Online Spreadsheet Integration**: Fetches arrest data directly from a public Google Sheets CSV export link into a Pandas DataFrame.
- **Data Cleaning & Standardization**: Cleans missing values, converts text `<Null>` entries into true nulls, and translates abbreviated police codes into clear labels.
- **Coordinate Bug Fix**: Automatically detects and fixes a data entry issue in the municipal database where Latitude and Longitude were swapped for recent arrests (which mistakenly placed points in Antarctica).
- **Recent Arrest Filtering (2021–Present)**: Filters the 40,000+ historical records down to ~4,800 recent arrests, keeping the analysis focused on recent trends and preventing browser lag.
- **Comparative Map Markers**: Highlights a selected demographic or offense category in one color against all remaining arrests in another color.
- **Age-Scaled Circles**: Scales marker circle sizes based on the individual's age (younger individuals display smaller circles; older individuals display larger ones).
- **Interactive Tooltips**: Hovering over any marker displays the selected category, the primary criminal charge, and the person's age.
- **Hotspot Heatmap**: Generates a glowing density heatmap for the selected group, highlighting high-concentration arrest zones and corridors.
- **On-Map Layer Controls**: An integrated toggle control allows users to switch individual marker circles and the heatmap layer on or off.

---

## Data Cleaning Pipeline

Real-world public data often contains formatting errors and missing values. The script processes the raw data through four main steps before mapping:

1. **Missing Data Handling**: Reads the dataset with `low_memory=False` and flags `<Null>` strings as actual `NaN` values so incomplete rows are dropped cleanly.
2. **Coordinate Correction**:
   - **Swapped Lat/Long**: For entries where `latitude < 0` and `longitude > 0`, the script swaps the coordinates back so they map to North Carolina instead of Antarctica.
   - **Invalid (0, 0) Points**: Filters coordinates to the North Carolina boundary box, removing placeholder `(0.0, 0.0)` records located off the coast of Africa.
3. **Date Filtering**: Converts the `Date_of_Arrest` column to datetime objects and filters for records after 2020 (`year > 2020`), reducing the dataset from ~40,000 rows to ~4,800 rows.
4. **Category Mapping**: Translates shorthand codes into clear readable text:
   - **Gender**: `M` $\rightarrow$ *Male*, `F` $\rightarrow$ *Female*
   - **Drugs/Alcohol**: `Y` $\rightarrow$ *Yes*, `N` $\rightarrow$ *No*, `U` $\rightarrow$ *Unknown*
   - **Ethnicity**: `H` $\rightarrow$ *Hispanic or Latino*, `N` $\rightarrow$ *Not Hispanic or Latino*
   - **Race**: `B` $\rightarrow$ *Black*, `W` $\rightarrow$ *White*, `A` $\rightarrow$ *Asian*, `I` $\rightarrow$ *American Indian*

---

## Map Components (`arrest_map`)

The map function combines several Leaflet and Folium layers into a single view:

| Component | Tool / Class | Description |
| :--- | :--- | :--- |
| **Base Map** | `folium.Map` | Centers the view over the mean coordinates of Chapel Hill at zoom level 12. |
| **Tiles** | `folium.TileLayer` | Uses OpenStreetMap with a custom referrer policy to prevent HTTP 403 Forbidden errors. |
| **Markers** | `folium.Circle` | Plots each arrest with custom colors, age-based radius sizing, and HTML tooltips. |
| **Heatmap** | `folium.plugins.HeatMap` | Extracts the latitude and longitude of the target group to draw a density heatmap. |
| **Layer Control** | `folium.LayerControl` | Positioned at `bottomleft` so notebook controls do not cover the toggle checkboxes. |

---

## Project Structure

```text
chapel-hill-arrest-visualizer/
|-- arrest_analysis.ipynb                # Main Jupyter / Google Colab notebook
|-- README.md                            # Project documentation
`-- requirements.txt                     # Dependencies (pandas, folium)
```

---

## Getting Started

### Prerequisites

- **Python**: Version 3.8 or higher.
- **Internet Connection**: Required to fetch the Google Sheet data and load OpenStreetMap tiles.

### Installation

Install the required packages using pip:

```bash
pip install pandas folium
```

*(If running inside **Google Colab**, no installation is necessary as Pandas and Folium are pre-installed).*

### Running the Project

Open and run `arrest_analysis.ipynb` in your preferred notebook environment:
- **Google Colab**: Upload the notebook and select **Runtime > Run all**.
- **Jupyter Notebook**: Run `jupyter notebook` in your terminal and open the file.
- **VS Code**: Open the notebook and click **Run All**.

---

## Usage & Example Queries

The `arrest_map` function takes four parameters:

```python
arrest_map(column_name, target_value, target_color, baseline_color)
```

### Example Visualizations

| Analysis Focus | Column | Target Value | Function Call |
| :--- | :--- | :--- | :--- |
| **Racial Disparity** | `Race` | `'Black'` | `arrest_map('Race', 'Black', 'cyan', 'black')` |
| **Substance Involvement** | `Drugs_or_Alcohol_Present` | `'Yes'` | `arrest_map('Drugs_or_Alcohol_Present', 'Yes', 'red', 'green')` |
| **Gender Distribution** | `Gender` | `'Female'` | `arrest_map('Gender', 'Female', 'purple', 'gray')` |
| **Arrest Type** | `Type_of_Arrest` | `'ON VIEW'` | `arrest_map('Type_of_Arrest', 'ON VIEW', 'orange', 'navy')` |

---

## Limitations & Lessons Learned

As an early Python project, several design limitations were valuable learning opportunities:

1. **Browser Performance with Many Markers**:
   - `folium.Circle` creates an individual SVG element on the webpage for each arrest record.
   - While rendering ~4,800 points works well on modern computers, plotting all 40,000+ historical records caused the browser to freeze. Filtering to post-2020 records keeps the map responsive without needing complex clustering.
2. **Single-Category Comparisons**:
   - The visualization only compares one selected value against all others (binary comparison). It does not currently support color-coding all four racial groups or multiple charge types at the same time with a multi-color legend.
3. **Reliance on Global Variables**:
   - The `arrest_map` function accesses the global `df` variable directly rather than taking the dataframe as a parameter, which limits modularity.
4. **Code-Based Querying**:
   - Filters must be changed by editing the function call in Python rather than through interactive UI controls like dropdown menus or date sliders.

---

## Future Enhancements

- [ ] **Marker Clustering**: Implement `folium.plugins.FastMarkerCluster` to allow mapping all 40,000+ historical records without browser performance drops.
- [ ] **Interactive Web UI**: Convert the notebook into a **Streamlit** dashboard with interactive dropdowns, charge category filters, and date range sliders.
- [ ] **Time-of-Day Analysis**: Add charts showing peak arrest hours (e.g., weekend night hours vs. daytime activity).
- [ ] **Multi-Color Legend**: Update the mapping logic to support a full categorical color legend for all demographic groups simultaneously.

---

## Data Source & Attribution

- **Data Source**: Public arrest records provided by the **Town of Chapel Hill / Orange County Open Data Portal**.
- **Usage**: Intended for educational, portfolio, and analytical exploration.
