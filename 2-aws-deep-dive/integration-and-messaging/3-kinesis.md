# Kinesis

Kinesis is a managed service that makes it easy to collect and analyze data and video streams in real-time. It is considered a managed alternative for Apache Kafka.

- Kinesis Streams: low latency streaming ingest at scale
- Kinesis Analytics: real-time analytics on streams using SQL
- Kinesis Firehose: load streams into S3, Redshift, opensearch etc.

## Overview

- Streams are divided in shards/partitions.
- Data is ordered in shards
- Data can be replayed
- Multiple applications can consume the same stream
- Data in Kinesis is immutable

## Shards

- A stream can contain one or more shards
- Write capacity: 1MB/s or 1000 messages per shard
- Read capacity: 2MB/s per shard
- Billing is per shard
- Records are ordered per shard

## Kinesis shard operations

- Shard splitting
  - if a shard is overwhelmed with huge traffic (hot shard) it can be split into two shards
  - once the data in the old shard has expired it will be deleted
  - one shard can be split only into a maximum of 2.
- Shard merging
  -  if 2 shards have very little traffic (cold shard) they can be merged into one shard
  -  only two shards can be merged into one shard
  -  once the data of the old shard expires the old shards will be deleted.
  -  Auto-scaling is not possible in Kinesis.

## Kinesis API

- PutRecord API: requires a partition key that gets hashed.
- Partition key should be highly distributed (otherwise a shard can become overwhelmed)
- PutRecord accepts batches, costs can be reduced
- PutRecord throws ProvisionedThroughputExceeded in case of capacity is exceeded. Solution: exponential back-off, more shards, ensure that partition key is highly distributed

## Kinesis Client Library (KCL)

- It is a Java library
- Each shard should be read by only one KCL instance!
- Progress checkpoint is written to DynamoDB
- Records are read in order at the shard level

## Kinesis Consumer types

- Shared (Classic) fan-out Consumer-pull:
  - limited throughput
  - 2mb/s across all consumers
  - Latency ~ 200ms
  - Consumer poll the data from stream , using GETRecordAPI call.
- Enhanced Fan-out Consumer-push:
  - multiple consuming application
  - 2mb/s per each consumer
  - Kinesis push the data to the consumer using HTTP/2(subscribe to shared API)
  - Latency ~ 70ms

## Security

- IAM policies
- Encryption at flight using HTTPS
- Encryption at rest using KMS
- Client side encryption/decryption (should be done manually)
- VPC Endpoints available for Kinesis

## Kinesis Data Analytics

- Real-time analytics on Kinesis Streams using SQL
- Pay for actual consumption, no need to pre-provision anything
- Can create streams from query results

## Kinesis Firehose

- Used for ETL, can load data into Redshift, S3, Amazonopensearch etc.
- Fully managed, automatic scaling
- Low latency (60 seconds)
- Pay for the amount of data which goes through Firehose
- No storage of data, therfore no replaying of data
- Support automatic scaling

## SQS vs SNS vs Kinesis

| SQS                                  | SNS                                           | Kinesis                              |
| -------------------------------------| --------------------------------------------- | ------------------------------------ |
| Consumer pulls data                  | Data is pushed to many subscribers            | Consumer pulls data                  |
| Data is deleted after consumed       | Up to 10 million subscribers                  | Can have as many consumer as we want |
| Can have as many consumer as we want | Data os not persisted                         | Can replay data                      |
| No need to provision throughput      | Pub/Sub                                       | Used for big data analytics and ETL  |
| No ordering (except FIFO)            | Up to 100k topics                             | Data expires after X days            |
| Individual message delay capability  | No need to provision throughput               | Must provision throughput            |
|                                      | Integrates with SQS for fan-out architecture  | Ordering at the shard level          |
