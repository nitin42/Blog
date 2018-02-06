## Programming concepts

Given below are some of the concepts in programming that may help you in understanding of the programs and the different processes and how they evolves.

### Compound procedures

Defining procedures in a language is a powerful technique for formulating an abstraction by which a compound action can be named and
evaluated as a unit. By using compound procedures, we can define and express our own ideas inside the language where a language serves as a framework
to organise our ideas within. For example - we can express an idea of defining cube of a number.

```js
function cube(n) {
  return n*n*n;
}
```

Here the name `cube` gives an identity to our procedure, parameter `n` serves as a formal parameter which we can use inside the body
of the function to yield the value of procedure.

**Using compound procedures as if they were built into the language**

We can use the compound procedures to define the other procedures. For example - defining a procedure for `sum of the cube` of two numbers.

```js
function sum_of_cube(a, b) {
  return cube(a) + cube(b);
}
```

Now looking at the code for `sum_of_cube` function, we may think that the `cube` was baked into the language. So compound procedures
are very helpful in expressing our ideas and using them as if they were language primitive.
