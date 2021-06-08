---
layout: default
title: Index transforms
nav_order: 40
parent: Index management
has_children: true
has_toc: false
---

# Index transforms

Whereas index rollup jobs let you reduce data granularity by rolling up old data into condensed indices, transform jobs let you create a different, summarized view of your data centered around certain fields, so you can visualize or analyze the data in different ways.

For example, suppose that you have airline data that’s scattered across multiple fields and categories, and you want to view a summary of the data that’s organized by airline, quarter, and then price. You can use a transform job to create a new, summarized index that’s organized by those specific categories.

You can use transform jobs in two ways:

1. Use the OpenSearch Dashboards UI to specify the index you want to transform and any optional data filters you want to use to filter the original index. Then select the fields you want to transform and the aggregations to use in the transformation. Finally, define a schedule for your job to follow.
2. Use the transforms API to specify all the details about your job: the index you want to transform, target groups you want the transformed index to have, any aggregations you want to use to group columns, and a schedule for your job to follow.

OpenSearch Dashboards provides a detailed summary of the jobs you created and their relevant information, such as associated indices and job statuses. You can review and edit your job’s details and selections before creation, and even preview a transformed index’s data as you’re choosing which fields to transform. However, you can also use the REST API to create transform jobs and preview transform job results, but you must know all of the necessary settings and parameters to submit them as part of the HTTP request body. Submitting your transform job configurations as JSON scripts offers you more portability, allowing you to share and replicate your transform jobs, which is harder to do using OpenSearch Dashboards.

Your use cases will help you decide which method to use to create transform jobs.

## Create a transform job

If you don't have any data in your cluster, you can use the sample flight data within OpenSearch Dashboards to try out transform jobs. Otherwise, after launching OpenSearch Dashboards, choose **Index Management**. Select **Transform Jobs**, and choose **Create Transform Job**.

### Step 1: Choose indices

1. In the **Job name and description** section, specify a name and an optional description for your job.
2. In the **Indices** section, select the source and target index. You can either select an existing target index or create a new one by entering a name for your new index. If you want to transform just a subset of your source index, choose **Add Data Filter**, and use the OpenSearch query DSL to specify a subset of your source index. For more information about the OpenSearch query DSL, see [query DSL](../../opensearch/query-dsl/).
3. Choose **Next**.

### Step 2: Select fields to transform

After specifying the indices, you can select the fields you want to use in your transform job, as well as whether to use groupings or aggregations.

You can use groupings to place your data into separate buckets in your transformed index. For example, if you want to group all of the airport destinations within the sample flight data, you can group the `DestAirportID` field into a target field of `DestAirportID_terms` field, and you can find the grouped airport IDs in your transformed index after the transform job finishes.

On the other hand, aggregations let you perform simple calculations. For example, you can include an aggregation in your transform job to define a new field of `sum_of_total_ticket_price` that calculates the sum of all airplane tickets, and then analyze the newly summer data within your transformed index.

1. In the data table, select the fields you want to transform and expand the drop-down menu within the column header to choose the grouping or aggregation you want to use.

    Currently, transform jobs support histogram, date_histogram, and terms groupings. For more information about groupings, see [Bucket Aggregations](../../opensearch/bucket-agg/). In terms of aggregations, you can select from `sum`, `avg`, `max`, `min`, `value_count`, `percentiles`, and `scripted_metric`. For more information about aggregations, see [Metric Aggregations](../../opensearch/metric-agg/).

2. Repeat step 1 for any other fields that you want to transform.
3. After selecting the fields that you want to transform and verifying the transformation, choose **Next**.

### Step 3: Specify a schedule

You can configure transform jobs to run once or multiple times on a schedule. Transform jobs are enabled by default.

1. For **transformation execution frequency**, select either **Define by fixed interval** and specify a **transform interval**.
2. Under **Advanced**, specify an optional amount for **Pages per execution**. A larger number means more data is processed in each search request, but also uses more memory and causes higher latency. Exceeding allowed memory limits can cause exceptions and errors to occur.
3. Choose **Next**.

### Step 4: Review and confirm details

After confirming your transform job’s details are correct, choose **Create Transform Job**. If you want to edit any part of the job, choose **Edit** of the section you want to change, and make the necessary changes. You can’t change aggregations or groupings after creating a job.

### Step 5: Search through the transformed index.

Once the transform job finishes, you can use the `_search` API operation to search the target index.

```json
GET <target_index>/_search
```

For example, after running a transform job that transforms the flight data based on a `DestAirportID` field, you can run the following request that returns all of the fields that have a value of `SFO`.

**Sample Request**

```json
GET finished_flight_job/_search
{
  "query": {
    "match": {
      "DestAirportID_terms" : "SFO"
    }
  }
}
```

**Sample Response**

```json
{
  "took" : 3,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 4,
      "relation" : "eq"
    },
    "max_score" : 3.845883,
    "hits" : [
      {
        "_index" : "finished_flight_job",
        "_type" : "_doc",
        "_id" : "dSNKGb8U3OJOmC4RqVCi1Q",
        "_score" : 3.845883,
        "_source" : {
          "transform._id" : "sample_flight_job",
          "transform._doc_count" : 14,
          "Carrier_terms" : "Kibana Airlines",
          "DestAirportID_terms" : "SFO"
        }
      },
      {
        "_index" : "finished_flight_job",
        "_type" : "_doc",
        "_id" : "_D7oqOy7drx9E-MG96U5RA",
        "_score" : 3.845883,
        "_source" : {
          "transform._id" : "sample_flight_job",
          "transform._doc_count" : 14,
          "Carrier_terms" : "Logstash Airways",
          "DestAirportID_terms" : "SFO"
        }
      },
      {
        "_index" : "finished_flight_job",
        "_type" : "_doc",
        "_id" : "YuZ8tOt1OsBA54e84WuAEw",
        "_score" : 3.6988301,
        "_source" : {
          "transform._id" : "sample_flight_job",
          "transform._doc_count" : 11,
          "Carrier_terms" : "ES-Air",
          "DestAirportID_terms" : "SFO"
        }
      },
      {
        "_index" : "finished_flight_job",
        "_type" : "_doc",
        "_id" : "W_-e7bVmH6eu8veJeK8ZxQ",
        "_score" : 3.6988301,
        "_source" : {
          "transform._id" : "sample_flight_job",
          "transform._doc_count" : 10,
          "Carrier_terms" : "JetBeats",
          "DestAirportID_terms" : "SFO"
        }
      }
    ]
  }
}

```