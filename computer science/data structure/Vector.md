---
tags:
  - data_structure
  - comp-sci
gardening: 🌱
date: 2025-04-22
reference:
  - https://computersciencewiki.org/index.php?title=Vector
  - https://codelikechamp.com/vector-data-structure/
  - https://www.cs.upc.edu/~jordicf/Teaching/programming/pdf/IP08_Vectors.pdf
  - https://en.wikibooks.org/wiki/A-level_Computing/AQA/Paper_1/Fundamentals_of_data_structures/Vectors
---
A vector, also known as a dynamic array, is a linear data structure that can grow or shrink at runtime. It provides random access (similar to a fixed array) while also allowing for the dynamic addition and removal of elements. Vectors allocate memory in a contiguous block, which enhances cache locality and improves memory usage compared to linked data structures. Some programming languages offer specialized vector implementations optimized for specific use cases, such as bit vectors for storing binary data or sparse vectors for managing large, mostly empty datasets.

Dynamic resizing operations, like adding or removing elements that exceed the allocated capacity, may lead to overhead due to memory reallocation and the need to copy elements. Additionally, inserting or deleting elements in the middle of a vector may require shifting subsequent elements, which results in linear time complexity and can negatively impact performance.

### API Examples

_Not full APIs_

```cpp
d.at(2) // Reference to the element at specified location
d.back() // Reference to the last element in the container
d.capacity() // The number of elements the container has space for
d.clear() // Erases all elements from the container. Returns size() to zero.
d.empty() // Checks if the container has no elements
d.erase(pos) // Erases the specified element and shifts elements
d.front() // Reference to the first element in the container
d.insert(pos, 200) // Inserts elements at the specified location
d.pop_back() // Removes the last element of the container
d.push_back("abc") // Appends a copy of value to the end of the container
d.size() // Returns the number of elements in the container
```

```rust
d.capacity() // The number of elements the vector currently has space for
d.clear() // Clears the vector, removing all values
d.insert(1, 'd') // Inserts an element at position, shifting elements right
d.is_empty() // true if the vector contains no elements
d.len() // The number of elements in the vector
d.pop() // Removes the last element from a vector and returns it
d.push(3) // Appends an element to the back of a collection
d.remove(1) // Removes/returns an element at position, shifting elements left
```
