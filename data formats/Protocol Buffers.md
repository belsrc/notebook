---
tags:
  - parquet
  - protobuf
gardening: 🌳
reference:
  - https://en.wikipedia.org/wiki/Protocol_Buffers
---
## Overview

The design goals for Protocol Buffers emphasized simplicity and performance. In particular, it was designed to be smaller and faster than XML.

Data structure schemas (called _messages_) and services are described in a proto definition file (`.proto`) and compiled with `protoc`.

Messages are serialized into a binary wire format which is compact, forward- and backward-compatible, but not self-describing (that is, there is no way to tell the names, meaning, or full datatypes of fields without an external specification). There is no defined way to include or refer to such an external specification (schema) within a Protocol Buffers file. The officially supported implementation includes an ASCII serialization format, but this format—though self-describing—loses the forward- and backward-compatibility behavior, and is thus not a good choice for applications other than human editing and debugging


## Limitations

Protobufs have no single specification. The format is best suited for small data chunks that don't exceed a few megabytes and can be loaded/sent into a memory right away and therefore is not a streamable format. The library doesn't provide compression out of the box. The format also isn't well supported in non–object-oriented languages.


## Example

A schema for a particular use of protocol buffers associates data types with field names, using integers to identify each field. (The protocol buffer data contains only the numbers, not the field names, providing some bandwidth/storage savings compared with systems that include the field names in the data.)

```cpp
// polyline.proto
syntax = "proto2";

message Point {
  required int32 x = 1;
  required int32 y = 2;
  optional string label = 3;
}

message Line {
  required Point start = 1;
  required Point end = 2;
  optional string label = 3;
}

message Polyline {
  repeated Point point = 1;
  optional string label = 2;
}
```

The "Point" message defines two mandatory data items, _x_ and _y_. The data item _label_ is optional. Each data item has a tag. The tag is defined after the equal sign. For example, _x_ has the tag 1.

The "Line" and "Polyline" messages, which both use Point, demonstrate how composition works in Protocol Buffers. Polyline has a _repeated_ field, and thus Polyline behaves like a set of points (of unspecified number).
