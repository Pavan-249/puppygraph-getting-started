# PuppyGraph Healthcare Demo

This repository contains a demonstration of analyzing healthcare data using Synthea, DuckDB, and PuppyGraph.

## Prerequisites

1. Python 3.x
2. Jupyter Notebook / JupyterLab
3. Docker (to run PuppyGraph)
4. Java 11+ (to run Synthea)

## 1. Generate the Synthea Dataset

Synthea™ is a Synthetic Patient Population Simulator. The Jupyter notebook processes a generated FHIR dataset.

1. Clone the Synthea repository (in the same directory as the notebook, or update the `FHIR_DIR` path in the notebook if placed elsewhere):
   ```bash
   git clone https://github.com/synthetichealth/synthea.git
   ```
2. Navigate to the Synthea directory and build it:
   ```bash
   cd synthea
   ./gradlew build check test
   ```
3. Run Synthea to generate a population of 15,000 patients:
   ```bash
   ./run_synthea -p 15000
   ```
   *Note: This will generate FHIR JSON files in `synthea/output/fhir`. Depending on your machine, generating 15,000 patients might take a few minutes.*

## 2. Run the Data Pipeline

1. Install the required Python packages:
   ```bash
   pip install pyarrow deltalake jupyterlab-rise pandas duckdb notebook
   ```
2. Open `puppygraph_demo.ipynb` in Jupyter Notebook:
   ```bash
   jupyter notebook puppygraph_demo.ipynb
   ```
3. Run all cells in the notebook. This will:
   - Parse the FHIR JSON files.
   - Extract patient, condition, and observation details.
   - Convert them to Parquet format in the `puppygraph_output/` folder.
   - Create a DuckDB file `puppygraph_output/healthcare.duckdb` that provides a SQL interface to the Parquet files.

## 3. Run PuppyGraph

PuppyGraph requires access to the schema configuration and the generated Parquet files.

1. Start PuppyGraph using Docker. We will mount our `puppygraph_output` directory to `/data` inside the container, which matches the paths configured in `schema.json`:
   ```bash
   docker run -d -p 8081:8081 -p 8182:8182 \
       -v $(pwd)/puppygraph_output:/data \
       -v $(pwd)/schema.json:/app/schema.json \
       puppygraph/puppygraph:latest
   ```
2. Open your browser and navigate to `http://localhost:8081` to access the PuppyGraph UI.
3. You can now visually explore and query the healthcare graph using Gremlin!
