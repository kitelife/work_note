<A tour of Go>学习笔记
=========================

The new function
^^^^^^^^^^^^^^^^^^

The expression ``new(T)`` allocates a zeroed T value and returns a pointer to it.

``var t *T = new(T) or t := new(T)``

Arrays
^^^^^^^^^^

The type ``[n]T`` is an array of n values of type T.
An array's length is part of its type, so arrays cannot be resized.

Slices
^^^^^^^^^

A slice points to an array of values and also includes a length.

``[]T`` is a slice with elements of type T.

Slices can be re-sliced, creating a new slice value that points to the same array.

The zero value of a slice is nil.
A nil slice has a length and capacity of 0.

Maps
^^^^^^^^

Test that a key is present with a two-value assignment:

``elem, ok = m[key]``

If key is in m, ok is true. If not, ok is false and elem is the zero value for the map's element type.
Similarly, when reading from a map if the key is not present the result is the zero value for the map's element type.

Function closures
^^^^^^^^^^^^^^^^^^^^

Go functions may be closures. A closure is a function value that references variables from outside its body. The function may access and assign to the referenced variables; in this sense the function is "bound" to the variables.

Methods with pointer receivers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Methods can be associated with a named type or a pointer to a named type.

There are two reasons to use a pointer receiver. First, to avoid copying the value on each method call(more efficient if the value type is a large struct). Second, so that the method can modify the value its receiver points to.

Goroutines
^^^^^^^^^^^^^

Goroutines run in the same address space, so access to shared memory must be synchronized. The sync package provides useful primitives, although you won't need them much in Go as there are other primitives.

Range and Close
^^^^^^^^^^^^^^^^^^^

A sender can close a channel to indicate that no more values will be sent. Receivers can test whether a channel has been closed by assigning a second parameter to the receiver expression. after

``v, ok := <-ch``

ok is false if there are no more values to receive and the channel is closed.
The loop ``for i := range c`` receives values from the channel repeatedly util it is closed.
Only the sender should close a chanel, never the receiver. Sending on a closed channel will cause a panic.
Channels aren't like files; you don't usually need to close them. Closing is only necessary when the receiver must be told there are no more values coming, such as to terminate a range loop.
