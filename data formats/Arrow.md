---
tags:
  - data
  - arrow
gardening: 🌱
date: 2025-04-06
reference:
  - https://arrow.apache.org/docs/format/Intro.html
  - https://arrow.apache.org/docs/format/Columnar.html
  - https://blog.djnavarro.net/posts/2022-05-25_arrays-and-tables-in-arrow/
  - https://arrow.apache.org/docs/cpp/arrays.html
  - https://arrow.apache.org/docs/cpp/tables.html
---
Apache Arrow is a language-agnostic, columnar in-memory data format designed to significantly enhances performance for analytical queries compared to traditional row-based storage systems. In row-oriented representations, complete records (rows) are stored together, making them efficient for transactional processing but leading to inefficiencies for analytical workloads that operate on specific columns. In contrast, Arrow's columnar format organizes data by columns, facilitating vectorized computations (SIMD), improving [cache locality](../computer%20science/Locality%20of%20Reference%20and%20Memory%20Locality.md), and providing better compression. But this comes with the trade-off of comparatively more expensive mutation operations.

```c
typedef struct {
	int lon;
	int lat;
	int alt;
} Row;

// === Row format: array of structures ===
Row rows[] = {
	{1, 0, 10},
	{2, 9, 11},
	{3, 8, 12},
	{4, 7, 13},
	{5, 6, 14}
};

// === Columnar format: separate arrays for each column ===
int lonColumn[] = {1, 2, 3, 4, 5};
int latColumn[] = {0, 9, 8, 7, 6};
int altColumn[] = {10, 11, 12, 13, 14};
```

![](../../images/arrow/arrow-row-v-column-dark.png)
_Difference between row-based and column-based layouts_

![](../../images/arrow/arrow-memory-layout-dark.png)
_Memory layout of the two different formats_

## Core Concepts

**Columnar Layout**:  
Data is organized in a columnar format, which enhances performance for analytical workloads. This layout allows for efficient utilization of modern CPU features, such as SIMD (Single Instruction, Multiple Data).

**Interoperability**:  
The Arrow format is designed as a shared memory format that can be used across different programming languages and systems.

**Zero-Copy Sharing**:  
Data in the Arrow format can be shared between processes or libraries without the need for serialization or deserialization, which reduces overhead.

**Rich Data Types**:  
Arrow supports a wide variety of data types, including nested and hierarchical structures like structs and lists. It also allows for customization to accommodate domain-specific data types.

**Efficient Compression**:  
Arrow is compatible with efficient compression schemes, such as Apache Parquet, for on-disk storage.

**Standardized Metadata**:  
Arrow includes metadata that defines the schema, making it easy to understand and process the data without requiring additional documentation.

## Data Structure

### Schema

At the highest level of an Apache Arrow dataset is a schema, which serves as a blueprint defining the dataset's structure and metadata. This schema includes key components such as field names (identifiers for each column), data types (specifying the type of data, e.g., `int32`, `float64`, or complex types like lists or structs), nullability (indicating if a field can contain null values), and additional metadata for optimizations or processing rules. The standardized description provided by the schema ensures that different systems and languages can consistently interpret the in-memory data, while also detailing the hierarchical relationships between data elements.

![](../../images/arrow/arrow-schema-dark.png)

### Record Batch

Record batches are essential units consisting of a subset of rows that follow a predefined schema. Each record batch contains schema information inherited from the overall dataset, a count of the number of rows present in the batch, and column arrays, where each column is represented by an Arrow array that encapsulates all data for that column. By grouping rows together, Arrow enhances performance through batch processing and allows systems to manage data in manageable segments. Each record batch represents a table-like collection of data, with every column corresponding to a field defined in the schema. This structure ensures that all columns have the same number of rows, resulting in a rectangular block of memory that holds the data for a set of records.

### Array



### Validity

About validity. Avail on all types
Endianness, little-endian
```
values: [1, 2, 3, null, 5]
bitmap: 0x10111

values left-to-right, bitmap right-to-left

index   4 3 2 1 0
bitmap: 1 0 1 1 1
```

## Data Types



### Implementation Differences

Chunked Arrays and Tables are implementation specific and not a part of the overall spec.