---
title: Module 5
description: ""
date: 2022-04-02T23:05:38.819Z
author:
  - Brandon Sexton
name: module-5
oxa: oxa:zyBWlmnbnltDRPejFqwi/s88iwMATrC3zw4kGjRGZ
---

# Module 5

+++ {"oxa":"oxa:zyBWlmnbnltDRPejFqwi/0QOMbz5UQRfwXHR23G8o.2"}

I know the first few modules aren't terribly interesting, but hopefully they're at least somewhat practical.  At least now we can start a bit of satellite propagation.  To keep these modules short, I'm only going to include the code snippets that allow for a single solution of a future two-body state.  In future modules we'll include some frame transformations that make the outputs a little more meaningful.  For now, start by adding a couple of updates to existing scripts.  Your elements.py script will need these helper methods.

# Code

## Updates

```python
    def get_mean_motion(self) -> float:
        """get the average orbital rate

        Follows equation 2.35 from Satellite Orbits to calculate the mean
        motion of the orbit

        Args:
            None

        Returns:
            mean orbital rate in radians per second

        """
        return sqrt(Earth.mu/self.semi_major_axis**3)

    def get_perigee_vector(self) -> Vector3D:
        """get vector pointing from central body to satellite perigee

        Follows equation 2.52 from Satellite Orbits 

        Args:
            None

        Returns:
            unit vector pointing to perigee

        """
        cw = cos(self.arg_of_perigee)
        cO = cos(self.raan)
        sw = sin(self.arg_of_perigee)
        sO = sin(self.raan)
        ci = cos(self.inclination)

        x = cw*cO - sw*ci*sO
        y = cw*sO + sw*ci*cO
        z = sw*sin(self.inclination)

        return Vector3D(x, y, z).normalize()

    def get_semi_latis_rectum_vector(self) -> Vector3D:
        """get vector pointing to true anomaly of 90 degrees

        Follows equation 2.53 from satellite orbits
        
        Args:
            None

        Returns:
            unit vector pointing to satellite location equal to a true anomaly
            of 90 degrees

        """
        cw = cos(self.arg_of_perigee)
        cO = cos(self.raan)
        sw = sin(self.arg_of_perigee)
        sO = sin(self.raan)
        ci = cos(self.inclination)

        x = -sw*cO - cw*ci*sO
        y = -sw*sO + cw*ci*cO
        z = cw*sin(self.inclination)

        return Vector3D(x, y, z).normalize()

    @staticmethod
    def equation_to_eccentric_anomaly(mean_anom:float, ecc:float) -> float:
        """Solve eccentric anomaly given eccentricity and mean anomaly 
        
        Follows the method found on page 24 of Satellite Orbits.

        Note:
            All angles are in radians

        Args:
            mean_anom:      mean anomaly of orbit
            ecc:            eccentricity of orbit

        Returns:
            value of eccentric anomaly
            
        """

        #Seed E sub i with mean anomaly
        ea_0 = mean_anom
        converged = False

        #For high eccentricities, seed E sub i with pi
        if ecc > .8:
            ea_0 = pi

        #Iterate until meeting tolerance
        while not converged:

            #Save the numerator and denominator separately to avoid long lines
            num = mean_anom - ea_0 + ecc*sin(ea_0)
            den = 1 - ecc*cos(ea_0)

            #Solve E sub i+1
            ea_n = ea_0 + num/den

            #Check tolerance and reseed E sub i if difference is too large
            if(abs(ea_n-ea_0) < 1e-12):
                converged = True
            else:
                ea_0 = ea_n

        #Correct for negative values
        if ea_n < 0:
            ea_n+=pi*2.0

        return ea_n
```

The first few methods will help provide some access to common vectors used in the propagation process.  The last method provides an iterative solution to Kepler's problem.  This will also be necessary in our propagation method.  Until I get feedback regarding my lack of detail, I'm going to continue to skip the explanation of the methods and focus on code examples.  If you do want additional detail, I'm happy to incorporate some supplementary explanations.  I can also direct you to some literature that may help.  Moving on...

## New

Now we should have everything we need to start a propagator.  Create a new path along spacebar>astro>propagators>inertial.py

Add these lines to your newly created script:

```python
from copy import deepcopy
from math import cos, sin, sqrt

from spacebar.astro.bodies import Earth
from spacebar.time.utc import UTC
from spacebar.math.linalg import Vector3D
from spacebar.astro.orbit.elements import ClassicalElements

class TwoBody:

    def __init__(self, epoch:UTC, pos:Vector3D, vel:Vector3D) -> None:
        """Class used to model basic propagation
        
        Note:
            Units are in kilometers and kilometers per second

        Args:
            epoch:      time of initial state validity
            pos:        ECI position of the satellite at epoch
            vel:        ECI velocity of the satellite at epoch
            
        Returns:
            None
            
        """
        self.epoch0 = deepcopy(epoch)
        self.position0 = deepcopy(pos)
        self.velocity0 = deepcopy(vel)

    def get_state_at_epoch(
        self, next_epoch:UTC
    ) -> tuple(UTC, Vector3D, Vector3D):
        """get future state of model
        
        Args:
            next_epoch:     desired time of next state
            
        Returns:
            new epoch, position, and velocity 
            """

        #Get the time difference in seconds
        t = next_epoch.timestamp - self.epoch0.timestamp

        #Save the COEs of the given state vector
        coes = ClassicalElements.from_position_and_velocity(
            self.position0, 
            self.velocity0
        )

        #Get mean anomaly after delta t (Equation 2.37)
        ma = coes.mean_anomaly + coes.get_mean_motion()*t

        #Get P and Q vectors
        p = coes.get_perigee_vector()
        q = coes.get_semi_latis_rectum_vector()

        #Solve eccentric anomaly
        e = coes.eccentricity
        en = ClassicalElements.equation_to_eccentric_anomaly(ma, e)

        #Solve position using equation 2.43
        a = coes.semi_major_axis
        pScaled = p.scale(a*(cos(en) - e))
        qScaled = q.scale(a*sqrt(1-e*e)*sin(en))
        pos = pScaled.plus(qScaled)

        #Solve velocity using equation 2.44
        pScaled = p.scale(-sin(en))
        qScaled = q.scale(sqrt(1-e*e)*cos(en))
        multiple = sqrt(Earth.mu*a)/self.position0.magnitude()
        vel = pScaled.plus(qScaled).scale(multiple)

        return next_epoch, pos, vel
```

At this point our tests will start to get a little muddy.  There are some free resources available to validate our work against, but I'm not all that interested in introducing more confusion than what is necessary at this point.  Instead, we'll use visual inspection in the next module to confirm our work.  This is not a best-practice for unit-testing, but it's certainly more fun.  

