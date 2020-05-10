# Apache Airflow and DBT using Docker Compose
Stand-alone project that utilises public eCommerce data from Instacart to demonstrate how to schedule dbt models through Airflow.

## Requirements 
* Install [Docker](https://www.docker.com/products/docker-desktop)
* Install [Docker Compose](https://docs.docker.com/compose/install/)
* Download the [Kaggle Instacart eCommerce dataset](https://www.kaggle.com/c/instacart-market-basket-analysis/data) 

## Setup
* Clone the repository
* Extract the CSV files within ./sample_data directory

### Initialisation
Change directory within the repository and run `docker-compose up`. This will perform the following:
* Based on the definition of [`docker-compose.yml`](https://github.com/konosp/dbt-airflow-docker-compose/blob/master/docker-compose.yml) will download the necessary images to run the project. This includes a Postgres DB for Airflow to connect, Adminer (a lightweight DB client). Finally a Python 3.7-based image to host Airflow.
* 3 containers in total will be build and all necessary files will load.
* Within the Postgres service, all the CSV files will be loaded (this will take 2-3 minutes depending on your hardware). In the meantime, Airflow will try to connect to the DB unsuccessfully. It will restart until it manages to connect.

### Connections
* Adminer UI: [http://localhost:8080](http://localhost:8080/?pgsql=postgres&username=airflowuser&db=airflowdb&ns=dbt) Credentials as defined at [`docker-compose.yml`](https://github.com/konosp/dbt-airflow-docker-compose/blob/master/docker-compose.yml)
* Airflow UI: http://localhost:8000

## Docker Compose Commands
* Enable the services: `docker-compose up` or `docker-compose up -d` (detatches the terminal from the services' log)
* Disable the services: `docker-compose down` Non-destructive operation.
* Delete the services: `docker-compose rm` Ddeletes all associated data. The database will be empty on next run.
* Re-build the services: `docker-compose build` Re-builds the containers based on the docker-compose.yml definition. Since only the Airflow service is based on local files, this is the only image that is re-build (useful if you apply changes on the `./scripts_airflow/init.sh` file. 

## Project Notes
Because the project directories (`./scripts_postgres`, `./sample_data`, `./dbt` and `./airflow`) are defined as volumes in `docker-compose.yml`, they are directly accessible from within the containers. This means:
* On Airflow startup the existing models are compiled as part of the initialisation script. If you make changes to the models, you need to re-compile them. Two options:
* * From the host machine navigate to `./dbt` and then `dbt compile`
* * Attach to the container by `docker exec -it dbt-airflow-docker_airflow_1 /bin/bash`. This will open a session directly in the container running Airflow. Then CD into `/dbt` and  `dbt compile`. In general attaching to the container, helps a lot in debugging.
* You can make changes to the dbt models from the host machine, `dbt compile` them and on the next DAG update they will be available (beware of changes that are major and require `--full-refresh`). 
* The folder `./airflow/dags` stores the DAG files. Changes on the `dag.py` file (or introduction of new files) will appear after a few minutes in the Airflow Admin console because the folder is mounted on the container. This is handy so you do not have to turn docker-compose down and up all the time.

Credit to the very helpful repository: https://github.com/puckel/docker-airflow