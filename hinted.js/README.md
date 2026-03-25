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
  active: bit,
  clown: (),
  bat: int 4 []
};

// Main file
import yourType from './yourType.js';

type myType: {
  man: int 1,       
  woman: str 3,     
  hen: [int 1, int 4, str], 
  cow: int [8],     
  horse: int 1 [8] [6], 
} extends yourType; // Inherits id and active, etc into the memory map
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
let v: . = 4.3; // dynamic float, standard js optimization, but it'll never be strings!
```

### 4. Function Declarations
```javascript
function fn(a: int 4, b: int 4) {
  // rest of the code
}

let myfn: () = (d: int 4) => {};
```

### 5. Using Pre-defined types
```javascript
let x: type myType = {
  man: 5,
  woman: 'eva',
  cow: [2,3,4,0,0,0,7,8]
}
```

---

## The Rulebook (Inheritance & Extension)

To keep the memory model predictable for the engine, we follow these constraints:

1.  **Objects:** Can only extend other objects. They define a static "locked" shape.
2.  **Functions & Arrays:** Both are treated as specialized objects. They **can** extend an object `type` to gain fixed-size `.props` without losing their callable/iterable nature.
    *   *Example:* `type myFn: () extends yourType;` creates a function that also can have a 4-byte `id` property.
3.  **Primitives:** Primitives cannot extend anything. They are the base units of storage.
4.  **Implicit Optimization:** 
    *   An empty object `{}` implies "any number of props" (standard JS).
    *   A typed object implies "blindly assume only these props exist."
    *   An empty array also implies any number of elements
    *   An array and function implies it doesn't have any props, unless its extends to a type object of defined size
5.  **Overflows:** 
    *   `int` overflows wrap (e.g., 128 becomes -127 in `int 1`).
    *   `str` overflows are truncated to the fixed size.
    *   `{prop}` hints are implicitly Memory-Locked. Assignments to undeclared properties are silently ignored to preserve the static memory offset. This ensures that property access remains a simple pointer-addition operation rather than a dynamic lookup.

> ## The Case for Hardware Trapping (Performance Note)
> While this proposal defines predictable behaviors like **wrapping** and **truncation**, for maximum performance, engines should implement **Hardware-Level Trapping**:
> 
> *   **Zero-Check Execution:** By allowing the engine to **Trap** (hard crash) on a type violation, we eliminate the "Safety Tax." The engine no longer writes `if (type)` checks in the machine code; it generates a single CPU instruction and trusts the hardware. 
> *   **Wasm Parity:** WebAssembly is allowed to Trap to gain near-native speeds. JavaScript should not be "babysat" by the engine when a developer explicitly opts into a high-performance hint.
> *   **Hardware Efficiency:** By Trapping, a "broken promise" results in an immediate hardware interrupt. This is the **fastest possible way** to handle errors, as it requires zero software logic during the "happy path" of execution.

---

## Why this is better than "Types as Comments"
The current TC39 proposal for types suggests stripping syntax so it doesn't break. **This proposal suggests using that syntax to actually do something.** 

*   **Zero-Cost Abstractions:** The engine doesn't verify; it just allocates. If you lie to the engine, it's your funeral—but if you tell the truth, your code runs at near-C speeds.
*   **No More Boilerplate:** Replace hundreds of lines of `ArrayBuffer` and `DataView` with native, readable syntax.
*   **Computational Dominance:** Node.js can move beyond I/O and rule the computational world (AI, Video Processing, Real-time Physics).

## Implementation Note
Engines that do not support these hints can simply ignore the `:` annotations. While bit-level allocations might be stored as bytes internally by some engines, the engine **interfaces** the data as if it were the specified size, allowing developers to write memory-perfect code.

The proposed hinted.js utilizes "Speculative Metadata," where type hints are promises regarding usage rather than strict constraints, enabling JIT engines to generate "zero-check" machine code and optimize storage layouts. The proposal mandates that broken promises result in predefined behaviors—specifically integer wrapping and string truncation—mirroring high-performance TypedArrays

By using str N, the developer guarantees that the value will never exceed N characters. The engine may choose to allocate a fixed-size buffer of N * 4 bytes to allow in-place mutation, eliminating the garbage collection overhead of creating new string objects during high-frequency updates.

Once a variable is declared with a hint (e.g., let score: uint 2), that hint is locked for the lifetime of that scope. The engine can generate optimized machine code for that specific memory layout immediately, knowing the developer will never 'change their mind' about the type.

The hints are exclusively applied at the point of variable declaration and function argument definition. This is not a system for type safety or return-type tracking; rather, it informs the engine of the intended storage format for the variable itself. If a function returns a value, the engine optimizes based on the hinted type of the variable receiving that value, applying the promised overflow or truncation rules (e.g., int wrapping) upon assignment.

```JavaScript
// Example of your "Receiver" logic:
function calculate() { return 500; } // Returns standard JS number

let result: int 1 = calculate(); 
// The engine doesn't care what calculate() returns. 
// It only cares that 'result' is an 'int 1'.
// The value 500 wraps to -12 because the 'bucket' is fixed.
```

> hinted.js is an add-on to the language rather than a mode switch. It can be defined on some variables, all, or nothing. It should exist alongside JS, not act as a fork of JS.

> hinted.js is not meant to replace Wasm or TypeScript. It ensures neither type safety nor actual bit-level allocations; instead, it allows the JS engine to evaluate and run code more efficiently and effectively. It eliminates the overhead of jumping between JS and Wasm for simple calculations, where the context-switching cost often results in diminished returns compared to native execution speeds.

## Additional Proposal: Declaration Keywords for Arguments
To maximize the performance gains of hinted.js, this proposal suggests (and is designed to integrate with) the use of let and const keywords within function argument lists.

While : type hints the storage format, let/const hints the access pattern. Together, they allow the engine to treat a JavaScript function exactly like a compiled C function.
```javascript
// Current proposal:
function move(x: int 4, y: int 4) { ... } 

// Additional proposal:
function move(const x: int 4, let y: int 4) { ... }
```

Immutable Constraints (const): When an argument is hinted as const, the engine knows the value will never change. It can inline the value directly into the machine code, bypassing the stack entirely.

Explicit Mutability (let): Marking an argument as let signals to the engine that this variable is a high-speed local register. The engine can "recycle" that register for the duration of the function, avoiding expensive memory lookups.

Engine Certainty: By combining a Fixed Storage Hint (: int 4) with a Fixed Access Hint (const), the engine achieves Zero-Speculation. It doesn't have to guess what the value is, what the type is, or if it will change.

---
**Let's make JavaScript fast by default.**
