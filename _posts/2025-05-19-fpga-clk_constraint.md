---
layout: post
title: "FPGA clk constraints, same external clock freq" 
date: 2025-05-19
---

A question popped up the other day, how does the constraints work on two clocks in an fpga when they have the same frequency but they are not from the same source, so they will not be synchronized?

Since the two clocks are from different sources they will never be exactly equal, the phase will not be at the same point.
So in a real design any signals going across these two clock domains needs attention to get the transition work correctly.

I tested this in quartus and if two clocks are constrained with the same frequency the analyzer will not find any issues if signals cross the two domains.
Also if the clocs are at a similar base frequency and low enough you get the same, e.g 10 and 15MHz clocks.

When you have these external clocks that you know is not related it is good to cut the timing analysis, since there is no point in doing it (the tool can never know). This can be done with the following command:
`set_clock_groups -asynchronous -group {clk_a} -group {clk_b}` remove the interception between the clock domains from timing analysis.

It is possible to check if there is any transfers between the clock domain by using the `report clock transfers` in timing analyzer, this should be used then to evaluate if there are any signals crossing the domains that should not cross.

## Conclusion
If two clocks input to an fpga has the same frequency but they are asynchronous it is important to have control over the constraints for them and have good control over all signals crossing between those clock domains.