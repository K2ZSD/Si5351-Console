Si5351/A Console
================

A simple Arduino sketch that makes most Si5351/A functions available in
command-line form using a serial terminal (typically: the Arduino "Serial Monitor"
feature).  

This is very useful during the bring-up/debug of home-brew radio projects.  Once
things are working properly you generally need some rig-specific firmware, but this
utility will get you up and running quickly/generically without the need for
displays, encoders, Arduino programming, etc.

See the reference below for the supported commands.

All three clocks on an Si5351 can be controlled independently.

Basic sweep capability is included.

Uses the Etherkit Si5351 library (version 2.1.0).  See: https://github.com/etherkit/Si5351Arduino

The Si5351/A datasheet is here: https://www.silabs.com/documents/public/data-sheets/Si5351-B.pdf

For reference, testing happened using version 1.8.5 of the Arduino IDE.

Author
======

Written by Bruce MacKinnon KC1FSZ
31-December-2017

Commands
========

    e0/e1/e2 <0|1>       Set CLK0/1/2 enabled
    f0/f1/f2 <freq Hz>   Set CLK0/1/2 display frequency
    o0/o1/o2 <freq Hz>   Set CLK0/1/2 offset
    m0/m1/m2 <mult>      Set CLK0/1/2 multiplier
    d0/d1/d2 <0|1|2|3>   Set CLK0/1/2 drive strength
    s0/s1/s2 <0|1|2>     Set CLK0/1/2 step mode (0=none, 1=display, 2=offset)
    ss <step Hz>         Set step size
    wc <count>           Sets the number of sweep cycles to run
    ws <count>           Sets the number of steps in a sweep
    wd <delay Ms>        Set the delay at each step in milliseconds
    sw                   Starts the sweep sequence running
    we                   Save all state to EEPROM
    re                   Load all state from EEPROM
    =                    Step up
    +                    Step up  
    -                    Step down
    st                   Display Si5351/A status
    co <correction>      Set correction in parts-per-billion

Hardware Requirements
=====================

This has been developed/tested using an Arduino Pro Mini and an Adafruit
Si5351 board.  It should work fine on any Arduino/Si5351 board combinations.

Reminder to home-brewers: put an RF choke on your USB cable! Mix 31.

Usage Notes
===========

This is normally used with the serial monitor component of the Arduino IDE.  However,
any terminal program can be used. Be sure to set your terminal program to
9600/8/N/1 and define the EOL character as "newline" (ie: \n, ASCII 10).

The Si5351/A clock is set to this value:

    Si5351 Clock = Display Frequency * Multiplier + Offset

Illustration For Superhet Developers
------------------------------------
An illustrative combination shows how this can be helpful when developing
superhets.  Imagine we're creating a 40m rig (7MHz, LSB) with an IF of 12MHz. A
common design would be to create a VFO at 5MHz and a BFO at 12MHz.  But also
imagine that your IF filter is a 2 KCs low, so the best BFO should actually be
at 11998000 Hz. The math can get a bit tricky because you need to subtract the
desired "display" frequency from the 12MHz IF (actually 11998000Hz) to create
the VFO.  Rather than keeping all of this in your head, you can configure the
Multiplier to be -1 and the Offset to be 11998000.  When you set the display
frequency to 7255000 the the output clock is actually set to:

    7255000 * -1 + 11998000 = 4743000

Or the usual way to think about this:

    11998000 - 7255000 = 4743000

Even better, when you step "up" the display frequency, this actually steps
down the VFO (as expected).  So pressing "+" when the step size is 500 Hz will
set the display frequency to 7255500, but the Si5351 clock will be set to:

    11998000 - 7255500 = 4742500

Another Illustration
--------------------
This illustration shows a helpful way to experiment with the optimal
BFO setting when you don't know exactly how your IF filter is working.
If can be helpful to step up/down the BFO while keeping the display frequency
constant.  You might do this by listening to a side-band conversation and
adjusting the BFO for the best audio quality.

But when you shift the BFO you also need to shift the VFO by the same
amount.  The math here can be tricky as well.  

**Step mode 1** causes the display frequency of the selected clock to step
up or down by the configured step size.  **Step mode 2** causes the *offset*
of the selected clock to step up/down while keeping the display frequency
constant.

**Step mode 0** causes the clock to stay fixed, even when other clocks
are being moved around.

So a typical configuration for filter optimization would put the VFO clock
into step mode 2 and the BFO clock into step mode 1.  That way pressing +/-
will move up/down the BFO and the VFO by the right amounts, while staying
on the same display frequency.  

Sweep Notes
===========

The sweep capability can be useful in a number of contexts.  Characterizing
filters is the one task that the feature was created for in the first
place.

Think of the sweep feature as an automation of pressing the +/- buttons a
bunch of times and recording an analog sample at each step.  

This article on N2CQR's excellent blog site may be helpful context since it
explains what I was doing when the sweep capability was first implemented: http://soldersmoke.blogspot.com/2017/05/homebrew-peppermint-bitx-kc1fsz-goes.html

Here are the key parameters:

* **Sweep Count** - The number of repeated sweeps to perform.  Sometimes if I
am looking at the result of the sweep on an old-school analog oscilloscope it
is useful to run the sweep 100s or even 1000s of times.
* **Step Count** - The number of steps in each sweep.  This is self-explanatory.
* **Step Delay** - The number of milliseconds we stop at each step in the sweep.

At the start of each sweep a digital pin (3 by default) is pulsed high for
10 milliseconds.  This is a good way to trigger an external tool like an
oscilloscope or something.

The regular step size parameter is used to control how far we move in each step.  All
of the normal capabilities of the step (step mode, etc.) are honored during the
sweep.  It's just like pressing +/- a lot of times!

At the end of each step (i.e. after things have settled) a reading is taken
from an analog pin (A0 by default) and saved. This can be used to read a
power level or some other data point.

At the end of each sweep the saved samples are output to the terminal for
further analysis.  I generally copy/paste this data into Excel (LibreOffice)
or MatLab or something else to get a look at the sweep data.

Version History
==============
1.0
---
* Initial version

1.1
---
* Merged sweep logic from other versions
* Removed the help to free memory
