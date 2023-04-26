# BigQuery-Storage-Write-API-Default_Mode

This repo contains instructions and a code example of streaming data into BigQuery using the Storage Write API's default mode.

Documentation on how to use the Storage Write API can be found [HERE](https://cloud.google.com/bigquery/docs/write-api).

Please refer and copy the files from the write-api-default-example folder to your dev environment.


To get started, we’ll first create a table in BigQuery named “customer_records” through the below DDL statement. The DDL also partitions the table by Customer_Enrollment_Date and clusters the table by customer_ID.
  ```
  #Replace Test_Tables with your preferred dataset name
  CREATE TABLE `Test_Tables.Customer_Records` (
   Customer_ID INT64,
   Customer_Enrollment_Date DATE,
   Customer_Name STRING,
   Customer_Address STRING,
   Customer_Tier STRING,
   Active_Subscriptions JSON)
  PARTITION BY
   Customer_Enrollment_Date
  CLUSTER BY
   Customer_ID
  ```

Now, we’ll ingest some data via the Storage Write API. In this example, we’ll use Python, so we’ll stream data as protocol buffers. For a quick refresher on working with protocol buffers, [here’s a great tutorial](https://developers.google.com/protocol-buffers/docs/pythontutorial). 

Using Python, we’ll first align our protobuf messages to the table we created using a .proto file in proto2 format. Use the sample_data.proto file from the write-api-default-example folder you downloaded to your developer environment, then run the following command within to update your protocol buffer definition:
  ```
  protoc --python_out=. sample_data.proto
  ```

Within your developer environment, use this sample default_stream_python_script.py Python script to insert some new example customer records by reading from the new_customers.json file and writing into the customer_records table. This code uses the BigQuery Storage Write API to stream a batch of row data by appending proto2 serialized bytes to the serialzed_rows repeated field like the example below:
  ```
  row = sample_data_pb2.SampleData()
      row.Customer_ID = 1
      row.Customer_Enrollment_Date = “2022-11-05”
      row.Customer_Name = "Nick"
      row.Customer_Address = "1600 Amphitheatre Pkwy, Mountain View, CA"
      row.Customer_Tier = "Commercial"
      row.Active_Subscriptions = ‘{"Internet_Subscription":"Trial", "Music_Subscription":"Free"}’
  proto_rows.serialized_rows.append(row.SerializeToString())
  ```

We can now query our table and see it has ingested a few rows from the Storage Write API.
  ```
  SELECT * FROM `Test_Tables.Customer_Records`
  ```
