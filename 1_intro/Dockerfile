FROM python:3.9.1

# We need to install wget to download the csv file
# RUN apt-get install wget

RUN pip install pandas sqlalchemy psycopg2

WORKDIR /app
COPY ingest_data.py ingest_data.py 
COPY yellow_tripdata_2021-01.csv yellow_tripdata_2021-01.csv

ENTRYPOINT [ "python", "ingest_data.py" ]

# docker build -t taxi_ingest:v001 .

# docker run -it \
#     --network=pg-network \
#     taxi_ingest:v001 \
#     --user=root \
#     --password=root \
#     --host=pg-database \
#     --port=5432 \
#     --db=ny_taxi \
#     --table_name=yellow_taxi_data \
#     --file="yellow_tripdata_2021-01.csv"