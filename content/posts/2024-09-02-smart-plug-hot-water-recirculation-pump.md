---
title: "Smart Plug for Hot Water Recirculation Pump"
date: 2024-09-11T15:00:00-06:00
type: post
author: James
categories:
  - Home Improvement
tags:
  - Home Improvement
comments: true
---

The [previous post]({{< relref "2024-09-02-hot-water-recirculation-pump.md" >}}) described
how a hot water recirculation pump solved a problem with a shower that took a long time to
get hot water.

One problem that was mentioned is that the included dial timer to control the pump was prone
to problems of getting out of synch if there is a power outage and the dial stops turning. Also
the settings of the dial timer aren't real accurate given that they are in 15-minute increments:

![Watts Programmable Timer](/images/watts-timer.jpg "Watts Programmable Timer")


### Solution - Kasa Outdoor Smart Plug

As mentioned in the previous post I thought that the
[Kasa Outdoor Smart Plug](https://www.amazon.com/Kasa-Smart-Resistance-SmartThings-KP401/dp/B099KLNM24)
would be a good solution as we already use one for the string lights on the outdoor patio.
These devices use the local wifi network for setup and integrate well with a smart-phone app
called Kasa.

#### Installation

Since the timing will be controlled by the plug and power will be delivered to the pump
only when it the plug is on, the switch on the pump dial timer that is shown above should
be set to "ON" insted of "TIMER". That way it will come on any time it is getting power.

The Smart Plug is easy to install.  It is first plugged into the wall or outdoor plug and an
indicator light will start flashing on the top of the device to indicate it is ready for
installation.

![Kasa Smart Plug Plugged In](/images/kasa-smart-plug.jpg "Kasa Smart Plug Plugged In")

Then at that point you can use the Kasa app to complete
the installation. Once installed the app will show the device with the name given during
the installation. Since there wasn't an icon available for the hot water pump I added one
from a picture that I took earlier and this is what it looks like from the app's device list:

![Kasa App Devices](/images/kasa-app-devices.png "Kasa App Devices")

The button to the right of each device can be used to turn the plug on and off remotely
from any location.

#### Scheduling the Plug

To schedule the smart plug to come on at the desired time, there is a scheduling function
on the app for each device. The start and stop time for one or more intervals can be set
by adding the times for the app:

![Kasa App Scheduling](/images/kasa-app-schedule.png "Kasa App Scheduling")

These timings are very precise and are based on the time from the network.

#### Integration With Google Home and Amazon Alexa

The Kasa app can also integrate with Google Home and Amazon Alexa which allows for voice
commands to control the pump directly. For example: "OK Google, turn on the Hot Water Pump".

### In Conclusion

The Kasa Smart Plug is a very good solution and eliminates the worry about the timing getting
out of synch and offers a lot of flexibility in case we need to turn the pump on at a different
time for a one-off shower.  In that case you can also set a timer on the app to ensure that
the pump will be turned off after an interval time.

There are other devices that are available for the Kasa system including smart bulbs, switches,
door bells, dimmers, security cameras, etc.  They can all be combined together to create a 
smart home environment.
