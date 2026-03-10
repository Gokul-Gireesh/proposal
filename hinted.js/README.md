# JS Memory Hints (hinted.js)

**Author:** [Gokul Gireesh](https://github.com/Gokul-Gireesh)  
**Status:** Draft Proposal

## The Vision
JavaScript is currently limited by its "guess-and-check" optimization strategy. Engines like V8 spend massive resources trying to guess the shapes of objects and the types of numbers. 

**hinted.js** is a proposal for a new, optional syntax that allows developers to provide **direct memory allocation hints**. This isn't TypeScript—it's not about "safety." It's about giving the engine the power to blindly optimize memory layout for raw speed, making JavaScript competitive in high-performance computing, AI, and systems programming.

---

## Core Syntax Examples

### 1. Base Type and Extension
Defining a base structure to be inherited.

```javascript
// yourType.js
type yourType: {
  id: int 4,
  active: bit
};

// Main file
import yourType from './yourType.js';

type myType: {
  man: int 1,       
  woman: str 3,     
  hen: [int 1, int 4, str], 
  cow: int [8],     
  horse: int 1 [8] [6], 
} extends yourType; // Inherits id and active into the memory map
```

### 2. Fixed-Width Primitives
```javascript
let g: bit = 1;      // Single bit allocation
let h: bit [8] = [0,0,0,0,0,1,0,1]; // 8 bits mapped to 1 byte
let l: uint 1 = 7;   // Unsigned 8-bit
let n: int 1 = 3;    // Signed 8-bit
```

### 3. Floating Point Resolution
```javascript
let r: 4.4 = 7.2; // 4-bit resolution before and 4-bit after '.'
let t: .8 = 1.3;  // 8-bit resolution after decimal, dynamic before
```

---

## The Rulebook (Inheritance & Extension)

To keep the memory model predictable for the engine, we follow these constraints:

1.  **Objects:** Can only extend other objects. They define a static "locked" shape.
2.  **Functions & Arrays:** Both are treated as specialized objects. They **can** extend an object `type` to gain fixed-size `.props` without losing their callable/iterable nature.
    *   *Example:* `type myFn: () extends yourType;` creates a function that also has a 4-byte `id` property.
3.  **Primitives:** Primitives cannot extend anything. They are the base units of storage.
4.  **Implicit Optimization:** 
    *   An empty object `{}` implies "any number of props" (standard JS).
    *   A typed object implies "blindly assume only these props exist."
5.  **Overflows:** 
    *   `int` overflows wrap (e.g., 128 becomes -127 in `int 1`).
    *   `str` overflows are truncated to the fixed size.

---

## Why this is better than "Types as Comments"
The current TC39 proposal for types suggests stripping syntax so it doesn't break. **This proposal suggests using that syntax to actually do something.** 

*   **Zero-Cost Abstractions:** The engine doesn't verify; it just allocates. If you lie to the engine, it's your funeral—but if you tell the truth, your code runs at near-C speeds.
*   **No More Boilerplate:** Replace hundreds of lines of `ArrayBuffer` and `DataView` with native, readable syntax.
*   **Computational Dominance:** Node.js can move beyond I/O and rule the computational world (AI, Video Processing, Real-time Physics).

## Implementation Note
Engines that do not support these hints can simply ignore the `:` annotations. While bit-level allocations might be stored as bytes internally by some engines, the engine **interfaces** the data as if it were the specified size, allowing developers to write memory-perfect code.

---
**Let's make JavaScript fast by default.**
