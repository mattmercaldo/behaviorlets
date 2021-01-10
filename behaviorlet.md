
# BEHAVIORLET: an application of beetree

This paper will discuss an overview and implementation of behaviorlets as a beetree application.


## What are behaviorlets?

To discuss behaviorlet we must agree on a defintion for robotic behavior.

A robotic behavior processes sensory input and coordinates appropriate motion control to achieve a specific goal.

A behaviorlet is a self contained and independent autonomous robotic behavior. 

As input the behaviorlet recieves all necessary sensory input to accomplish its goal.

As output the behaviorlet coordinates and authors control of a specific set of motion profiles.

There is a specific goal of the behaviorlet as pertains to intent, purpose and action.

A simple behaviorlet could be to make a robot move forward 1 meter.  The goal would be to move forward 1 meter from the current pose (location and heading).
The sensors necessry to achive this are some form of odometry or localization that is fed to a current pose value. 
The action could be a velocity command presented by a motion profile (Velocity, Acceleration, Angular Velocity) to an appropriate motion primitive.
The goal would be achieved when the current pose is "near enough" to the target pose.

An integral component of a behaivorlet is a dialog between a strategy and the behaviorlet.

A strategy in this discussion is that software component necessary to strategize the completion of some overall task.
A strategy uses behaviorlets to accomplish its overall task.

The dialog between a strategy and behaviorlet has the following aspects:
- behaviorlet selection
- behaviorlet configuration
- behaviorlet monitoring/health/pose
- behaviorlet pose tune
- behaviorlet interruption
- behaviorlet completion
- behaviorlet transition

In turn the dialog between behaviorlet and strategy has the following aspects:
- strategy monitoring/health/pose

Behaviorlets can be "cued" to start when a running behaviorlet is completed.
This concept is similar in kind to that of video splicing. 

A behaviorlet has points in its life where it can gracefully transition to another behaviorlet.
These are considered "out points". Just as with Video, there are timing considerations associated with the transition from one behaviorlet to another.
  
This has to do with an agreement of pose over time. 

The transition from one behaviorlet to another has to take into account the assumptions of the incoming behaviorlet and current motion profile of the running behaviorlet.
Based on these two, a "bridge" behaviorlet may need to be created on the fly so as to assure smooth transition between the two behaviorlets.


### THOUGHT SECTION

A behaviorlet is commanded in relative (robot) coordinates.

Strategy is sensative to the behaviorlet or robot pose and will account for error accumulation in configuration settings.

Internal to the behaviorlet operation, the strategy can adjust the behaviorlet pose so as to account for error accumulation.

primitive behaviorlets are the building blocks that a strategy can use to support complex behaviors

compound behaviorlets are complex behaviors that can be called such as rank and wall follow.

application is simply a dispather at the mission level that can call into any of the behaviorlets.

#### on pose

When a behaviorlet starts it is passed a pose from the startegy thru the configuration packet.
This pose is used as the pose of origin.
The steps are to null the local pose (encoders etc), then push the strategy pose into the behaviorlet's "global" pose.
By this means, the behaivorlet will "track with"  the strategy's pose and the strategy can update (tune) the pose
while the behviorlet is running.

## alphabet of behaviorlets

In this section we describe the alphabet or the basic behaviorlets necesary to support any strategy in it's overall success.

### line - primitive 

The **line** behaviorlet will drive in a forward or reverse direction in a line for a specified distance.

Bump, cliff, and wheel drop reflexes terminate this behaviorlet.

Update monitoring return current pose.

Pose tune adjusts heading.

This behaviorlet can be interrupted and return final pose in the monitor path.

This behaviorlet ends in a stop.

This behaviorlet can end at velocity and pose for transition to another behaviorlet based on end configuration.

### turn in place TIP - primitive

The **TIP** behaviorlet will turn in place given a configured direction and relative target heading (-pi to pi).

Bump, cliff, and wheel drop reflexes terminate this behaviorlet.

Update monitoring return current pose.

Pose tune adjusts heading.

This behaviorlet can be interrupted and return final pose in the monitor path.

This behaviorlet ends in a stop.

### pause - primitive

The **pause** behaviorlet stops the robot for a prescribed period of time.
Configuration is the amount of time to stop the robot.

### stop - primitive

The **stop** behaviorlet stops the robot.

### servo - primitive

The **servo** behaviorlet will facilitate a servo motion or turn based on configuration which includes a mixture of velocity and angular velocity 
as well as direction (positive or negative). It can also run forward or reverse.

Bump, cliff, and wheel drop reflexes terminate this behaviorlet.

Update monitoring return current pose.

Pose tune adjusts heading as well as velocity and angular velocity on the fly.

This behaviorlet can be interrupted and return final pose in the monitor path.

This behaviorlet ends in a stop.

This behaviorlet can end at velocity for connection to another servo or the servo can be manipulated from strategy to follow some sensor or other heading indicator.

### ranking - compound

The **ranking** behaviorlet is used to cover an area in an intelligent manner.  This behaviorlet can be configured to complete when a specified area is covered. 
The configuration includes rank width (default is robot width), rank length (default is xx meters), rank turn angles (default is 90 degrees), criss cross.

Bump or cliff is used to end a rank when in the rank (not turn on rank) portion of the behaviorlet is running. 
If configured for criss cross, when turn on rank's inter rank motion forward detects a bump (?) or cliff, then the criss cross is initiated.
The robot will allow the reflex to push away from the bump/cliff and continue the next rank. Upon completion the criss cross will commence with a renegrade turn.

Wheeldrop stops this behaviorlet

Update monitoring return current pose.

Pose tune adjusts heading.

This behaviorlet can be interrupted and return final pose in the monitor path.

This behaviorlet ends in a stop.

### wall follow - compound

The **wall follow** behaviorlet is used to follow along a wall or cliff. 
Wall follow is configured to follow left or right.
There are different phases to wall follow. There is the bump follow, followed by the IR follow, if an IR wall sensor is available.

Bump follow will bump follow a wall until it determines from the bump points whether it is following a wall or a curve.

First, the wall follow behavior will determine if three points form a line using the following formula. 

    Angle AngleBetween3Points(Coordinate A, Coordinate B, Coordinate C)
    {
        Angle atanA = units::atan2(A.x - B.x, A.y - B.y);
        Angle atanC = units::atan2(C.x - B.x, C.y - B.y);
        return atanC - atanA;
    }

If this angle is less than 4 degrees and the summation of the distance of the two lines is greater than 100mm a line is called.

If a line is detected the wall follow switches to a servo on the wall IR.  If a bump is detected, the behaviorlet switches back to bump follow.

There is another component of the algorithm where when the robot is bump following, the and the current servo or "scallop" in is continuing for a period of time longer than expected,
the angular velocity will start to increase and the forward motion will start to decrease. This lets the behaviorlet "hug" objects.

Bump reflex is integral to the behaviorlet.

Cliff and wheel drop reflexes stops this behaviorlet.

Update monitoring return current pose.

Pose tune not active

This behaviorlet can be interrupted and return final pose in the monitor path.

This behaviorlet ends in a stop.

### spiral - ziggurat - compound

spiral and ziggurat are similar behaviorlets in that they spiral out from a central location. 
Spiral is a circle; ziggurate is a box.

bump, cliff and wheel drop reflexes stop this behaviorlet.

Update monitoring return current pose.

Pose tune not active

This behaviorlet can be interrupted and return final pose in the monitor path.

This behaviorlet ends in a stop.


### OTHER BEHAVIORLET

While these work for mobile robotic platforms - what type of behaviorlets work with arms, hands and the like?

## interfaces

## dialog

