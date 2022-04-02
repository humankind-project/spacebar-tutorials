---
title: Module 3
description: ""
date: 2022-04-02T22:33:01.363Z
author:
  - Brandon Sexton
name: module-3
oxa: oxa:zyBWlmnbnltDRPejFqwi/ck6W2oF7HSBXkkPwL038
---

# Module 3

+++ {"oxa":"oxa:zyBWlmnbnltDRPejFqwi/m6k3btmwy3SDrwimtIXI.1"}

Now we're ready to get into some basic space operations.  We'll start with the conversion of an ECI vector to classical orbital elements.  In this context, ECI is true Earth-Centered Inertial with the following axis definitions:

* x is toward the first point of aries
* y is the cross product of z and x
* z is aligned with the Earth's spin axis

Slight changes in the Earth's pole make the use of a true ECI frame a bit difficult, but we'll talk more about reference frames later.  For now, create spacebar>astro>orbit>elements.py

## Code

Now populate the elements.py file with the code below.

```python
from spacebar.astro.bodies import Earth
from spacebar.math.linalg import Vector3D
from math import atan2, sqrt, pi, sin, cos

class ClassicalElements:
    
    def __init__(
        self, sma:float, inc:float, ecc:float, raan:float, aop:float, ma:float
    ) -> None:
        """Basic constructor when elements are known

        Note:
            All lengths are measured in kilometers, and all angles are measured
            in radians

        Args:
            sma:    semi-major axis of orbit
            inc:    angle between momentum vector and z axis
            ecc:    eccentricity of the orbit 
            raan:   right ascension of ascending node
            aop:    argument of perigee
            ma:     mean anomaly

        Attributes:
            semi_major_axis:    the average radius relative to the central body
            inclination:        angle between momentum vector and z axis
            eccentricity:       measure where 0 == circular 1 == parabolic
            raan:               right ascension at which orbit becomes positive
            arg_of_perigee:     angle of raan vector and eccentricity vector
            mean_anomaly:       location angle mapped to circle

        Returns:
            None

        """
        self.semi_major_axis = sma
        self.inclination = inc
        self.eccentricity = ecc
        self.raan = raan
        self.arg_of_perigee = aop
        self.mean_anomaly = ma

    @classmethod
    def from_position_and_velocity(
        cls, pos:Vector3D, vel:Vector3D
    ) -> "ClassicalElements":
        """Constructor when ECI position and velocity are known.

        This method follows the procedures on pages 28-29 of Satellite Orbits
        by Oliver Monenbruck and Eberhard Gill

        Note:
            Values are in kilometers

        Args:
            pos:    Object containing x, y, and z location of satellite
            vel:    Object containing x, y, and z velocity of satellite

        Returns:
            ClassicalElements representation of input position and velocity
        """
        #Get the momentum vector from cross product of pos and vel (Eq. 2.56)
        h = pos.cross(vel)

        #Unitize the momentum vector (Eq. 2.57)
        w = h.normalize()

        #Get inc and raan (Eq. 2.58)
        inc = atan2(sqrt(w.x**2 + w.y**2), w.z)
        if inc == 0:
            raan = 0    #zero inc would create infinite raans without this
        else:
            raan = atan2(w.x, -w.y)

        #Correct for negative raan
        if raan < 0:
            raan+=2*pi

        #Solve for semi-latus rectum (Eq. 2.59)
        p = h.magnitude()**2/Earth.mu

        #Solve semi-major axis (Eq. 2.60)
        a = 1/(2/pos.magnitude() - vel.magnitude()**2/Earth.mu)

        #Solve mean motion (Eq. 2.61)
        n = sqrt(Earth.mu/a**3)

        #Solve eccentricity (Eq. 2.62)
        e = sqrt(1 - p/a)

        #Solve eccentric anomaly (Eq. 2.64)
        num = pos.dot(vel)/(a**2*n)
        den = 1 - pos.magnitude()/a
        ea = atan2(num, den)

        #Solve mean anomaly (Eq. 2.65)
        ma = ea - e*sin(ea)

        #Correct for negative mean anomaly
        if ma < 0:
            ma+=pi*2

        #Solve argument of latitude (Eq. 2.66)
        u = atan2(pos.z, -pos.x*w.y + pos.y*w.x)

        #Solve true anomaly (Eq. 2.67)
        ta = atan2(sqrt(1 - e**2)*sin(ea), cos(ea) - e)

        #Correct for negative true anomaly
        if ta < 0:
            ta+=pi*2

        #Solve argument of perigee (Eq. 2.68)
        aop = u - ta

        #Correct for negative argument of perigee
        if aop < 0:
            aop+=pi*2

        return ClassicalElements(a, inc, e, raan, aop, ma)
```

## Tests

Now we add the corresponding tests.

```python
import unittest
from math import degrees
from spacebar.astro.orbit.elements import ClassicalElements
from spacebar.math.linalg import Vector3D

class TestClassicalElements(unittest.TestCase):
    """
    Test class to validate ClassicalElements
    """

    def test_from_position_and_velocity(self):
        """
        Test to validate the calculation of classical elements given position
        and velocity.  This test data can be found on page 49 of Satellite
        Orbits by Oliver Montenbruck and Eberhard Gill
        """
        pos = Vector3D(10000, 40000, -5000)
        vel = Vector3D(-1.5, 1, -.1)
        coes = ClassicalElements.from_position_and_velocity(pos, vel)
        self.assertAlmostEqual(25015.181, coes.semi_major_axis, 3)
        self.assertAlmostEqual(.7079772, coes.eccentricity, 7)
        self.assertAlmostEqual(6.971, degrees(coes.inclination), 3)
        self.assertAlmostEqual(173.290, degrees(coes.raan), 3)
        self.assertAlmostEqual(91.553, degrees(coes.arg_of_perigee), 3)
        self.assertAlmostEqual(144.225, degrees(coes.mean_anomaly), 3)
```

