---
title: Module 4
description: ""
date: 2022-04-02T22:54:03.870Z
author:
  - Brandon Sexton
name: module-4
oxa: oxa:zyBWlmnbnltDRPejFqwi/nmk5E2ZF7MCREPxlqaY5
---

# Module 4

+++ {"oxa":"oxa:zyBWlmnbnltDRPejFqwi/2mYbFJi0eGjirJzj3DG3.1"}

To make this less boring for the near term, we're only going to make one time object.  Our reference system is going to be Universal Coordinated Time (UTC - no I didn't mix up the letters). Just keep in mind that eventually we will introduce plenty of other time systems to create sufficient confusion.  For now, make a file on the path spacebar>time>utc.py - this should be a new directory for you so don't forget to include \__init_\_.py

```python
from datetime import datetime, timezone

class UTC:
    
    STRING_FORMAT = "%b %d %Y %H:%M:%S.%f"  # Mmm DD YYYY hh:mm:ss.ssssss
    
    def __init__(self, epoch_string:str) -> None:
        """Class to represent calendar UTC epochs.

        Args:
            epoch_string:   string representing calendar date

        Attributes:
            timestamp:      posix time of calendar date

        Returns:
            None

        """
        local_datetime = datetime.strptime(epoch_string, self.STRING_FORMAT)
        utc_datetime = local_datetime.replace(tzinfo=timezone.utc)
        self.timestamp = utc_datetime.timestamp()

    def plus_seconds(self, seconds_to_add:float) -> "UTC":
        """get new UTC of self plus variable seconds
        
        Args:
            seconds_to_add:     the number of seconds to be added to self
            
        Returns:
            new UTC representing self plus seconds_to_add

        """
        utc_dt = datetime.utcfromtimestamp(self.timestamp + seconds_to_add)
        utc_string = utc_dt.strftime(self.STRING_FORMAT)
        return UTC(utc_string)

    def to_string(self) -> str:
        """get the calendar format of the UTC object
        
        Args:
            None
            
        Returns:
            string in Mmm DD YYYY hh:mm:ss.ssssss format
            
        """
        utc_datetime = datetime.utcfromtimestamp(self.timestamp)
        return utc_datetime.strftime(self.STRING_FORMAT)
```

The code here is pretty simple, and there are fewer lines than our previous modules.  That's an attempt to keep these modules nice and digestible.  Plus time objects are pretty boring, and I'm trying to get to applications that are more fun.  Having said that, this should be the bare minimum time module to move us into some spacecraft propagation.  Finish up by adding a test module.  Then, look into [Module 5](oxa:zyBWlmnbnltDRPejFqwi/s88iwMATrC3zw4kGjRGZ "Module 5") to start viewing a spacecraft in orbit.

```python
import unittest

from spacebar.time.utc import UTC

class TestVector3D(unittest.TestCase):

    EPOCH_AS_STRING = "Mar 04 2022 04:42:42.000"

    def test_utc(self):
        """
        Test to UTC object creation and timestamp conversion
        """
        utc = UTC(self.EPOCH_AS_STRING)
        self.assertAlmostEqual(1646368962, utc.timestamp, 7)

    def test_plus_seconds(self):
        """
        Test the ability to time to a UTC object
        """
        utc = UTC(self.EPOCH_AS_STRING)
        utc2 = utc.plus_seconds(34)
        self.assertAlmostEqual(1646368996, utc2.timestamp, 7)
        self.assertEqual("Mar 04 2022 04:43:16.000000", utc2.to_string())
```

