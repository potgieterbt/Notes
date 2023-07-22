Concrete:

A class that has no virtual functions and can be instantiated. Can be used as a blueprint for objects or child classes.

Abstract:

A class that is used as a general concept and is used as a base for concrete classes. Serve as blueprint for all classes inherited from it. Provides structure and behaviour for subclasses and cannot be instantiated.

Containers:

A generic collection of class templates and algorithms that allow implementation of common data structures.

Copy:

A normal copy is done with a pointer so that the new variable points to the same address as the original variable, meaning that if the new variable changes the value the original value also changes.

A copy of a container without using pointers can be done by looping through the original variable and copying the value of each element into the new variable.

Move:

Moving a container is when the new variable points to the address that the original variable is pointing at and then the original variable gets deassigned. A return is a move as the variable returned gets deassigned as the value is assigned to the variable the it is returned to.

Suppressing operations:

For a container because the default copy and move use pointers can be disastrous as the compiler does not know how to copy or move each part of a container. To remove the possibility of this can delete the operations by putting =delete; for operations (type&) and (type&&).