# TSBS Supplemental Guide: Akumuli

Akumuli is an open-source time-series database written in C++ with. 
performance in mind. This guide explains how to use TSBS to generate
test data and run the tests.

## Data format

Data generated by `tsbs_generate_data` for Akumuli is serialized in a
dictionary-based RESP format. The header of the file contains dictionary entries. 
Every entry contains unique series name and id. 
The header is followed by data-points which are using ids instead of full series names. 
The text data in the input file is interleaved with binary data 
wich acts as a cue for `tsbs_load_akumuli` tool. It uses them for emulating 
realclients that send data independently without actually parsing the messages.

---

## `tsbs_load_akumuli` Additional Flags

#### `--endpoint` (type: `string`, default: `http://localhost:8282`)

TCP endpoint to connect to for inserting data. Workers will create individual connections.

---

## `tsbs_run_queries_akumuli` Additional Flags

#### `--endpoint` (type: `string`, default: `http://localhost:8181`)

HTTP API endpoint to run queries.

---

## Getting started

You can install Akumuli from this [packagecloud repository](https://packagecloud.io/Lazin/Akumuli).
It contains pre-built amd64 and arm64 packages for Debian, Ubuntu, and CentOS.
After installing you can run `akumulid --init` to create configuration file (~/.akumulid).
You can set the database size there. Also, you can remove or comment out 'WAL' section (Akumuli is
marginally slower if WAL is enabled). 
The next step is to create the database using `akumulid --create`. This will create the database files.
After that you can run the database by running `akumulid` without parameters.

To start over you should stop the database by sending SIGINT to the akumulid process followed by 
`akumulid --delete; akumulid --create`. 

#### Generating the data

Run `tsbs_generate_data` with `-format=akumuli` to generate the input.
Here is the sample command:
`./cmd/tsbs_generate_data/tsbs_generate_data --use-case="cpu-only" --seed=123 --scale=1000 --timestamp-start="2016-01-01T00:00:00Z" --timestamp-end="2016-01-02T00:00:00Z" --log-interval="10s" --format="akumuli" | gzip > /tmp/akumuli-data.gz`

The easiest way to generate queries is to use `scripts/generate_queries.sh` script.
`FORMATS="akumuli" SCALE=1000 SEED=123 TS_START="2016-01-01T00:00:00Z" TS_END="2016-01-02T00:00:01Z" QUERY_TYPES="cpu-max-all-1 cpu-max-all-8 double-groupby-1 double-groupby-5 double-groupby-all high-cpu-1 high-cpu-all lastpoint single-groupby-1-1-1 single-groupby-1-1-12 single-groupby-1-8-1 single-groupby-5-1-1 single-groupby-5-1-12" QUERIES=1000 BULK_DATA_DIR="/tmp/bulk_queries" scripts/generate_queries.sh`

#### Loading the data

This can be done using `scripts/load_akumuli.sh`. Note that the script expects certain file name format.
`NUM_WORKERS=2 BATCH_SIZE=10000 BULK_DATA_DIR=/tmp scripts/load_akumuli.sh`

#### Running the queries

This could be done the same way documentation suggests.
`cat /tmp/bulk_queries/akumuli-high-cpu-1-queries.gz | gunzip | tsbs_run_queries_akumuli`