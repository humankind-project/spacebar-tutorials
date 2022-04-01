---
title: Module 2
description: Vector Operations
date: 2022-04-01T23:42:48.285Z
author:
  - Brandon Sexton
name: module-2
oxa: oxa:zyBWlmnbnltDRPejFqwi/Z5OC1a3Z4cqlzVa3kyn2
---

# Module 2

+++ {"oxa":"oxa:zyBWlmnbnltDRPejFqwi/xlRNMSxS0ygjpnHtVXhU.1"}

Module 2 focuses on the development of vector operations within our script. The majority of the calculations performed in future astrodynamics modules will require vector math, so it is important to build that foundational library early.

# Create Paths

Start by creating a path within your package that creates an intuitive trail for the user. This will be a script full of linear algebra functions used for the entire package, so let’s use spacebar > math > linalg.py to get a folder structure similar to the list seen below.

* spacebar
  * spacebar
    * math
      * linalg.py
      * \__init_\_.py
    * astro
    * tests

# Code

## Class Definitions

Before showing the entire code block, take a look at the constructor class for a Vector3D object. This class has a new method that was omitted when we created the Earth object in [Module 1](oxa:zyBWlmnbnltDRPejFqwi/pbXeJXnaITQbVuUlGull "Module 1"). We intentionally bypassed the constructor for Earth because we only want to use one Earth in our future modules. Vectors, though, will need a method to create new instances. Thus, you see the \__init_\_ method to construct new Vector3D objects. This will be a required method for any classes we create that are to be used in more than one instance.

```python
from math import sqrt

class Vector3D:

    def __init__(self, x:float, y:float, z:float) -> None:
        """three-dimensional vector used for basic linear algebra

        Args:
            x:      The first component of the vector
            y:      The second component of the vector
            z:      The third component of the vector

        Returns:
            None

        """
        self.x = x
        self.y = y
        self.z = z
```

## The “self” Argument

You might find it weird that we are required to pass “self” as an argument to this method and future methods you see. This is just a way of informing Python that this is a non-static method. Static vs non-static methods are represented in the code block below. Note that this block is purely for demonstration of static methods and is not required for our package

```python
class Motorcycle:
  
  	WHEEL_COUNT = 2 # This is all caps to indicate it is a constant
    
    def __init__(self, top_speed:float) -> None:
        self.top_speed = top_speed
        
    def get_top_speed(self): # Notice the use of self
        """This is a non-static method because you MUST create an instance of motorcycle
        to know what the top speed is
        """
        return self.top_speed
      
    @staticmethod
    def get_wheel_count(): # Notice the removal of self
        """This is a static method because you can call it without defining a new 
        motorcycle.  We can define it as a static method because it pertains to ALL motorcycles
        instead of a single instance.
        """
        return self.WHEEL_COUNT
```

## Type-hinting

Another thing you likely noticed is the colon pairing variable definition with object types. This is called type-hinting. It allows future developers and IDEs to easily identify what types of objects to pass and receive when using your methods.

```python
def __init__(self, x:float, y:float, z:float) -> None:
```

In the example above, x:float means x must be a float while the -> None says the method returns nothing when it is constructed.

## Formatting

I won’t go into too much detail regarding the formatting of the comments within the code, but you should know that it is important to choose a standard method of documenting your code. To follow the guidelines in this tutorial, you can research [google formatting for python](https://www.sphinx-doc.org/en/master/usage/extensions/example_google.html).

## Remaining Methods

Now that we’ve discussed the added content for this module, let’s look at the full script.

```python
from math import sqrt

class Vector3D:

    def __init__(self, x:float, y:float, z:float) -> None:
        """three-dimensional vector used for basic linear algebra

        Args:
            x:      The first component of the vector
            y:      The second component of the vector
            z:      The third component of the vector

        Returns:
            None

        """
        self.x = x
        self.y = y
        self.z = z

    def plus(self, vec_to_add:"Vector3D") -> "Vector3D":
        """Performs vector addition

        Args:       
            vec_to_add:     The vector which will be added to the calling vector

        Returns:
            AVector representing element addition of self and vec_to_add

        """
        return Vector3D(
            self.x + vec_to_add.x, 
            self.y + vec_to_add.y,
            self.z + vec_to_add.z
        )

    def minus(self, vec_to_subtract:"Vector3D") -> "Vector3D":
        """Performs vector subtraction

        Args:
            vec_to_subtract:    The vector which will be subtracted from self

        Returns:
            Vector representing element difference of self and vec_to_subtract

        """
        return Vector3D(
            self.x - vec_to_subtract.x, 
            self.y - vec_to_subtract.y,
            self.z - vec_to_subtract.z
        )

    def dot(self, vec_to_dot:"Vector3D") -> float:
        """Performs dot product

        Args:
            vec_to_dot:     The vector to be used in the product with self

        Returns:
            scalar value representing sum of element multiplication

        """
        return self.x*vec_to_dot.x + self.y*vec_to_dot.y + self.z*vec_to_dot.z

    def cross(self, vec_to_cross:"Vector3D") -> "Vector3D":
        """Performs cross product

        Args:
            vec_to_cross:   The vector to be used in the product with self

        Returns:
            Vector perpendicular to self and vec_to_cross 

        """
        return Vector3D(
            self.y*vec_to_cross.z - self.z*vec_to_cross.y,
            self.z*vec_to_cross.x - self.x*vec_to_cross.z,
            self.x*vec_to_cross.y - self.y*vec_to_cross.x
        )

    def normalize(self) -> "Vector3D":
        """Method used to get the unit vector for the calling vector

        Args:
            None

        Returns:
            Vector parallel to self and of length 1

        """
        mag = self.magnitude()
        return Vector3D(self.x/mag, self.y/mag, self.z/mag)
        

    def magnitude(self) -> float:
        """Method used to get the length of the calling vector

        Args:
            None

        Returns:
            Square root of the sum of squared components

        """
        return sqrt(self.x**2 + self.y**2 + self.z**2)

    def scale(self, multiple:float) -> "Vector3D":
        """Method used to scale each element by the specified multiple

        Args:
            multiple:   value to be multiplied across elements

        Returns:
            Vector parallel to self with a scaled magnitude 
        """
        return Vector3D(self.x*multiple, self.y*multiple, self.z*multiple)
```

