---
layout: post
title: Greenhouse Lift
---

> In which a programmer puts on his mechanical (?) engineer hat.

Recently we lost a greenhouse cover because the U-channel it was attached to came off of the greenhouse. We've already had two other wind related events with this particular greenhouse. I'd like to avoid having any more. So, I sat down with a pen, paper, and the Internet of engineering information.

Lift is what we're working with here. I've seen a couple different renderings of it ([NASA's](https://www.grc.nasa.gov/www/k-12/WindTunnel/Activities/lift_formula.html), [Wikipedia's](https://en.wikipedia.org/wiki/Lift_(force)#Lift_coefficient)). Here's the one we'll use, so that I don't have to type Greek letters.

*L = 1/2 \* d \* v^2 \* s \* CL*

* *d* is the density of air
* *v* is the speed of the aircraft
* *s* is the area of the airfoil
* *CL* is the coefficient of lift

[*d = 1.225 kg / m^3*](https://en.wikipedia.org/wiki/International_Standard_Atmosphere).

The airfoil is fixed, so I used wind speed instead. Gusts of 50mph have been a problem for us, and that's about the highest wind gusts that we get. So *v = 50mph = 73 ft / s = 22.352 m /s*.

The area of the airfoil is the footprint of the greenhouse. If I recall correctly, it's 96 ft x 30 ft. So *s = 96 * 30 ft^2 = 2880 ft^2 = 270 m^2*.

The coefficient of lift is a tough one to figure out. NASA has a [set of academic problems related to lift](https://www.grc.nasa.gov/www/k-12/WindTunnel/Activities/lift_formula.html), and the accompanying chart shows a peak *CL* of approximately 1.6. Wikipedia's [Lift coefficient page](https://en.wikipedia.org/wiki/Lift_coefficient) shows a maximum *CL* of 1.75. An aerospace engineering school's [high lift slide deck](http://www.dept.aoe.vt.edu/~mason/Mason_f/HiLiftPresPt1.pdf) maxes out around 1.75. I'm going to assume that we haven't found a magic formula for generating even higher lift, so I used *CL = 1.75* to have a worst-case estimate.

*L* = 1/2 * 1.225 * (22.352)^2 * 270 * 1.75<br>
*L* = 144,468.292 Newtons<br>
*L* = 32,477 lbf<br>

Where does this force end up? We have 34 metal 2 5/8" ground posts. Each post is bolted to a wooden hip board with a 3/8" carriage bolt. U-channel is screwed onto the hip board with Tek screws or wood screws. The plastic is clamped to the U-channel with wiggle wire.

I don't know how much force you need to apply to pull the metal pipes out of the ground. We have heavy clay soil. I'm going to assume that this is more than enough (over 1000 lb per post?) to handle the wind.

I'm not sure what grade the bolts are. Nucor has a [shear strength data sheet](https://www.nucor-fastener.com/Files/PDFs/TechDataSheets/TDS_013_Shear_Strength.pdf) that says that 3/8" bolts have at least 3900 lb shear strength. So our 34 bolts should be able to hold down 132,600 pounds.

The U-channels may have been screwed onto the boards about every 2 ft. This is what pulled off, so I can't measure it now. We used a combination of Tek screws and wood screws. A [random study of wood screw pullout strength](http://www.9wood.com/files/rd_reports/screw_withdrawl.pdf) shows that screws pull out well before 300 lb. So, if we managed to get 200 lb of pullout resistance per screw (and I doubt that we did), the U-channel should have been able to withstand 28,800 pounds of pull out force. That doesn't directly translate to the type of force the screws received, but if it's anything like the actual number, it's way too low. Our plan is to attach new double-U-channel with carriage bolts. If we use 1/4" carriage bolts at every post, we should be able to handle 60,000 pounds of force. If we use 2 at each post and one in between, we should be good for 180,000 pounds of force.
