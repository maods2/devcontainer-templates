# Dev Container

To set up a local development environment, the `Dockerfile` inside the `.devcontainer` folder specifies all the dependencies and libraries required. This setup is designed to streamline the development process within this container. It includes the necessary libraries to run PySpark code locally while connecting to Google Cloud Platform (GCP) BigQuery.

### Prerequisites

1. Install Docker
2. Install the **Remote - Containers** extension for Visual Studio Code.
3. Use Visual Studio Code to open the development container.

The Visual Studio Code Dev Containers extension allows you to use a container as a full-featured development environment. It enables you to open any folder inside (or mounted into) a container and utilize the complete feature set of Visual Studio Code. A `devcontainer.json` file in your project specifies how VS Code should access (or create) a development container with a well-defined tool and runtime stack.

![Dev Container Architecture](https://code.visualstudio.com/assets/docs/devcontainers/containers/architecture-containers.png)

For more details, visit the [Dev Containers documentation](https://code.visualstudio.com/docs/devcontainers/containers).

### Customizations

The `devcontainer.json` file in this folder includes properties you may modify:

```json
"customizations": {
    "vscode": {
        "extensions": [
            "ms-python.python",
            ...
        ]
    }
}
```

To preload a specific VS Code extension in your dev container, add it to the `extensions` array as shown above.

```json
"mounts": [
    "source=${localEnv:HOME}${localEnv:USERPROFILE}/AppData/Roaming/gcloud/application_default_credentials.json,target=/tmp/keys/application_default_credentials.json,type=bind,consistency=cached"
]
```

For Windows users, the mount property will map a predefined volume to the container, mapping the GCP Authentication credential location. For Linux and Mac users, the source path for Google credentials will be slightly different.

## PySpark BigQuery Connector

To enable connection to GCP BigQuery via PySpark, we copy the JAR dependency into the container with this line:

```dockerfile
COPY ./.devcontainer/spark-3.2-bigquery-0.40.0.jar "${SPARK_EXTRA_JARS_DIR}"
```

If you need to update the Spark BigQuery JAR version due to incompatibilities with your preferred Spark version, download the new JAR from the [spark-bigquery-connector repository](https://github.com/GoogleCloudDataproc/spark-bigquery-connector), place it in the `.devcontainer` folder, and update the Docker COPY command.

To ensure the PySpark BigQuery connector works properly when running locally, define the dependency location when starting the Spark session:

```python
spark = SparkSession.builder.config(
    'spark.jars', '/opt/spark/jars/spark-3.2-bigquery-0.40.0.jar'
).getOrCreate()
```

To work locally, you also need to define the GCP `project_id` variable in PySpark connector queries:

```python
spark.read.format('bigquery')
.option('query', query)
.option('parentProject', project_id)
.option('viewsEnabled', 'true')
.option('materializationDataset', self.materialization_dataset)
.option('materializationExpirationTimeInMinutes', '120')
.schema(schema)
.load()
```



