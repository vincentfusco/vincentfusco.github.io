---
layout: post
title:  "Open Source 555 Timer"
date:   2025-03-08 10:09:56 -0500
math: true
categories: projects 
---
## Introduction ##

The 555 Timer is perhaps the most iconic IC in hobbyist electronics. [It is also the most popular analog chip ever manufactured](https://en.wikipedia.org/wiki/555_timer_IC), with estimates that there are over a billion units still produced annually.

Wiring up a DIP 8 555 timer on a breadboard in order to blink an LED is a rite-of-passage     — the ["hello world"]( https://en.wikipedia.org/wiki/%22Hello,_World!%22_program) moment for many beginner electronics enthusiasts. For me, my first introduction to PCB design came from [Chris Gammell’s Getting To Blinky](https://contextualelectronics.com/lessons/introduction-to-gtb-5-0/) course. I still have a few of those ubiquitous purple [OshPark]( https://oshpark.com/) PCBs laying around in my parts drawer.

So it seemed fitting as a first experiment with hobbyist IC design to continue the tradition and to, “blink an LED the hard way” while I focused on learning the open source tools.

## Design ##

For schematic capture I used [Xschem](https://sourceforge.net/projects/xschem/).

The actual 5V CMOS version of the 555 schematic is simple and compact, from an era where cost-per-transistor and hand-cutting [Rubylith](https://en.wikipedia.org/wiki/Rubylith) was still a factor:

The below schematic comes from Hans Camenzind's - the creator of the original 555- own book, [which is free online](http://www.designinganalogchips.com/).

![Original CMOS 555 Timer](https://github.com/vincentfusco/tt06_555/blob/main/docs/555_cmos.PNG?raw=true).

However, cost-per-transistor and having to hand cut Rubylith are no longer constraints, so I decided to implement it my own way. Xschem does an excellent job supporting hierarchical design. Aside from the resistors and discharge FET, I put everything else inside of its own level of hierarchy. I implemented my comparator using a cross-coupled pair:

![555 Schematic](https://github.com/vincentfusco/tt06_555/blob/main/docs/timer_core_schematic.PNG?raw=true)

![Comparator Schematic](https://github.com/vincentfusco/tt06_555/blob/main/docs/comp_p_schem_vs_layout.PNG?raw=true).

I will write another post specifically about the comparator design.

## Simulation ##

I built a top-level symbol and wired up a testbench where I configured the timer in astable mode. 

I wanted to test out Monte Carlo simulation. With [B. Minch's](https://www.youtube.com/watch?v=fGxs2TnDgrU) excellent YouTube tutorials I was able to figure it out.

Here is a histogram plotting the measured frequency at TYP:

![Monte Carlo Simulation](https://github.com/vincentfusco/tt06_555/blob/main/docs/timer_core_mc_results.png?raw=true)

Pretty cool!

## Layout ##

For layout I used Magic. I found [This video](https://www.youtube.com/watch?v=XvBpqKwzrFY) extremely useful for learning how to use the tool. In the video Tim Edwards, the creator of Magic, shows you how to lay out an op-amp and he gives some really useful tips.

![555 Timer Layout](https://github.com/vincentfusco/tt06_555/blob/main/docs/555_layout.png?raw=true)

## LVS ##

For LVS I used Netgen. ![The tutorials on Tim's site were useful](https://opencircuitdesign.com/netgen/).

## Extraction ##

I extracted an RC netlist from the layout and created the below testbench. I put an instance of the schematic-only version of the timer next to the RC extracted version of the timer.

I ran a simulation to compare the resulting frequency from each.

![Extracted Test Bench](https://github.com/vincentfusco/tt06_555/blob/main/docs/tb_tt_um_vaf_555_timer_astable_schematic.PNG?raw=true)

![Extrated Numerical Simulation Results](https://github.com/vincentfusco/tt06_555/blob/main/docs/tb_tt_um_vaf_555_timer_astable_results.png?raw=true)

The frequency measurement of the simulated schematic and extracted RC netlists match closely, as shown above.

## Testing ##

Silicon is back! [Here is a video of Matt Venn - creator of Tiny Tapeout - testing the chip on the bench](https://www.linkedin.com/posts/matt-venn_asic-tinytapeout-opensourcesiliconstream-activity-7293652281994964992-QAga?utm_source=share&utm_medium=member_desktop&rcm=ACoAAA1Ud1sBSTgYI5kdaUC5rC26cc-DA3BvHL4).

## Conclusion ##

This project provided me with a solid foundation in the open-source tool flow. I gained hands-on experience with Xschem, NGSpice, Netgen (LVS), and Magic (layout and DRC). Additionally, I learned to use GitHub and became familiar with the process of delivering GDS to Tiny Tapeout. It was only after completing this project that I decided to start writing about my work, but now that I understand the tool flow and can approach future projects with writing in mind from the start, I’m excited to take on more. One of the most rewarding aspects of open-source chip design is the freedom to share ideas openly, without NDAs, and I look forward to contributing to this collaborative environment in future projects.
