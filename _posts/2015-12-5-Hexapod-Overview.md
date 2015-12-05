---
layout: post
title: The Hexapod...
---

So I'm building a Hexapod... I've been thinking about this for a loooong time (8 years by now probably), and finally got around to forking out $350NZD for a set of 20 servos (3 per leg + 2 spare 'cos I'm sure I'll break at least 2 in the process...)

Here's the design so far: 

![Hexapod Design V1]({{ site.baseurl }}/images/hexapod-v1.png)

Overall goals
--------------

So, I'm going to start documenting my thought process / workflow etc for building up this beast...

Overall goal is an autonomous robot, with some basic computer vision, that can track a brightly coloured object (e.g. ball).... Kinda ironic given that I've got a dog who will happily do that for me for hours on end already... 

Extra for experts will be some kinect / point cloud generation, and autonomous navigation within an environment

Tools
-----

Here's the general set of tools I'm planning to use for this work. Bound to change by the time I'm finished, but here's the list at the outset...

* http://www.onshape.com - Onshape is awesome for 3D modelling... 
* My Huxley RepRapPro 3D printer (mostly for prototyping at home - I'll use the Makerbot Replicator 2 at work for the final prints) 
* Ardiuno Uno for initial PWM generation and prototyping. Nice and easy to get up and running quickly
* Raspberry PI B for final Servo control, and webcam input.
* ROS for simulations, and overall control. 
* OpenCV for computer vision
* Laser cutter for larger body parts


The Design
----------

* [Hexapod by Jeremy Gold](https://cad.onshape.com/documents/8c371044a61f4659a7fb9109/w/6f36ea3f6f37415387b44e93/e/a08924425af74449a2a145db) in Onshape. You might need to register / log in to be able to see this...


Cheers,

Jeremy

