---
title: "Roomba Dark Rug Hack"
date: 2023-08-19T11:00:00-05:00
type: post
author: James
categories:
  - General
tags:
  - Roomba
  - Hacks
comments: true
---

We bought an area rug for the living room and it caused the Roomba to hang up while
it was doing the cleaning. Basically it would stop in the middle of the rug because
it couldn't enter areas where there were dark patterns on the rug. After reviewing
what might cause this I found out it was due to the "cliff sensors" under the Roomba seeing the dark areas and it was interpreting it as a ledge and to not go over it. Also it could happen if the whole rug is very dark and the roomba would likely not enter it at all.

Here is an example of the type of patterns that it was hanging it up:

![Rug Patterns](/images/roomba_problematic_rug.jpg)

In particular it would try to go over the dark edge then back up and try to go
another direction. It would then hit another
edge and back up and try again. After a few attempts it would stop with an error saying to move the Roomba to a new location. Therefore the rug wouldn't be cleaned and also it also couldn't be left unattended to finish the rest of the room since it would eventually hang up.

**Caution: This hack will disable the cliff sensors which will allow the Roomba
to to over the edge of stairs or other ledges. When in use make sure any stairs or ledges are blocked!**

## Researching Different Sensor Hacks

After researching on the web and watching YouTube videos, I found there were several ways people hacked the sensors to "fix" this problem:

1. Cover the sensors with white paper and/or reflective tape.
2. Cover the sensors with foil and then covering with tape.
3. Modify the unit such that the sensor's LED shines directly on the detecting sensor (bypassing the outside completely).

I tried the first two methods and neither worked for me (Roomba 6 series). I kept
getting the error to move the Roomba as if it was already sensing a dark place. The
third method looks like it would be a lot of work and there is some risk that
a mistake could break the unit. It would also void the warranty which is not an
issue for me since the Roomba is over 10 years old.

## Main Problem

The reason that the simple solution didn't work for me is that a closeup view of the sensor show that the two sides of the sensor are separated by a raised black ridge in the center as shown in the following image:

![Sensor Detailed View](/images/roomba_cliff_sensor.jpg)

The raised black divider shown next to the red arrow separates the LED emitter on one side from the light sensor on the other side. As the sensor is pointing down it normally relies on reflection from a lighter floor to show light on the sensor. The divider helps ensure that the light sensor doesn't get LED light directly from the other side. So when white tape or foil is put over the sensor it covers the two sides and the raised divider prevents light from reflecting back to the sensor.

What is needed is a way to raise a reflective surface so that it rises above the height of the divider and allows the reflective light from one side to be seen by the light sensor.

## Solution That Worked

I read in a thread on this problem where a few people had cut a large-diameter straw (as in a sipping cup) in half longitudinally. Then the half barrel was cut to length and taped over the sensor.  There were several replies that this technique worked so I decided to try it.

### Preparing Straws

I didn't have any large-diameter straws on hand so decided to use a smal-diameter drinking straw. One advantage of these straws we have is that there is a longitudinal strip down the straw and that can be used as a guide for scissors to cut it in half. Then the halves can be laid togther long-ways on a table and some tape placed (lightly) over them. Then cut any excess tape from the assembly.  Below are the straws that I used and the completed sensor cover with yellow tape to hold it together:

![Straws and Completed Sensor Cover](/images/roomba_straws_and_sensor_cover.jpg)

### Attaching the straw assemblies

The straw assemblies can then be attached to cover the cliff sensors. Cut them to 1 and 3/8 inches in length and place one over the sensor with some overlap on one of the straws going into the gap nex to the sensor. Carefully put Scotch tape over the assembly, being sure not to flatten the straws with too much downward tension. Below is a completed assembly in place over the sensor noting that slack has been given so that the straw halves maintain their arch:

![Attached Sensor Cover](/images/roomba_sensor_cover_detail.jpg)

### Covering all of the sensors

Repeat the process for the remaining sensors (4 total on the Roomba 6 series). One thing to watch out for is that the tape doesn't obstruct the wheel movement, the brush movement, or the bumper movement. Below is the completed hack with all four covers in place:

![Completed Hack](/images/roomba_with_sensor_covers.jpg)

### Testing for Success

**First make sure any stairs are blocked since the cliff sensors will be disabled!** I used a broom to span the stair threshold at our place and that
was effective in keeping it above the stairs.

Move the Roomba next to the problematic carpet and click "Clean" to start it up. It may go in circles at first which is normal startup behavior. It should now be able to go over the dark areas of the rug where it was pausing before.

#### If it doesn't start at all...

This may be because one sensor is completely blocked and the light is not reflecting to it from the LED.

### Troubleshooting

#### If it backs up from the dark areas...

This indicates that at least one sensor is still seeing the black area of the rug and needs to be moved.

### Conclusion

This turned out to be a useful technique and hopefully will help others with
similar issues.




















