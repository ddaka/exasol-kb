---
tool_name: internal-knowledgebase
doc_type: guide
category: database-sql
title: "Import Data from MongoDB to Exasol with IMPORT FROM JDBC"
summary: "Configure a MongoDB JDBC connection in Exasol and run IMPORT FROM JDBC statements for document and aggregation-based extracts."
---

# Import Data from MongoDB to Exasol with IMPORT FROM JDBC

## Overview

This guide shows how to import data from MongoDB into Exasol using `IMPORT FROM JDBC`.

## Prerequisites

- MongoDB JDBC driver package available in Exasol.
- Network access from Exasol to MongoDB.
- Valid MongoDB credentials.
- Target schema and table design prepared in Exasol.

## Procedure

### 1. Upload MongoDB JDBC driver

Upload the MongoDB JDBC driver into your Exasol environment.

Reference driver project:

- <https://github.com/wise-coders/mongodb-jdbc-driver>

### 2. Create a JDBC connection in Exasol

```sql
CREATE OR REPLACE CONNECTION jdbc_mongo
TO 'jdbc:mongodb+srv://<user>:<password>@<cluster-host>/?retryWrites=true&w=majority&expand=true';
```

## Import Examples

### Example 1: Import selected fields from a collection

```sql
IMPORT
INTO (
    id INTEGER,
    listing_url VARCHAR(128),
    bedrooms INT
)
FROM JDBC AT jdbc_mongo STATEMENT
'sample_airbnb.listingsAndReviews.find(
    {minimum_nights:"12"},
    {"_id":1,"listing_url":1,"bedrooms":1}
)';
```

### Example 2: Import transformed date values via aggregation

```sql
IMPORT
INTO (
    id INTEGER,
    x DATE
)
FROM JDBC AT jdbc_mongo STATEMENT
'sample_airbnb.listingsAndReviews.aggregate([
  {
    $project: {
      yearMonthDayUTC: {
        $dateToString: { format: "%Y-%m-%d", date: "$last_scraped" }
      }
    }
  }
])';
```

## Validation

- Run a limited import and verify row counts.
- Check datatype compatibility for projected fields.
- Confirm date and timestamp transformations are correct.

## Security Notes

- Do not store secrets in article text or SQL history.
- Use credential management approved by your environment.
- Restrict connection permissions to required schemas and collections only.
