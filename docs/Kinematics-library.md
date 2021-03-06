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
- The library considers offsets to the y-axis i.e. if your leg lies _next_ to the center of rotation of motor 2. The center of rotation will
  be considered the top of the leg, or above the foot, regardless of the actual rotation position.

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

## Setup
All configuration parameters can be found in ```quadrupedConfig.h```. If you don't configure your quadruped correctly, the math won't work!

```#define LIMB_1``` - The length from the y-axis motor axis to the shoulder (above the foot).

```#define LIMB_2``` - The length of the upper leg (equivalent to a thigh on a human leg)

```#define LIMB_3``` - The length of the lower leg (equivalent to a calf on a human leg)

```#define SHOULDER_FOOT_MAX``` - This is a constraint for the maximum length from the shoulder directly to the foot.

```#define SHOULDER_FOOT_MIN``` - This is a constraint for the minimum length from the shoulder directly to the foot.

```#define MAX_SPEED_INVERSE``` - A scale factor for milliseconds per degrees of change from the current foot position to the final foot position. Only necessary
if you wish to use dynamic position updates. 

## Functions

### ```Kinematics()```

``` cpp
Kinematics::Kinematics();
```

Constructor. Create an instance of a Kinematics class as such:

``` cpp
Kinematics LegKinematics();
```

<hr/>

### ```init()```

``` cpp
void Kinematics::init(LegID legID, int16_t inputX, int16_t inputY, int16_t inputZ, Motor legMotors[])
```

<span style="color:rgb(255,20,147)">&rarr;</span>  Performs basic initialization of the library and writes the origin leg coordinates passed as arguments. Should be called in ```void setup()```.

```legID``` - The leg that the Kinematics object is controlling. Should be of type [```LegID```](#LegID)

```inputX``` - The initial (startup) X position that the foot should go to.

```inputY``` - The initial (startup) Y position that the foot should go to.

```inputZ``` - The initial (startup) Z position that the foot should go to.

```legMotors[]``` - An array of length 12 (3 motors per leg, 4 legs) whose variables are of struct type [```Motor```](#Motor).

Note that inputX, inputY, and inputZ are integrated into the ```Quadruped``` and ```StepPlanner``` classes and serve as origin foot positions
there.

<hr/>

### ```setFootEndpoint()```

``` cpp
void Kinematics::setFootEndpoint(int16_t inputX, int16_t inputY, int16_t inputZ)
```

<span style="color:rgb(255,20,147)">&rarr;</span> Allows you to set the endpoint of the foot in cartesian coordinates.

```inputX``` - The demand X coordinate of the foot endpoint.

```inputY``` - The demand Y coordinate of the foot endpoint.

```inputZ``` - The demand Z coordinate of the foot endpoint.

<hr/>

### ```updateDynamicFootPosition()```

``` cpp
void Kinematics::updateDynamicFootPosition()
```

<span style="color:rgb(255,20,147)">&rarr;</span> Updates the foot position for dynamic movement. After calling ```setFootEndpoint```, you may 
choose to update the foot position with a static calculation sequence (move the foot to the endpoint immediately) or a dynamic calculation sequence 
(interpolate the position of the foot to the endpoint and calculate it's position dynamically). This function updates the dynamic foot coordinate 
with respect to a time interval.

Change the time factor with the ```MAX_SPEED_INVERSE``` constant

<hr/>

## Constants

### ```MAX_SPEED_INVERSE```

A scale factor for milliseconds per degrees of change from the current foot position to the final foot position. Increasing this makes interpolation occur slower, 
decreasing it makes interpolation faster. Avoid going less than 2 (most hobby servo motors cannot handle any faster).
``` cpp
#define MAX_SPEED_INVERSE   20
```

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