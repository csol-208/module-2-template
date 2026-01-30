# VS Code + AI Assistant Instructions for CSOL-208

This file provides context to AI coding assistants (GitHub Copilot, Continue, Antigravity) about project setup and coding conventions.

## Environment Setup

**Goal**: A Python environment (Conda or venv) with `requirements.txt` installed, active in VS Code.

### 1. Check Git
Run `git --version` in terminal.
- If missing on **Windows**: Install [Git for Windows](https://gitforwindows.org/).
- If missing on **Mac**: Run `xcode-select --install`.

### 2. Create Python Environment

**Option A: Conda (Preferred)**
If user has Miniconda/Anaconda:
1. `conda create -n csol208 python=3.11 -y`
2. `conda activate csol208`
3. `pip install -r requirements.txt`

**Option B: venv (No Conda)**
1. `python3 -m venv .venv`
2. **Windows**: `.venv\Scripts\activate` | **Mac/Linux**: `source .venv/bin/activate`
3. `pip install -r requirements.txt`

### 3. VS Code Activation
1. Open any `.py` or `.ipynb` file.
2. Click the interpreter version in the bottom-right status bar (or `Ctrl+Shift+P` -> "Python: Select Interpreter").
3. Select `csol208` (Conda) or `.venv` (Virtual Env).
4. Restart terminal to ensure it activates automatically.


## Coding Conventions

### Data Operations: Use Ibis
We use **ibis** for data manipulation. Ibis provides a pandas-like API that compiles to DuckDB, enabling work with larger-than-RAM datasets.

```python
import ibis
from ibis import _
ibis.options.interactive = True

# Connect to DuckDB (in-memory or file)
con = ibis.duckdb.connect()

# Load data
# For spatial data: use DuckDB's ST_Read or specific extensions if needed
# Standard read:
df = ibis.read_csv("data.csv")
```

### Geospatial with Ibis/DuckDB:
Use the `ibis.geo` namespace or raw SQL if Ibis coverage is incomplete for a specific function.
Ensure `spatial` extension is loaded in DuckDB.
```python
# Create point from x, y
# NOTE: Use .cast('float') on columns that might be strings (inferred by CSV reader)
# NOTE: Use column methods directly, e.g., x.point(y) in newer Ibis, rather than ibis.geo.point(x,y)
points = df.mutate(geom = df.x.point(df.y.cast('float')))

# Spatial Join (Points in Polygons)
# For join keys, ensure geometries are comparable.
joined = points.join(polygons, points.geom.within(polygons.geom))
```

**Common Pitfalls & Fixes:**
*   **Lazy Evaluation**: Ibis uses lazy evaluation - operations build a query plan without executing. Use `.execute()` to materialize results as a Python value or pandas DataFrame. This applies to all operations including `count()`, aggregations, and table operations.
*   **Parquet/Spatial Schema**: Loading Parquet files with geometry via `ibis.read_parquet` may result in binary/blob columns. To ensure DuckDB's spatial extension correctly infers `geometry` type:

    ```python
    # Use con.sql with read_parquet
    con = ibis.duckdb.connect()
    table = con.sql("SELECT * FROM read_parquet('data.parquet')")
    ```

*   **Sorting**: Use `.order_by()` instead of the deprecated `.sort_by()`.

### Visualization: AnyMap & Altair
*   **Vector Data (Maps)**: Use **anymap** with maplibre backend for interactive maps of points/polygons.
*   **Statistical Plots**: Use **altair** for charts (bar, scatter, line).

**AnyMap Example:**
```python
from anymap import Map
import geopandas as gpd

# Create a map with maplibre backend
m = Map(center=[-98, 39], zoom=4, height="600px")

# Add vector data (points or polygons) from GeoDataFrame or GeoJSON
# Convert ibis table to GeoDataFrame first
points_gdf = points_ibis_table.to_pandas()
points_gdf = gpd.GeoDataFrame(points_gdf, geometry='geometry', crs='EPSG:4326')

# Add as a layer with styling
m.add_vector(
    points_gdf,
    layer_type="circle",
    paint={
        "circle-radius": 8,
        "circle-color": "#ff0000"
    },
    name="points"
)

# For polygons, similarly use add_vector
m.add_vector(
    polygons_gdf,
    layer_type="fill",
    paint={
        "fill-color": "#0080ff",
        "fill-opacity": 0.5
    },
    name="polygons"
)

m
```

**Altair Example:**
```python
import altair as alt
alt.Chart(df).mark_bar().encode(x='category', y='count')
```

### Raster Analysis: Rasterio/Rioxarray
For raster data (GeoTIFFs), use **rioxarray** (built on rasterio + xarray).

```python
import rioxarray
rds = rioxarray.open_rasterio("file.tif")
rds.plot()
```

---

## Dependencies

Core packages for Module 2:
- `ibis-framework[duckdb,geospatial]` — Spatial SQL via Ibis
- `anymap` — Interactive maps with maplibre backend
- `geopandas` — Vector data I/O
- `rioxarray` — Raster data handling
- `pooch` — Data downloading

---

## Course Context

This is **Module 2: Spatial Data & Environmental Justice** of Climate Solutions 208.

**Sessions:**
1. **Mapping Data Centers**: Coordinates, CRS, Point Maps.
2. **Environmental Justice**: Spatial Joins, Census Data, Vector Analysis.
3. **Biodiversity**: Raster data, intersection analysis.

**Pedagogy**:
Focus on *spatial thinking*: projections, spatial relationships (containment, distance), and the social implications of where infrastructure is placed.
