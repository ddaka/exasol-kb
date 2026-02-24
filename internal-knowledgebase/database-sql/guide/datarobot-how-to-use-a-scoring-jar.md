---
tool_name: internal-knowledgebase
doc_type: guide
category: database-sql
title: "Use a DataRobot Scoring JAR in Exasol"
summary: "Guide for loading DataRobot scoring JARs in Java UDFs, identifying predictor type, preparing feature maps, and executing model scoring."
---

# Use a DataRobot Scoring JAR in Exasol

## Overview

This guide explains how to use a downloadable DataRobot scoring JAR inside Exasol Java UDF workflows.

## Procedure

### 1. Load the scoring JAR in the UDF context

```sql
%jar <BucketFS-path>/5bfd063110c2301283e34cef.jar;
```

### 2. Detect model type

Try loading predictor classes:

```java
Class.forName("com.datarobot.prediction.RegressionPredictor");
```

or

```java
Class.forName("com.datarobot.prediction.ClassificationPredictor");
```

### 3. Create predictor instance

Regression:

```java
BaseRegressionPredictor predictor = BaseRegressionPredictor.getPredictor(modelId);
```

Classification:

```java
BaseClassificationPredictor predictor = BaseClassificationPredictor.getPredictor(modelId);
```

### 4. Inspect model features

```java
Map<String, Type> modelFeatures = predictor.getFeatures();
```

The map contains feature name and expected type.

### 5. Build input row and score

Input row format:

- `Map<String, Object>`
- key: feature name
- value: feature value (`double` or `String` depending on model expectations)

Score execution:

```java
Map<String, Double> score = predictor.score(row);
```

For classification models, retrieve class-specific score:

```java
Double pTrue = score.get("True");
```

## Implementation Notes

- Do not assume all training columns are used by the final model; iterate over `predictor.getFeatures()`.
- If UDF input names differ from model feature names, add explicit mapping logic.

## Downloads

- [datarobot_CSVTester.java](https://github.com/exasol/internal-knowledgebase/blob/main/Data-Science/attachments/datarobot_CSVTester.java)
- [datarobot_example.sql](https://github.com/exasol/internal-knowledgebase/blob/main/Data-Science/attachments/datarobot_example.sql)
