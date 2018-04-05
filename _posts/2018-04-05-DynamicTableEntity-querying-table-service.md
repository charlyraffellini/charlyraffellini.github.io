---
layout: post
title: Azure Storage .NET SDK and OData
description: About TableQuery, DynamicTableEntity and OData
date: 2018-04-05
categories: saas azure storage rest microsoft api odata
img: 2018/DynamicTableEntity-and-TableQuery.jpg
author: Carlos Raffellini
---

# DynamicTableEntity and TableQuery in Azure Storage SDK

- [DynamicTableEntity and TableQuery in Azure Storage SDK](#dynamictableentity-and-tablequery-in-azure-storage-sdk)
    - [The problem](#the-problem)
    - [The solution](#the-solution)
    - [Further understanding](#further-understanding)
    - [Conclusion](#conclusion)
    - [Bonus](#bonus)
        - [Inserting Rows Using Batch Operation](#inserting-rows-using-batch-operation)
        - [Rest API Batch Transaction](#rest-api-batch-transaction)
    - [References](#references)

## The problem

Query Storage Table in all partitions projecting only a few columns. I am using the Azure Storage SDK for .NET.

## The solution

```csharp
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    CloudTable table = tableClient.GetTableReference("people");

    TableQuery<DynamicTableEntity> projectionQuery =
        new TableQuery<DynamicTableEntity>()
            .Where("PartitionKey eq 'Smith'")
            .Select(new string[] {"Email"});

    EntityResolver<string> resolver = (pk, rk, ts, props, etag) =>
        props.ContainsKey("Email") ? props["Email"].StringValue : null;
```

## Further understanding

First we check the table *people* exists.

```http
GET https://charlietest.table.core.windows.net/Tables('people') HTTP/1.1
```

If the previous query returns `404`, we create a new Table.

```http
POST https://charlietest.table.core.windows.net/Tables() HTTP/1.1

{"TableName":"people"}
```

Assuming we have data in the table. This is the OData query for the request showed above:

```http
GET https://charlietest.table.core.windows.net/people?$filter=PartitionKey%20eq%20%27Smith%27&$select=Email%2CPartitionKey%2CRowKey%2CTimestamp HTTP/1.1
```

Response:

```
287
{"odata.metadata":"https://charlietest.table.core.windows.net/$metadata#people&$select=Email,PartitionKey,RowKey,Timestamp","value":[{"odata.etag":"W/\"datetime'2018-04-05T10%3A28%3A00.4488639Z'\"","PartitionKey":"Smith","RowKey":"Ben","Timestamp":"2018-04-05T10:28:00.4488639Z","Email":"Ben@contoso.com"},{"odata.etag":"W/\"datetime'2018-04-05T10%3A28%3A00.4488639Z'\"","PartitionKey":"Smith","RowKey":"Ed","Timestamp":"2018-04-05T10:28:00.4488639Z","Email":null},{"odata.etag":"W/\"datetime'2018-04-05T10%3A28%3A00.4488639Z'\"","PartitionKey":"Smith","RowKey":"Jeff","Timestamp":"2018-04-05T10:28:00.4488639Z","Email":"Jeff@contoso.com"}]}
0
```

## Conclusion

Using `TableQuery<DynamicTableEntity>` is used to create the OData REST query to the Table Service. In this way we manage to query in all the `Partitions`, filtering and projecting the results.

## Bonus

### Inserting Rows Using Batch Operation

```csharp
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    CloudTableClient tableClient = storageAccount.CreateCloudTableClient();

    CloudTable table = tableClient.GetTableReference("people");
    table.CreateIfNotExists();

    TableBatchOperation batchOperation = new TableBatchOperation();

    Guy customer1 = new Guy("Smith", "Jeff");
    customer1.Email = "Jeff@mail.com";
    customer1.PhoneNumber = "425-555-0104";

    Guy customer2 = new Guy("Smith", "Ben");
    customer2.Email = "Ben@mail.com";
    customer2.PhoneNumber = "425-555-0102";

    Guy customer3 = new Guy("Smith", "Ed");
    customer2.Email = "Ben@mail.com";
    customer2.PhoneNumber = "425-555-0102";

    batchOperation.Insert(customer1);
    batchOperation.Insert(customer2);
    batchOperation.Insert(customer3);
    table.ExecuteBatch(batchOperation);
```

### Rest API Batch Transaction

Batch Transactions are documented [here](https://docs.microsoft.com/en-us/rest/api/storageservices/performing-entity-group-transactions).

I am highlighting a few things:

- `Change set`: is a group of one or more insert, update, or delete operations.
- `Batch`: is a container of operations, including one or more change sets and query operations.
- The transaction can include at most 100 entities, and its total payload may be no more than 4 MB in size.
- All entities subject to operations as part of the transaction must have the same `PartitionKey`.

```http
POST https://charlietest.table.core.windows.net/$batch HTTP/1.1
x-ms-version: 2015-12-11
Accept-Charset: UTF-8
x-ms-date: Thu, 05 Apr 2018 10:28:00 GMT
Authorization: SharedKey charlietest:b+uSR36KQCr2j62hI8FXdxXf6ec70stQhsfkNcmvaQQ=

--batch_d6b79d54-b442-473b-a086-dcc63be2bbee
Content-Type: multipart/mixed; boundary=changeset_b4caa82e-bcde-42ad-a806-81a6393bb82e

--changeset_b4caa82e-bcde-42ad-a806-81a6393bb82e
Content-Type: application/http
Content-Transfer-Encoding: binary

POST https://charlietest.table.core.windows.net/people() HTTP/1.1
Accept: application/json;odata=minimalmetadata
Content-Type: application/json
Prefer: return-no-content
DataServiceVersion: 3.0;

{"PartitionKey":"Smith","RowKey":"Jeff","Email":"Jeff@contoso.com","PhoneNumber":"425-555-0104"}
--changeset_b4caa82e-bcde-42ad-a806-81a6393bb82e
Content-Type: application/http
Content-Transfer-Encoding: binary

POST https://charlietest.table.core.windows.net/people() HTTP/1.1
Accept: application/json;odata=minimalmetadata
Content-Type: application/json
Prefer: return-no-content
DataServiceVersion: 3.0;

{"PartitionKey":"Smith","RowKey":"Ben","Email":"Ben@contoso.com","PhoneNumber":"425-555-0102"}
--changeset_b4caa82e-bcde-42ad-a806-81a6393bb82e
Content-Type: application/http
Content-Transfer-Encoding: binary

POST https://charlietest.table.core.windows.net/people() HTTP/1.1
Accept: application/json;odata=minimalmetadata
Content-Type: application/json
Prefer: return-no-content
DataServiceVersion: 3.0;

{"PartitionKey":"Smith","RowKey":"Ed","Email":null,"PhoneNumber":null}
--changeset_b4caa82e-bcde-42ad-a806-81a6393bb82e--
--batch_d6b79d54-b442-473b-a086-dcc63be2bbee--

```

---

## References

[DynamicTableEntity Class](https://docs.microsoft.com/en-us/dotnet/api/microsoft.windowsazure.storage.table.dynamictableentity?view=azure-dotnet)

[TableQuery<TElement> Class](https://docs.microsoft.com/en-us/dotnet/api/microsoft.windowsazure.storage.table.tablequery-1?view=azure-dotnet)

[Performing Entity Group Transactions, Storage Service REST API](https://docs.microsoft.com/en-us/rest/api/storageservices/performing-entity-group-transactions)

[Using .NET Table API](https://docs.microsoft.com/en-us/azure/cosmos-db/table-storage-how-to-use-dotnet)

[.NET Storage API](https://docs.microsoft.com/en-us/dotnet/api/overview/azure/storage?view=azure-dotnet)
