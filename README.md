# Data Lake

## Purpose

A music streaming startup, Sparkify, has their data in Amazon S3. The data is contained in two directories:
1. Directory of JSON metadata for the songs in their app, and
2. Directory of JSON logs on user activity in their app.

The goal of this project is to build an ETL pipeline to:
- Extract data from S3,
- Transforms the data using Spark on a AWS cluster into a set of dimensional tables, and
- Loads them back to S3 as partioned parquet files.

These tables will be used by Sparkify's analytics team to gain insight into the songs that their users are listening to.

## Dataset

The first [dataset](s3://udacity-dend/song_data) contains the song data in JSON format. Files are partitioned by first three letters of each song's track ID e.g. `song_data/A/B/C/TRABCEI128F424C983.json`. Example of a single song file:
```
{
    "num_songs": 1, 
    "artist_id": "ARJIE2Y1187B994AB7", 
    "artist_latitude": null, 
    "artist_longitude": null, 
    "artist_location": "", 
    "artist_name": "Line Renaud", 
    "song_id": "SOUPIRU12A6D4FA1E1", 
    "title": "Der Kleine Dompfaff", 
    "duration": 152.92036, 
    "year": 0
}
```

The second [dataset](s3://udacity-dend/log_data) contains the log metadata. The log files are paratitioned by year and month e.g., `log_data/2018/11/2018-11-12-events.json`. Example entry in a log file:
```
{
    "artist": "Pavement",
    "auth": "Logged in",
    "firstName": "Sylvie",
    "gender": "F",
    "iteminSession": 0,
    "lastName": "Cruz",
    "length": 99.16036,
    "level": "free",
    "location": "Kiamath Falls, OR",
    "method": "PUT",
    "page": "NextSong",
    "registration": 1.540266e+12,
    "sessionId": 345,
    "song": "Mercy: The Laundromat",
    "status": 200,
    "ts": 1541990258796,
    "userAgent": "Mozzilla/5.0...",
    "userId": 10
}
```

## Data Lake Schema
This project implements a star schema. songplays is the fact table in the data model, while users, songs, artists, and time are all dimensional tables.

# Fact Table
`songplays` - records in event data associated with song plays (records with page=NextSong)

`start_time`, `userId`, `level`, `sessionId`, `location`, `userAgent`, `song_id`, `artist_id`, `songplay_id`

# Dimensional Tables
`users` - users of the Sparkify app.

`firstName`, `lastName`, `gender`, `level`, `userId`

`songs` - collection of songs.

`song_id`, `title`, `artist_id`, `year`, `duration`

`artists` - information about artists.

`artist_id`, `artist_name`, `artist_location`, `artist_lattitude`, `artist_longitude`

`time` - timestamps of records in songplays, deconstructed into various date-time parts.

`start_time`, `hour`, `day`, `week`, `month`, `year`, `weekday`


## Prerequisites

- Python3 is recommended as the environment

- pyspark (+ dependencies) to enable script to create a SparkSession

- AWS S3 bucket (us-west-2) for storing the parquet files

- IAM user with required permissions created in the same region as the source S3(us-west-2) to get the keys.

## ETL Pipeline

In context of Sparkify, this Data Lake ETL pipeline achieves the following:

1. Reads data from the S3 bucket.
2. Processes the data into analytics tables as per the schema using Spark.
3. Write the processed data back to S3 in the form of partitioned parquet files.

## How to run

1. Add appropriate AWS IAM Credentials in dl.cfg
2. Specify desired output data path in the main function of etl.py
3. Run etl.py
  `python3 etl.py`
  
## Files

`etl.py` - This script retrieves the song and log data in the s3 bucket, transforms the data into fact and dimensional tables then              loads the table data back into s3 as parquet files.

`dl.cfg` - Configuration file used that contains the AWS keys.

`README.md` - This very document which mentions the specifics and process of the project.

`/data` -  Folder containing sample data.

## Example queries

- Get count of rows in each Dimension table:

`SELECT COUNT(*)
FROM songs_table;`

`SELECT COUNT(*)
FROM artists_table;`

`SELECT COUNT(*)
FROM users_table;`

`SELECT COUNT(*)
FROM time_table;`


- Get count of rows in Fact table:

`SELECT COUNT(*)
FROM songplays_table;`


- Get users and songs they listened at particular time. Limit query to 1000 hits:

`SELECT  sp.songplay_id,
        u.user_id,
        s.song_id,
        u.last_name,
        sp.start_time,
        a.name,
        s.title
FROM songplays AS sp
        JOIN users   AS u ON (u.user_id = sp.user_id)
        JOIN songs   AS s ON (s.song_id = sp.song_id)
        JOIN artists AS a ON (a.artist_id = sp.artist_id)
        JOIN time    AS t ON (t.start_time = sp.start_time)
ORDER BY (sp.start_time)
LIMIT 1000;`