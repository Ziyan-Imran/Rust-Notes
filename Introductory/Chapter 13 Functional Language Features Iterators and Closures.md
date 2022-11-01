Rust also includes aspects popular from *functional programming*. 
- Includes using functions as values by passing them in as arguments
- Returning functions from other functions
- Assigning functions to variables for later executions

This chapter will focus on covering certain aspects that are similar to features in other languages

The topics include:

- #closures : a function-like construct you can store in a variable
- #iterators: a way of processing a series of elements
- How to use closures and iterators to improve the I/O project from Chapter 12
- The performance of closures and iterators

# Closures: Anonymous Functions that Capture Their Environment

Rust closures:
- anonymous function you can save in a variable or pass as arguments to other functions
- create the closure in one place and then call the closure elsewhere to evaluate it in a different context
- Unlike functions, closures can capture values from the scope in which they're defined


## Capturing the Environment with Closures

