# Kinematics library
_Updated on July 22, 2021_

## Description

The Kinematics class calculates leg joint angles based on a provided cartesian foot position of a desired foot position.
An object should be created for each leg so that all legs can be positioned individually.

Important things to note:

- Calculations assume that provided cartesian coordinates are __relative to the quadruped's horizontal position__ 
    - If the quadruped is not horizontal i.e. tilted, you must take account for that and compute adjustments relative to it
- All units are in __millimeters__
- Negative Y-coordinates specify a position __towards__ the quadruped
- This class only supports legs with 3 degrees of freedom. It is assumed that each leg has 3 motors to control each joint. 

<br/>

<table style-"width:100%>
  <tr>
    <td>
    <figure>
    <img src="/images/footCoordinates.png" width="300" />
    <figcaption>Foot Coordinate Axis</figcaption>
    </figure>
    </td>
    <td>
    <figure>
    <img src="/images/legNumbering.png" width="300" />
    <figcaption>Leg Numbering</figcaption>
    </figure>
    </td>
  </tr>
</table>

## Functions

### ```Kinematics()```

``` cpp
Kinematics::Kinematics();
```

Constructor. Create an instance of a Kinematics class as such:

``` cpp
Kinematics LegKinematics();
```

### ```init()```

``` cpp
void Kinematics::init(LegID legID, int16_t inputX, int16_t inputY, int16_t inputZ, Motor legMotors[])
```

```legID``` - The leg that the Kinematics object is controlling. Should be of type [```LegID```](#LegID)
```inputX``` - The initial (startup) X position that the foot should go to.
```inputY``` - The initial (startup) Y position that the foot should go to.
```inputZ``` - The initial (startup) Z position that the foot should go to.
```legMotors[]``` - An array of length 12 (3 motors per leg, 4 legs) whose variables are of struct type [```Motor```](#Motor).

Note that inputX, inputY, and inputZ are integrated into the ```Quadruped``` and ```StepPlanner``` classes and serve as origin foot positions
there.


## Enums

### LegID

An enum to describe each leg. 
``` cpp
typedef enum {
  LEG_1 = 1, LEG_2, LEG_3, LEG_4
} LegID;
```

## Structs

### Motor

A struct that holds variables describing motor parameters.
``` cpp
typedef struct {
  uint8_t controlPin;

  // Angle/calculation stuff
  int16_t angleDegrees;
  int16_t previousDegrees;     // previous degrees since last call to updateDynamicPositions()
  int16_t dynamicDegrees;

  // Calibration
  uint16_t calibOffset;         // This is an offset for calibration (to keep the motor accurate)
  uint16_t maxPos;
  uint16_t minPos;
  uint16_t applicationOffset;   // This is an offset for putting the calculated angles in contex.
                                // It is likely that the zero positions of the motors isn't where
                                // calculations assumes it to be, so you need an offset to make 
                                // sure that the angle is correct relative to the motor's zero.
} Motor;
```