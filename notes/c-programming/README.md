# C Programming

## Table of Contents
- [Program Memory](#program-memory)
- [Pointers](#pointers)
  - Pass by Value vs Pass by Reference
  - Pointer Arithmetic
  - Pointer Chaining
- [Dynamic Memory Allocation](#dynamic-memory-allocation)
- [Arrays](#arrays)
  - Static vs Dynamic
  - 2D Arrays
- [Strings](#strings)
- [Data Structures](#data-structures)
  - Structs (Composite)
  - Linked Lists (Dynamic)
- [Memory Allocators](#memory-allocators)

<details>
<summary><h2> Program Memory </h2></summary>

<br>

<!-- INSERT INFO HERE -->

</details>

<details>
<summary><h2> Pointers </h2></summary>

<br>

<!-- INSERT INFO HERE -->

</details>

<details>
<summary><h2> Dynamic Memory Allocation </h2></summary>

<br>

> Dynamic memory allocation refers to the process of allocating program memory manually through code

Dynamic memory allocation grants flexibility to programs that:
- Do not know the size of arrays or other data structures until runtime (e.g. size depends on user input)
- Need to allow for a variety of input sizes (not just fixed capacity)
- Want to allocate exactly the memory needed, avoiding wasted space
- Need to grow or shrink memory usage during program execution, reallocating space as needed and freeing it when no longer used

---

### Characteristics of Dynamically Allocated (Heap) Memory
- Dynamically allocated memory resides in the **heap** region of a program’s address space.
- When memory is allocated at runtime, the heap returns a **pointer** to the start of that memory block.
- Heap memory is **anonymous** — addresses are not tied to named variables.
  - For example, a local pointer on the stack can point to a block of memory in the heap.
- Heap memory provides global access to data as long as pointers to that data exist
- Heap memory must be **explicitly allocated and deallocated** by the programmer.
  - Failure to do so can result in **memory leaks**.
- After deallocating memory, it is always good practice to reset pointers referring to such memory to NULL.
    - This prevents future access to invalid memory which may contain garbage values.
- Dynamic memory can be used to allocate [arrays](#arrays) as well:
  - The returned pointer is the **base address** of the array.
  - You can use the memory just like a statically declared array.
  - Functions receiving dynamically or statically allocated arrays treat them the same.
    - By convention, **pointer syntax** is preferred for dynamically allocated arrays.

---

### Heap Memory Management
C provides `malloc()` and `free()` as the interface to manage heap memory:

- The **heap manager** maintains a **free list** — a set of unallocated memory extents.
- Each **extent** represents a contiguous chunk of free memory with a start address and size.
- Initially, all heap memory is free.
- **Repeated calls** to `malloc()` and `free()` may lead to **fragmentation** of heap memory.
- The manager uses the free list to:
  - Track available memory regions
  - Locate suitable contiguous blocks to fulfill future `malloc()` requests

#### Memory Metadata
- When you call `malloc()`, the heap manager also allocates a few bytes **before** the memory block to store **metadata**.
- This metadata includes information such as the **size** of the allocated block.
- This is why `free()` does **not** require the size of the memory — it retrieves it from the hidden header.

#### 🔑 Key Functions
| Function     | Description                                      |
|--------------|--------------------------------------------------|
| `malloc()`   | Allocates a block of memory                      |
| `calloc()`   | Allocates and zero-initializes memory            |
| `realloc()`  | Resizes previously allocated memory              |
| `free()`     | Frees allocated memory                           |

> ⚠️ Always `free()` any memory allocated with `malloc()` or `calloc()` to prevent memory leaks.
> ⚠️ Always reset pointers referring to deallocated memory to NULL to prevent accessing invalid data.

</details>

<details>
<summary><h2> Arrays </h2></summary>

<br>

> Arrays provide **contiguous storage** of elements of the **same data type**. They can be allocated either statically or dynamically depending on the needs of the program.

C supports arrays of multiple dimensions, however the most commonly used are 1D and 2D arrays.

---

### Static Arrays
- **Statically declared arrays** are allocated:
  - On the **stack** if declared as local variables.
  - In the **data segment** if declared as global or static variables.
- Their **capacity must be known at compile time**.

> Modern C implementations may optionally support variable length arrays (VLAs) which allow for the declaration of arrays whose size is determined at runtime, however VLAs are subject to some limitations (cannot be initialized, cannot be resized, cannot be members of some composite data types) and risks (stack overflow)
  ```c
  // Example
  <type> arr[n];

  // where n is determined at runtime
  ```

---

### Dynamic Arrays
- Allocated on the **heap** at runtime using `malloc`, `calloc`, or `realloc`.
- Can be **resized** by allocating new memory and copying elements.
- Used when:
  - The array size is unknown at compile time.
  - You need to support **variable-length input**.
  - Memory usage needs to be more efficient.

---

### Arrays in Functions
- In C, when passing an array to a function, the array **decays** to a pointer.
  - Only the **base address** of the array is passed.
  - The parameter and argument both refer to the **same memory**.
- This means:
  - **Modifications** to the array in the function will affect the original array.
  - Semantics are identical to **pointer-based** parameter passing.
- The size of an array must be passed too the function as an additional parameter if it is to be utilized.
    - There is no other way to retrieve the size of an array as the function only contains a pointer to the base address of the array's memory.
- When creating and returning an array locally within a function, it should be dynamically allocated
  - otherwise the base address returned would refer to invalid memory

---

### Arrays as Pointers
- Array variables are nothing more than pointers that refer to the base address of the contiguous storage
- Using array indexing syntax is equivalent to performing [pointer arithmetic](#pointer-arithmetic) and dereferencing the result
  ```c
  arr[i] <--> *(arr_ptr + i)

---

### 2D Arrays

#### Static 2D Arrays
- **Statically allocated 2D arrays** are stored in **row-major order**:
  - Each row is a 1D array stored **contiguously** in memory.
  - Rows are placed one after the other in memory.

#### Dynamic 2D Arrays
There are two common ways to dynamically allocate a 2D array:

#### 1. Single Block Allocation

- Allocate **one large contiguous block** of `N * M` elements.
- Use pointer arithmetic to map 2D indices to 1D memory:
  ```c
  arr[i * M + j]

> ⚠️ Syntactically, this is no different than allocation for a 1D array which prevents the compiler from being able to differentiate between the two; this is what causes the need for indexing logic

> **Advantages:**
> - Memory and cache efficient.
> - Entire matrix is stored contiguously.
> - Fast access due to spatial locality.

> **Disadvantages:**
> - Less intuitive syntax.
> - Requires manual indexing logic.

#### 2. Array of Row Pointers

- Allocate a 1D array of N pointers, where each pointer points to a 1D array of M values.
- This provides more programmer-friendly syntax, allowing for use of matrix-style notation (`arr[][]`).

> **Advantages:**
> - Natural, intuitive matrix syntax.
> - Easier to work with in code and debugging.

> **Disadvantages:**
> - Only the elements within each row are contiguous.
    > - Consecutive rows may not be adjacent in memory.
> - Less efficient in memory usage and access time due to fragmentation.

#### 2D Array Summary

| Feature                  | Static Array   | Dynamic Array (1 Block)  | Dynamic Array (Row Pointers)  |
|--------------------------|----------------|--------------------------|-------------------------------|
| Memory Location          | Stack/Data     | Heap                     | Heap                          |
| Size Known at Compile?   | ✅ Yes         | ❌ No                   | ❌ No                         |
| Contiguous in Memory     | ✅ Yes         | ✅ Yes                  | ❌ Rows only                  |
| `arr[i][j]` Syntax       | ✅ Yes         | ❌ No                   | ✅ Yes                        |
| Memory Efficient         | ✅ Yes         | ✅ Yes                  | ❌ No (fragmentation)         |

#### 2D Arrays in Functions
- Passing a 2D array to a function works the same as with 1D arrays:
  - The parameter receives the **base address** of the matrix.

- **Static:**
  - Number of columns must be specified in function parameter for array when using standard array notation (in addition to a row parameter).
    - Needed for compiler to determine where a row ends.
  - When using pointer notation (pointer-to-pointer), row and column parameters must be provided.

- **Dynamic:**
  - Single block allocation will result in passing a single pointer just like a 1D array, along with row and column parameters.
    - Pointer arithmetic will be required to properly map indices.
  - An array of row pointers will use standard pointer-to-pointer notation, with row and column parameters provided.

</details>
