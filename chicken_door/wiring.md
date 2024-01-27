---
layout: page
title: Wiring
subtitle: hardware overview
menubar: door_menu
show_sidebar: false
toc: true
---

## Door switch overview

* The door is controlled by a [single pole, centre off SPDT switch](https://www.jaycar.com.au/spdt-centre-off-miniature-toggle-switch-solder-tag/p/ST0336) (open - auto - close).
* Conveniently the switch is wired in using a [RJ14 cable](https://en.wikipedia.org/wiki/Registered_jack#Pinout)

This modification is therefore non-destructive as we can wire a new RJ14 cable into the relay HAT for the Pi and plug this into the door controller.

![wiring overview](/stakach.github.io/img/door-wiring-overview.png)

## Failure modes

There are two ways the door should can be configured depending on your primary use case

1. fallback auto - fall back to the light sensor if the computer fails
1. fallback close - close the door if the computer fails

Our chickens are free range so we have the relay wired up to revert to auto light sensor operation.

The main risk with this is that [foxes can be crepuscular, diurnal, nocturnal or cathemeral](https://www.squirrelsatthefeeder.com/are-red-foxes-nocturnal-or-diurnal/) which can lead to undesirable opening times.

The benefit being that if the computer were to fail during the day, the chickens are not locked out of the coop.

## Wiring details

Only 3 of the 4 RJ14 wires are used when connected to the switch

<img src="/stakach.github.io/img/toggle-switch.png" width="500em" alt="toggle switch wiring">

* Closed == Red + Green
* Sensor == no connection
* Opened == Red + Yellow

<img src="/stakach.github.io/img/rj14-pinout.png" width="150em" alt="RJ14 pinout">

So given the red wire is common but needs to be connected to both relay modules, after stripping the RJ45 cable I used the spare black wire as a bridge.

![relay wiring](/stakach.github.io/img/relay-wiring.png)

* NO (Normally Open) Switch = Stays in open state, circuit disconnected, unless the relay is active
* NC (Normally Closed) Switch = Stays in closed state unless the condition is met

The wiring in the diagram represents fallback to auto / sensor, both yellow and green wires connected to the NO terminal.

To fallback to closed state then the green wire should be moved to the NC terminal.
