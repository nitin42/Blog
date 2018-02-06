# Understanding and implementing a garbage collector

![garbage-collector](https://cdn-images-1.medium.com/max/800/1*Gl5xl5_o6Q1GaJaphW2zBw.jpeg)

## Introduction

Garbage collection is the process of recycling the dynamically allocated memory. It is performed by a garbage collector whose job is to recycle the memory which will never be used again (no object references or no reachability). In high-level languages, like C, have memory management primitives, **malloc()** and **free()** which allocate the memory and release the allocated memory when it is not required anymore. But in JavaScript, memory allocation and deallocation is implicit.

## Writing a garbage collector

There are various different ways of writing a garbage collector but we will use the simplest and the first algorithm ever
invented by “John McCarthy” in 1958 as part of the implementation of [Lisp](http://www.memorymanagement.org/mmref/lang.html#term-lisp).

The garbage collection algorithms rely on the conception of references. According to the Mozilla Developer Network documentation,

> An object is said to reference another object if the former has an access to the latter (either implicitly or explicitly). For instance, a JavaScript object has a reference to its prototype (implicit reference) and to its properties values (explicit reference).

For the sake of simplicity, we will implement the basic concept of mark and sweep algorithm because the actual implementation involves a lot of optimisations.

## Mark and Sweep Algorithm

The algorithm says that -

> Starting at the root, traverse an entire object graph (objects are stored in heap) and mark the bits on it to 1 whenever we reach an object. After we’re done with the traversing, delete those objects whose bits are not marked to 1.

![](https://cdn-images-1.medium.com/max/800/1*D1ajA5CUwGSqwQKcLHCZ7w.png)

Now according to the algorithm, we’ve to start with the root. We’ll assume the root to be the first element of heap array. Garbage collector algorithms rely on notion of references or reachability, so to implement one we will need to follow all the reachable objects starting from the root.

![](https://cdn-images-1.medium.com/max/800/1*ojb6bnIOq4bdh7-5e-xwpA.png)

![](https://cdn-images-1.medium.com/max/800/1*urzMuOMNVYbOlP8gWGtAtg.png)

Let’s remove “C” reference from “A” and add another object “D”.

![](https://cdn-images-1.medium.com/max/800/1*RjddVEZjgzaHkPGtBMPBrQ.png)

![](https://cdn-images-1.medium.com/max/800/1*mbNR4uxs1YX-FRuZzVnSiQ.png)

Now let’s remove “B” reference from “A”.

![](https://cdn-images-1.medium.com/max/800/1*lVKJWT8bCjWzk5eXLSdOSQ.png)

It means that “D” still has a reference to “A” (via “B”) but it’s not reachable because “B” is not reachable anymore (remember the main notion is reachability and reference).

Let’s create a function that will traverse all the reachable objects starting from the root and will set the “__markBit__” bit on it to 1.

![](https://cdn-images-1.medium.com/max/800/1*0rgUu8rZ2vWg-FOUZEji8g.png)

Now let’s create a function for sweep to move all the unmarked or unreachable objects to the free list.

![](https://cdn-images-1.medium.com/max/800/1*0HGnnTu0nhxpitX41_XWQg.png)

We have defined the process for mark and sweep, so now we’ll create our garbage collector function.

![](https://cdn-images-1.medium.com/max/800/1*udnWrTIUW1HzPiI6tr4Pjw.png)

And we’re done! Now let’s create one last function to execute the algorithm.

![](https://cdn-images-1.medium.com/max/800/1*oiBy-fZql6LNa3RuM9nJlQ.png)

**Output**

![](https://cdn-images-1.medium.com/max/800/1*WVy6IjkgMw0HGzkudrQ5wQ.png)

Yeah! We did it 😎. Next time someone ask you to implement a garbage collector (mostly in interviews), you know how to do it 👍

This was possible only because of [dmitrysoshnikov](https://twitter.com/dmitrysoshnikov?lang=en) 👏👏 so don’t forget to follow him on Twitter. His [repository](https://github.com/DmitrySoshnikov/) is a gold mine if you love parsers, compilers and VM.

## Full source code

```js

let HEAP = [];

const A = {
  language: 'JavaScript'
};

HEAP.push(A);

const root = () => HEAP[0];

const B = {
  language: 'Rust'
};

HEAP.push(B);

A.B = B;

const C = {
  language: 'Elm'
};

HEAP.push(C);

A.C = C;

// Let's remove the reference C
delete A.C;

const D = {
  language: 'GoLang'
};

HEAP.push(D);

// Object "D" is reachable from "B" and is allocated the memory
B.D = D;

// "B" reference is removed from "A".
delete A.B;

// It means that "D" still has the reference to it from "B" but it's
// not reachable (because B is not reachable anymore)

// After these manipulations, the heap still contains four objects:
// [{ A }, { B }, { C }, { D }], but only the "A" object is reachable (root)

// Garbage collector (uses mark and sweep algorithm )
const gc = () => {
  // Set __mark__ bits on the reachable objects to 1
  mark();
  
  // Collect the garbage (objects with __mark__ bit not set to 1)
  sweep();
}

// Traverse all the reachable objects starting from the root and set the
// __mark__ bit on it to 1
const mark = () => {

  // Initially only the root is reachable
  let reachables = [ root() ];
  
  while (reachables.length) {
    // Get the next object
    let current = reachables.pop();
    // Mark the object if it is not already marked
    if (!current.__markBit__) {
      current.__markBit__ = 1;
      // add all the reachable objects from the current object
      // reachables array
      for (let i in current) {
        if (typeof current[i] === 'object') {
          // Add it to the reachables
          reachables.push(current[i]);
        }
      }
    }
  }
}


// Traverse the heap and move all unmarked or unreachable objects to the free list.
const sweep = () => {
  // Update the state
  HEAP = HEAP.filter((current) => {
    // For future Garbage collection cycles, reset the __markBit__ bit to 0
    if (current.__markBit__ === 1) {
      current.__markBit__ = 0;
      return true;
    } else return false; // move it to the free list
  });
}


const main = () => {
  console.log("\nHeap state before garbage collection: ", HEAP);

  // Make a call to garbage collector
  gc();

  console.log("\nHeap state after garbage collection: ", HEAP);
}

main();
```
