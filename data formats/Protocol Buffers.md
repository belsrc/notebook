---
tags:
  - parquet
  - protobuf
  - data
gardening: 🌳
reference:
  - https://en.wikipedia.org/wiki/Protocol_Buffers
  - https://protobuf.dev/overview/
---
## Overview

The design goals for Protocol Buffers emphasized simplicity and performance. In particular, it was designed to be smaller and faster than XML.

Data structure schemas (called _messages_) and services are described in a proto definition file (`.proto`) and compiled with `protoc`.

Messages are serialized into a binary wire format which is compact, forward- and backward-compatible, but not self-describing (that is, there is no way to tell the names, meaning, or full datatypes of fields without an external specification). There is no defined way to include or refer to such an external specification (schema) within a Protocol Buffers file. The officially supported implementation includes an ASCII serialization format, but this format—though self-describing—loses the forward- and backward-compatibility behavior, and is thus not a good choice for applications other than human editing and debugging.

## What Are Protocol Buffers

Protocol Buffers (protobuf) is a language-neutral, platform-neutral extensible mechanism for serializing structured data developed by Google in 2001 and open-sourced in 2008. The format uses a schema-based approach where data structures are defined in `.proto` files and compiled into language-specific code.

The key innovation is the use of **field numbers** instead of field names in the binary representation, combined with a variable-length integer encoding scheme that minimizes wire size.

## When to Use Protocol Buffers

**Good Use Cases:**
- Microservice communication (especially with gRPC)
- Mobile app to server communication (bandwidth-constrained)
- Inter-process communication (IPC)
- Long-term data storage with schema evolution needs
- High-throughput data pipelines
- Configuration files that need versioning
- Binary log formats for analytics

**Poor Use Cases:**
- Human-readable configuration (use JSON/YAML)
- Debugging/logging output (not human-readable)
- Documents larger than a few MB (not streamable)
- When you need to query data without deserializing (use Parquet/database)
- Web browser communication (JSON is native, protobuf requires extra libraries)
- Quick prototyping where schema changes constantly

## Basic Workflow

```
┌────────────────┐      ┌──────────┐      ┌──────────────────┐
│                │      │          │      │                  │
│  .proto Schema │─────▶│  Protoc  │─────▶│ Generated Code   │
│  File          │      │ Compiler │      │ (.ts, .rs, .cpp) │
│                │      │          │      │                  │
└────────────────┘      └──────────┘      └──────────────────┘
       │                                          │
       │                                          ▼
       │                              ┌──────────────────┐
       │                              │ Your application │
       │                              │ imports and uses │
       │                              │ generated types  │
       └─────────────────────────────▶│                  │
         (shared with other services) └──────────────────┘
```

## Field Types and Modifiers

### Proto2 vs Proto3

**Proto2** uses `required`, `optional`, and `repeated`:

```protobuf
syntax = "proto2";

message User {
  required string username = 1;
  optional string email = 2;
  repeated string tags = 3;
}
```

**Proto3** simplified this (all fields are optional by default):

```protobuf
syntax = "proto3";

message User {
  string username = 1;
  string email = 2;
  repeated string tags = 3;
}
```

### Scalar Types

| Proto Type | C++      | TypeScript | Rust       |
|------------|----------|------------|------------|
| double     | double   | number     | f64        |
| float      | float    | number     | f32        |
| int32      | int32    | number     | i32        |
| int64      | int64    | bigint     | i64        |
| uint32     | uint32   | number     | u32        |
| uint64     | uint64   | bigint     | u64        |
| sint32     | int32    | number     | i32        |
| sint64     | int64    | bigint     | i64        |
| fixed32    | uint32   | number     | u32        |
| fixed64    | uint64   | bigint     | u64        |
| sfixed32   | int32    | number     | i32        |
| sfixed64   | int64    | bigint     | i64        |
| bool       | bool     | boolean    | bool       |
| string     | string   | string     | String     |
| bytes      | string   | Uint8Array | Vec\<u8\>  |

## Schema Evolution and Compatibility

### Forward Compatibility

Older code can read data from newer code:
- New fields are ignored by old parsers
- Field numbers must never be reused

### Backward Compatibility

Newer code can read data from older code:
- New fields have default values when missing
- `optional` fields can be added safely

### Rules for Safe Schema Evolution

1. **Never change field numbers**
2. **Never change field types** (with some exceptions)
3. **Use `reserved` for deleted fields:**
```protobuf
   message Foo {
     reserved 2, 15, 9 to 11;
     reserved "foo", "bar";
   }
```
4. **Always add new fields, never remove old ones**
5. **Make new fields optional**

## Editions: The Future of Protocol Buffers

Starting in 2023, Protocol Buffers introduced **editions** as a replacement for the `syntax = "proto2"` and `syntax = "proto3"` declarations. Editions provide a more flexible evolution model.

### What Are Editions?

Instead of major version bumps (proto2 → proto3), Protocol Buffers now uses year-based editions that allow incremental changes:

```protobuf
edition = "2023";

message User {
  string name = 1;
  int32 age = 2;
}
```

### Edition 2023

Edition 2023 is essentially proto3 with feature flags:

```protobuf
edition = "2023";

message Point {
  int32 x = 1;
  int32 y = 2;
}
```

**Defaults in Edition 2023:**
- All fields are optional by default (like proto3)
- Repeated fields are packed by default (like proto3)
- No `required` keyword (like proto3)
- Can opt-in to proto2 behaviors using features

### Edition 2024

Edition 2024 introduced refinements and new capabilities:

```protobuf
edition = "2024";

message Event {
  string event_id = 1;
  int64 timestamp = 2;
}
```

**Changes in Edition 2024:**
- Improved defaults for certain encoding behaviors
- Better support for modern language features
- Incremental language improvements

### Feature Flags

Editions allow fine-grained control through features, which can be set at file, message, or field level:

```protobuf
edition = "2023";

// File-level feature
option features.field_presence = EXPLICIT;

message Config {
  // Message-level feature
  option features.repeated_field_encoding = EXPANDED;
  
  repeated int32 values = 1;
  
  // Field-level feature
  string name = 2 [features.field_presence = IMPLICIT];
}
```

**Common Features:**
- `field_presence`: Controls how presence is tracked (EXPLICIT, IMPLICIT, LEGACY_REQUIRED)
- `enum_type`: Controls enum behavior (OPEN, CLOSED)
- `repeated_field_encoding`: Controls repeated field packing (PACKED, EXPANDED)
- `utf8_validation`: Controls string validation (VERIFY, NONE)
- `message_encoding`: Controls message encoding strategy

### Migration Path

```protobuf
// Old proto3
syntax = "proto3";

message User {
  string name = 1;
}
```

Becomes:

```protobuf
// New edition-based
edition = "2023";

message User {
  string name = 1;
}
```

### Editions vs Proto2/Proto3

| Aspect | Proto2/Proto3 | Editions |
|--------|---------------|----------|
| Version model | Major versions | Year-based incremental |
| Feature control | All-or-nothing | Granular feature flags |
| Migration | Hard break | Smooth evolution |
| Defaults | Fixed per syntax | Can be overridden |
| Future-proofing | Limited | Better |

### Should You Use Editions?

**Use Editions if:**
- Starting a new project in 2024+
- Want fine-grained control over features
- Need to gradually migrate from proto2/proto3
- Want to future-proof your schemas

**Use Proto3 if:**
- Working with older tooling
- Need maximum compatibility
- Team is not yet familiar with editions
- Existing codebase uses proto3

**Use Proto2 if:**
- Maintaining legacy code
- Stuck on older protoc versions

### Practical Edition 2023 Example

```protobuf
edition = "2023";

package example;

// Use explicit presence for optional semantics
option features.field_presence = EXPLICIT;

message UserProfile {
  string user_id = 1;
  string email = 2;
  
  // Override to implicit for this field
  string display_name = 3 [features.field_presence = IMPLICIT];
  
  // Repeated fields are packed by default in edition 2023
  repeated string tags = 4;
  
  // Can opt-out of packing if needed
  repeated int32 legacy_values = 5 [features.repeated_field_encoding = EXPANDED];
}
```

### Edition 2024 Improvements

Edition 2024 refined several behaviors and introduced quality-of-life improvements. The changes are incremental and focus on:

- More consistent UTF-8 validation defaults
- Better enum semantics
- Improved JSON encoding behaviors
- Enhanced compatibility with modern programming languages

**Recommendation:** For new projects started in 2024 or later, use `edition = "2024"`. For existing projects, proto3 remains a solid choice until you're ready to migrate.

## Comparison with Other Formats

| Feature         | Protobuf  | JSON     | XML      | Avro      |
| --------------- | --------- | -------- | -------- | --------- |
| Size            | Small     | Medium   | Large    | Smallest  |
| Speed           | Fast      | Medium   | Slow     | Fastest   |
| Schema          | Required  | Optional | Optional | Required  |
| Human Readable  | No        | Yes      | Yes      | No        |
| Evolution       | Excellent | Good     | Good     | Excellent |
| Self-Describing | No        | Yes      | Yes      | Yes       |

**Size Comparison Example:**

Given this data:

```json
{
  "name": "John Doe",
  "age": 30,
  "email": "john@example.com"
}
```

- JSON: 62 bytes
- Protobuf: 34 bytes
- XML: 98 bytes

## Limitations

Protobufs have no single specification. The format is best suited for small data chunks that don't exceed a few megabytes and can be loaded/sent into a memory right away and therefore is not a streamable format. The library doesn't provide compression out of the box. The format also isn't well supported in non-object-oriented languages.

Additional limitations:
- No polymorphism or inheritance
- Limited to 2GB message size (due to 32-bit length prefix)
- No native support for nullable fields in proto3 (requires `google.protobuf.NullValue` wrapper)
- Schema must be known at compile time
- Binary format is not human-readable
- Cannot represent circular references
- No built-in versioning in the format itself

## Binary Wire Format

### Encoding Principles

Each field in a protobuf message is encoded as a key-value pair where:
- **Key** = `(field_number << 3) | wire_type`
- **Value** = The actual data, encoded according to wire type

### Wire Types

| Type | Meaning          | Used For                                                 |
| ---- | ---------------- | -------------------------------------------------------- |
| 0    | Varint           | int32, int64, uint32, uint64, sint32, sint64, bool, enum |
| 1    | 64-bit           | fixed64, sfixed64, double                                |
| 2    | Length-delimited | string, bytes, embedded messages, repeated fields        |
| 5    | 32-bit           | fixed32, sfixed32, float                                 |

### Varint Encoding

Varints encode integers using one or more bytes where the most significant bit (MSB) indicates continuation:

- MSB = 1: More bytes follow
- MSB = 0: Last byte

For example, encoding the number 300:

```
300 in binary: 100101100
Split into 7-bit groups: 0000010 0101100
Reverse order: 0101100 0000010
Add continuation bits: 10101100 00000010
Result in hex: 0xAC 0x02
```

### Zigzag Encoding

Signed integers use zigzag encoding to map signed numbers to unsigned. This is necessary because regular varint encoding is inefficient for negative numbers.

With regular varint encoding (used by `int32` and `int64`), negative numbers are represented using two's complement, which means all high-order bits are set to 1. This causes negative numbers to always use the maximum bytes:

```
-1 in 32-bit two's complement: 11111111 11111111 11111111 11111111
As varint: 5 bytes (0xFF 0xFF 0xFF 0xFF 0x0F)

-2 in 32-bit two's complement: 11111111 11111111 11111111 11111110
As varint: 5 bytes (0xFE 0xFF 0xFF 0xFF 0x0F)
```

**The Zigzag Solution:**

Zigzag encoding (used by `sint32` and `sint64`) maps signed integers to unsigned integers so that small absolute values encode efficiently:

$$\text{zigzag}(n) = (n \ll 1) \oplus (n \gg 31) \text{ for 32-bit}$$
$$\text{zigzag}(n) = (n \ll 1) \oplus (n \gg 63) \text{ for 64-bit}$$

**Mapping:**
```
Signed → Unsigned
     0 → 0
    -1 → 1
     1 → 2
    -2 → 3
     2 → 4
    -3 → 5
     3 → 6
```

**Detailed Example - Encoding -1:**

```
Step 1: Start with -1 (32-bit)
Binary: 11111111 11111111 11111111 11111111

Step 2: Left shift by 1
n << 1: 11111111 11111111 11111111 11111110

Step 3: Arithmetic right shift by 31 (sign extension)
n >> 31: 11111111 11111111 11111111 11111111

Step 4: XOR the results
11111111 11111111 11111111 11111110
⊕
11111111 11111111 11111111 11111111
=
00000000 00000000 00000000 00000001 = 1

Step 5: Encode 1 as varint
Result: 0x01 (1 byte)
```

**Detailed Example - Encoding -2:**

```
Step 1: Start with -2 (32-bit)
Binary: 11111111 11111111 11111111 11111110

Step 2: Left shift by 1
n << 1: 11111111 11111111 11111111 11111100

Step 3: Arithmetic right shift by 31
n >> 31: 11111111 11111111 11111111 11111111

Step 4: XOR the results
11111111 11111111 11111111 11111100
⊕
11111111 11111111 11111111 11111111
=
00000000 00000000 00000000 00000011 = 3

Step 5: Encode 3 as varint
Result: 0x03 (1 byte)
```

**Comparison Table:**

| Value | int32 (varint only) | sint32 (zigzag + varint) | Bytes Saved |
|-------|---------------------|--------------------------|-------------|
| 0     | 1 byte (0x00)       | 1 byte (0x00)            | 0           |
| -1    | 5 bytes             | 1 byte (0x01)            | 4           |
| 1     | 1 byte (0x01)       | 1 byte (0x02)            | 0           |
| -2    | 5 bytes             | 1 byte (0x03)            | 4           |
| 2     | 1 byte (0x02)       | 1 byte (0x04)            | 0           |
| -64   | 5 bytes             | 1 byte (0x7F)            | 4           |
| 64    | 1 byte (0x40)       | 2 bytes (0x80 0x01)      | -1          |
| -65   | 5 bytes             | 2 bytes (0x81 0x01)      | 3           |

**Key Insight:** Use `sint32`/`sint64` when your values are frequently negative or when the range is symmetric around zero (e.g., temperature, offsets, deltas). Use `int32`/`int64` when values are predominantly positive.

## Common Patterns

### Wrapper Messages for APIs

```protobuf
syntax = "proto3";

// Request/Response pattern
message CreateUserRequest {
  string username = 1;
  string email = 2;
}

message Response {
  oneof result {
    User user = 1;
    Error error = 2;
  }
}

// Or use optional "tuple" type response
message CreateUserResponse {
  bool success = 1;
  string user_id = 2;
  string error_message = 3;  // populated if success = false
}
```

### Timestamps

```protobuf
syntax = "proto3";

import "google/protobuf/timestamp.proto";
import "google/protobuf/duration.proto";

message Event {
  string event_id = 1;
  google.protobuf.Timestamp occurred_at = 2;
  google.protobuf.Duration processing_time = 3;
}
```

### Enumerations with Reserved Values

```protobuf
enum Status {
  // Always reserve 0 for unknown/unspecified
  STATUS_UNSPECIFIED = 0;
  STATUS_PENDING = 1;
  STATUS_ACTIVE = 2;
  STATUS_COMPLETED = 3;
  STATUS_FAILED = 4;
}
```

### Using Maps

```protobuf
message UserPreferences {
  map<string, string> settings = 1;
  map<string, int32> counters = 2;
}
```

### Nested Messages

```protobuf
message Company {
  string name = 1;
  
  message Address {
    string street = 1;
    string city = 2;
    string country = 3;
  }
  
  Address headquarters = 2;
  repeated Address offices = 3;
}
```

## Common Mistakes and Pitfalls

### ❌ Changing Field Numbers

```protobuf
// Version 1
message User {
  string name = 1;
  int32 age = 2;
}

// Version 2 - WRONG!
message User {
  string name = 2;  // Changed from 1 to 2
  int32 age = 1;    // Changed from 2 to 1
}
```

**Result:** Completely breaks compatibility. Old data will be misinterpreted.

### ❌ Forgetting to Set Proto Syntax

```protobuf
// Missing syntax declaration defaults to proto2
message User {
  string name = 1;  // Actually requires 'required' or 'optional' in proto2
}
```

**Always specify:** `syntax = "proto3";` or `syntax = "proto2";`

### ❌ Using 0 Values in Proto3

```protobuf
message Settings {
  bool enabled = 1;  // Default is false
  int32 timeout = 2; // Default is 0
}
```

**Problem:** Can't distinguish between "not set" and "set to 0/false". Use wrapper types if needed:

```protobuf
import "google/protobuf/wrappers.proto";

message Settings {
  google.protobuf.BoolValue enabled = 1;  // null vs true vs false
  google.protobuf.Int32Value timeout = 2; // null vs 0 vs other
}
```

### ❌ Large Repeated Fields Without Packing

```protobuf
// Inefficient for large arrays in proto2
repeated int32 values = 1;

// Better (proto3 does this automatically)
repeated int32 values = 1 [packed=true];
```

### ❌ Not Using Reserved Fields

```protobuf
message User {
  string name = 1;
  // Deleted: int32 age = 2;  // WRONG - don't just delete
}

// CORRECT
message User {
  reserved 2;
  reserved "age";
  string name = 1;
}
```

### ❌ Choosing Wrong Integer Type

```protobuf
// Inefficient for negative numbers
int32 temperature = 1;  // Uses 5 bytes for -1

// Better for negative numbers
sint32 temperature = 1;  // Uses 1 byte for -1

// Use when values are always positive and large
uint32 count = 2;
```

## Real-World Examples

### gRPC Service Definition

Protocol Buffers are the default serialization format for gRPC:

```protobuf
syntax = "proto3";

service UserService {
  rpc GetUser(GetUserRequest) returns (User);
  rpc ListUsers(ListUsersRequest) returns (stream User);
  rpc CreateUser(CreateUserRequest) returns (User);
}

message GetUserRequest {
  string user_id = 1;
}

message ListUsersRequest {
  int32 page_size = 1;
  string page_token = 2;
}

message CreateUserRequest {
  string username = 1;
  string email = 2;
  map<string, string> metadata = 3;
}

message User {
  string user_id = 1;
  string username = 2;
  string email = 3;
  int64 created_at = 4;
  map<string, string> metadata = 5;
}
```

### Event Logging System

```protobuf
syntax = "proto3";

message LogEvent {
  enum Severity {
    UNKNOWN = 0;
    DEBUG = 1;
    INFO = 2;
    WARNING = 3;
    ERROR = 4;
    CRITICAL = 5;
  }
  
  string event_id = 1;
  int64 timestamp_ms = 2;
  Severity severity = 3;
  string source = 4;
  string message = 5;
  
  map<string, string> labels = 6;
  
  oneof payload {
    ErrorDetails error = 7;
    MetricData metric = 8;
    TraceData trace = 9;
  }
}

message ErrorDetails {
  string error_code = 1;
  string stack_trace = 2;
  string user_id = 3;
}

message MetricData {
  string metric_name = 1;
  double value = 2;
  string unit = 3;
}

message TraceData {
  string trace_id = 1;
  string span_id = 2;
  int64 duration_ms = 3;
}
```
