---
layout: post
title:  "Open Source 555 Timer"
date:   2025-03-08 10:09:56 -0500
math: true
categories: projects 
---
## Introduction ##

In this post, I’ll share how I designed, laid out, and manufactured my own analog chip using open-source tools for a total cost of only $210. 

## Design ##

For schematic capture I used [Xschem](https://sourceforge.net/projects/xschem/). To learn the tool I was able to find video tutorials on YouTube. I found [B. Minch](https://www.youtube.com/watch?v=BpPP2hE_eK8) the most useful. I referenced the [Xschem Manual](https://xschem.sourceforge.io/stefan/xschem_man/xschem_man.html) quite a bit as well.

The actual 5V CMOS version of the 555 schematic is simple and compact, but it was designed in an era where cost-per-transistor was still a bigger factor. Imagine hand-cutting [Rubylith](https://en.wikipedia.org/wiki/Rubylith) for each transistor. A lot more work than having to learn some new open source CAD tools.

The below schematic is the *actual* schematic of the real CMOS 555 timer, published by the late Hans Camenzind himself. His book [is free online](http://www.designinganalogchips.com/).

[![Original CMOS 555 Timer](https://github.com/vincentfusco/tt06_555/blob/main/docs/555_cmos.PNG?raw=true)](https://github.com/vincentfusco/tt06_555/blob/main/docs/555_cmos.PNG?raw=true)

Since cost-per-transistor is cheap and I won't be having to cut Rubylith, I decided to implement it my own way. 

Xschem supports hierarchical design, and aside from the resistors and discharge FET, I put everything else inside of its own level of hierarchy. 

I implemented the comparator using a positive feedback decision circuit sized to provide some hysteresis and I used three 10k resistors to generate the reference voltages. [(So perhaps I should refer to this open source version as a 10-10-10 Timer)](https://www.electronicdesign.com/technologies/analog/article/21252714/electronic-design-the-origin-explanation-and-applications-of-triple-five-timers).

I sized a large NFET as a discharge device, and I put some additional tapered inverters between the SR latch and the discharge FET.

I designed my own logic cells, including the inverters and SR latch, but the Skywater PDK does come with a standard cell library that I haven't looked into yet.

[![555 Schematic](https://github.com/vincentfusco/tt06_555/blob/main/docs/timer_core_schematic.PNG?raw=true)](https://github.com/vincentfusco/tt06_555/blob/main/docs/timer_core_schematic.PNG?raw=true)

## Simulation ##

For simulation I used NG SPICE. Again, YouTube for simulation tutorials is really useful, but don't be afraid to reference the [NG Spice Manual](https://ngspice.sourceforge.io/docs/ngspice-manual.pdf) too. I used it a lot, and was able to find examples of what I needed by searching with CTRL + F.

I built a top-level symbol and wired up a testbench where I configured the timer in astable mode. 

I wanted to test out Monte Carlo simulation. With [B. Minch's](https://www.youtube.com/watch?v=fGxs2TnDgrU) excellent YouTube tutorials I was able to figure it out.

Here is a histogram plotting the measured frequency at TYP:

[![Monte Carlo Simulation](https://github.com/vincentfusco/tt06_555/blob/main/docs/timer_core_mc_results.png?raw=true)](https://github.com/vincentfusco/tt06_555/blob/main/docs/timer_core_mc_results.png?raw=true)

To get more practice using the tools, I also measured the spread in offset voltage and hysteresis of just the comparator in another Monte Carlo test bench shown below. 

[![Monte Carlo Simulation of Comparator](https://github.com/vincentfusco/tt06_555/blob/main/docs/comp_p_simulations.PNG?raw=true)](https://github.com/vincentfusco/tt06_555/blob/main/docs/comp_p_simulations.PNG?raw=true)

Although I have plotted a Gaussian curve over top of both plots, the offset voltage looks bimodal, and the hysteresis plot is right skewed. Interesting, and I might have to make a follow up post to discuss why this is later.

I think it's pretty cool to have Monte Carlo simulation working with open source tools! It takes a bit of setting up and is not GUI driven but it's not too hard, and once you get everything set up once its easy to copy for future testbenches.
## Layout ##

For layout I used Magic. I found [this tutorial](https://www.youtube.com/watch?v=XvBpqKwzrFY) the most useful for learning how to use the tool.

[![555 Timer Layout](https://github.com/vincentfusco/tt06_555/blob/main/docs/555_layout.png?raw=true)](https://github.com/vincentfusco/tt06_555/blob/main/docs/555_layout.png?raw=true)

I followed Tim's layout style, using full guard rings for all analog devices. 

I laid out the comparator input-pair in a common-centroid fashion, which you can see annotated on the layout below.

[![Comparator Schematic](https://github.com/vincentfusco/tt06_555/blob/main/docs/comp_p_schem_vs_layout.PNG?raw=true)](https://github.com/vincentfusco/tt06_555/blob/main/docs/comp_p_schem_vs_layout.PNG?raw=true)

I used wide metal for VDD and VSS. I copied Tim's method of "filling" out VSS metal in the local interconnect level between guard rings. I thought his style made for a really clean analog layout.

The total width of my design was about 45um x 40um.

## LVS ##

For LVS I used Netgen. I recommend reading [the tutorials on Tim's site](https://opencircuitdesign.com/netgen/) for this one.

## Extraction ##

I extracted an RC netlist from the layout and created the below testbench. I put an instance of the schematic-only version of the timer next to the RC extracted version of the timer.

I ran a simulation to compare the resulting frequency from each.

The top waveforms are the output from the extracted version. 

The bottom waveforms are the output from the schematic version.

Highlighted in yellow are the result of a .MEASURE statement used to measure the time high, time low, and total frequency of each.

You can see that the observed difference in frequency is about 50Hz. Close, but not zero!

The extraction tool lets you extract R-only, C-only, or RC. The results below are for the RC version. It would be interesting to see if this error is present in the R-only or C-only versions.

[![Extracted Test Bench](https://github.com/vincentfusco/tt06_555/blob/main/docs/tb_tt_um_vaf_555_timer_astable_schematic.PNG?raw=true)](https://github.com/vincentfusco/tt06_555/blob/main/docs/tb_tt_um_vaf_555_timer_astable_schematic.PNG?raw=true)

[![Extrated Numerical Simulation Results](https://github.com/vincentfusco/tt06_555/blob/main/docs/tb_tt_um_vaf_555_timer_astable_results.png?raw=true)](https://github.com/vincentfusco/tt06_555/blob/main/docs/tb_tt_um_vaf_555_timer_astable_results.png?raw=true)

## Testing ##

I used Tiny Tapeout to manufacture my chip. For $210.00 including shipping they sent me one packaged IC in a QFN mounted on a test PCB shown below:

[![Test Board](/pics/555_test_board.jpg)](/pics/555_test_board.jpg)

I wired up the 555 in astable mode with a frequency low enough for some good LED blinking. The below scope shot shows the comparator switching at 600mV and 1.2V:

[![555_test_board](/pics/555_scope.jpg)](/pics/555_scope.jpg)

Full lab setup below comparing schematic, RC extracted, and bench-top results:

[![555_blinking_LED.gif](/pics/555_blinking_LED.gif)](/pics/555_blinking_LED.gif)

Matt Venn - creator of Tiny Tapeout - put together [this video of his bench top testing of the 555](https://www.linkedin.com/posts/matt-venn_asic-tinytapeout-opensourcesiliconstream-activity-7293652281994964992-QAga?utm_source=share&utm_medium=member_desktop&rcm=ACoAAA1Ud1sBSTgYI5kdaUC5rC26cc-DA3BvHL4).

## Conclusion ##

This project provided me with a solid foundation in the open-source tool flow. I gained hands-on experience with Xschem, NGSpice, Netgen (LVS), and Magic (layout and DRC). Additionally, I learned to use GitHub and became familiar with the process of delivering GDS to Tiny Tapeout. It was only after completing this project that I decided to start writing about my work, but now that I understand the tool flow and can approach future projects with writing in mind from the start, I’m excited to take on more. One of the most rewarding aspects of open-source chip design is the freedom to share ideas openly, without NDAs, and I look forward to contributing to this collaborative environment in future projects.
