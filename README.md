# Network Analysis

A Python-based geospatial network analysis workflow for calculating walking routes from a defined hub to multiple destinations using the [OpenRouteService (ORS)](https://openrouteservice.org/) Directions API.

The workflow reads destination coordinates from a CSV file, calculates the shortest walking route from the hub to each destination, extracts route distance, estimated duration, street names and turn-by-turn instructions, and exports the results as CSV and GeoJSON files.

## What the Workflow Does

For each destination in the input dataset, the workflow:

1. Reads the destination name and coordinates from a CSV file.
2. Uses a predefined hub as the route origin.
3. Sends a routing request to OpenRouteService.
4. Calculates a walking route using the shortest-route preference.
5. Extracts route distance and estimated duration.
6. Extracts turn-by-turn navigation instructions.
7. Identifies the streets encountered along the route.
8. Decodes the returned route geometry.
9. Exports the results to the `outputs` directory.

A one-second delay is applied between routing requests.

## Requirements

* Python 3.9 or later
* Jupyter Notebook, JupyterLab, or VS Code with Jupyter support
* Internet connection
* An OpenRouteService API key

The workflow uses:

* `openrouteservice`
* `pandas`
* `geopandas`
* `shapely`
* `requests`
* `python-dotenv`

## OpenRouteService

This project uses OpenRouteService for route calculation.

Before running the notebook, review the official ORS resources for account requirements, API documentation, service information and current usage restrictions:

* **OpenRouteService:** https://openrouteservice.org/
* **API Documentation:** https://docs.openrouteservice.org/
* **API Restrictions:** https://openrouteservice.org/restrictions/
* **Developer Login / API Key:** https://openrouteservice.org/log-in/

OpenRouteService provides routing for several travel profiles, including walking, cycling and driving. This project specifically uses the `foot-walking` profile.

API availability, limits and restrictions may change. Users should consult the official ORS documentation before running large-scale analyses.

## Installation

Clone the repository:

```bash
git clone https://github.com/YOUR_USERNAME/YOUR_REPOSITORY.git
cd YOUR_REPOSITORY
```

Install the required packages:

```bash
pip install openrouteservice pandas geopandas shapely requests python-dotenv
```

## API Key Configuration

The OpenRouteService API key must **not** be written directly into the notebook or committed to GitHub.

Create a `.env` file in the project directory:

```text
API_KEY=your_openrouteservice_api_key
```

The notebook loads the key using `python-dotenv`:

```python
from dotenv import load_dotenv
import os

load_dotenv()

API_KEY = os.getenv("API_KEY")
```

The `.env` file should be listed in `.gitignore` and must never be committed to the repository.

## Input Data

The workflow expects a CSV file named:

```text
Ojota_Hub.csv
```

The file should contain the following columns:

| Column | Description                           |
| ------ | ------------------------------------- |
| `Name` | Name or identifier of the destination |
| `Hub`  | Hub associated with the destination   |
| `Long` | Destination longitude                 |
| `Lat`  | Destination latitude                  |

Example:

```csv
Name,Hub,Long,Lat
Arowolo Filling station,Ojota,3.375609,6.582764
Berger Motor Park,Ojota,3.376671,6.581990
Ojota Snr Sec. School,Ojota,3.377717,6.583223
```

### Coordinate Requirements

Destination coordinates must be provided as decimal degrees.

The workflow uses the coordinate order:

```text
(Longitude, Latitude)
```

For example:

```text
(3.375609, 6.582764)
```

The coordinates are handled as WGS 84 geographic coordinates (EPSG:4326).

Incorrect coordinate order may result in incorrect or failed routing requests.

## Hub Configuration

The current hub is defined as:

```python
HUB = (3.3822397715, 6.5913332783)
```

The coordinate is expressed as:

```text
(Longitude, Latitude)
```

To perform the analysis from a different hub, modify the `HUB` variable in the notebook.

## Project Structure

```text
Network_Analysis/
│
├── network_analysis.ipynb
├── Ojota_Hub.csv
├── README.md
├── requirements.txt
├── .env.example
├── .gitignore
│
└── outputs/
    ├── route_summary.csv
    ├── turn_by_turn.csv
    └── routes.geojson
```

The `outputs` directory is created automatically when the notebook runs.

Generated output files should normally be excluded from version control unless there is a specific reason to publish them.

## Running the Notebook

1. Clone the repository.
2. Install the required Python packages.
3. Obtain an OpenRouteService API key.
4. Create a `.env` file containing your API key.
5. Place the input CSV in the project directory.
6. Confirm that the CSV contains the required columns.
7. Open `network_analysis.ipynb`.
8. Run the notebook cells in order.

The notebook automatically creates the `outputs` directory if it does not already exist.

## Routing Configuration

Routes are requested using the OpenRouteService Directions service with:

```python
profile="foot-walking"
preference="shortest"
instructions=True
format="json"
```

The analysis therefore requests **walking routes** using the shortest-route preference.

The calculated route is determined by the routing network and data available through OpenRouteService. The resulting route should therefore be treated as a routing result and not as a field-surveyed or independently verified route.

## Outputs

### `route_summary.csv`

Contains a summary of successfully calculated routes.

| Field          | Description                         |
| -------------- | ----------------------------------- |
| `Destination`  | Destination name                    |
| `Distance_km`  | Route distance in kilometres        |
| `Duration_min` | Estimated route duration in minutes |

Distance and duration are rounded to two decimal places.

### `turn_by_turn.csv`

Contains turn-by-turn instructions returned by the routing service.

| Field         | Description                            |
| ------------- | -------------------------------------- |
| `Destination` | Destination name                       |
| `Step`        | Sequential instruction number          |
| `Instruction` | Turn-by-turn instruction               |
| `Street`      | Street associated with the instruction |

### `routes.geojson`

Contains the calculated route geometries as GeoJSON `LineString` features.

Each route contains:

* Destination name
* Streets encountered along the route
* Route geometry

The output is written using WGS 84 / EPSG:4326.

The GeoJSON can be opened in GIS applications such as QGIS, ArcGIS Pro and other software that supports the GeoJSON format.

## Error Handling

Destinations are processed individually.

If a routing request fails, the notebook reports the destination and the associated error and then continues processing the remaining destinations.

For example:

```text
Failed: Destination Name
<error message>
```

A failed request is not included in the corresponding output collections.

This allows a batch analysis to continue even when an individual destination cannot be routed.

## API Usage and Limitations

Each destination requires a routing request to the OpenRouteService Directions API.

The notebook includes a one-second delay between requests to reduce request frequency. This does not override or guarantee compliance with current API quotas or restrictions.

OpenRouteService publishes current restrictions for its services, including Directions. Users should check the official restrictions page before processing large datasets:

https://openrouteservice.org/restrictions/

For analyses involving a large number of destinations, review the current API limits and consider whether a different ORS service, such as the Matrix service, is more appropriate for distance/time calculations. ORS describes Matrix as a service designed for structured time-distance calculations between multiple origins and destinations.

## Reproducibility

The workflow uses project-relative paths rather than machine-specific paths such as:

```text
C:\GIS\...
```

This allows the repository to be cloned to another computer without modifying the file paths in the notebook.

The API key is kept outside the source code through the `.env` file.

Generated results are written automatically to the `outputs` directory.

## Security

Never commit any of the following to the repository:

* API keys
* Passwords
* Access tokens
* Personal credentials
* Private datasets
* Machine-specific paths containing sensitive information

Before pushing changes to GitHub, check the notebook and Git staging area to ensure that no credentials or sensitive information are included.

## Data and Routing Considerations

OpenRouteService uses geographic data and routing networks to calculate routes. Route availability and results can therefore depend on the underlying network data and its coverage.

A calculated route should not automatically be interpreted as confirmation that a location, road, footpath or access route has been physically verified.

For operational, engineering, safety-critical or legal applications, route results should be independently checked against appropriate authoritative or field data.

## License

Add an appropriate open-source licence before publishing the repository if you intend others to reuse, modify or distribute the code.

For example, the repository can use the MIT License if that matches your intended terms of use.

The use of OpenRouteService remains subject to the applicable OpenRouteService terms, policies and service restrictions.

## Author

Developed as a Python-based geospatial network analysis workflow.
