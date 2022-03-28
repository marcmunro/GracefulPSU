# RPi Graceful Power Supply Controller

This is a graceful-shutdown power supply for a Raspberry Pi or
similar Single Board Computer (SBC).

It has 3 major functions:

1) Provide very short term backup power when input power is lost
   A supercapacitor provides enough power to allow time for a clean
   shutdown. 
2) Provide software and push-button control of power
   This allows the computer to be turned on by a short button press
   and the power removed by a long press.  It also allows the computer
   to turn off the power itself, and allows it to monitor the button
   state.
3) Provide a regulated 5V supply from an unregulated ~12V supply

## Circuit Description

Note that components marked with an asterisk (*) are optional.  The
optionality of these components is decribed in the appropriate section
below.

### Power Input

The circuit is designed to allow 2 independent 12V power inputs.  This
allows the SBC, for usage on a sailboat, to be powered by independent
circuit breakers: 1 for accessories allowing it to be used as an
entertainment system; and 1 for instruments allowing it to be used for
navigation.

Schottky Diodes (D6, D7) carry the power from each input source,
preventing one source from feeding power into the other, and
preventing our backup power feeding back to the source.

Similarly D1 and D4 are used to provide an input voltage signal
whenever input voltage is provided.  This allows us to sense loss of
input power independently of the backup power from our
supercapacitor.

If only a single input source is required J2 may be ignored; D1 and
D7 may be ommitted; and D4 replaced with a wire link.

### Power Sensing

Sense input is provided to GPIO pins (header pins 29 and 31) to allow
the SBC to determine whether, and which, inputs are providing power.

If only a single power source is needed D2, R2, R8 and C2 can be
ommitted, and D5 replaced with a wire link.

### SuperCapacitor Backup Power

This part of the circuit:

- limits the charge current for the supercapacitor, C4, to avoid a
  large current spike when power is connected;
- ensures that the supercapacitor fully charges;
- allows current to flow from the supercapacitor unimpeded when
  input power is removed;
- disables the Power Control Latch until the supercapacitor has
  reached a suitable level of charge. 

When power is applied, C1 slowly charges through R3, lowering the
input voltage to the inverting input of U1B.  R6, R7 and R9 form a
voltage dividing and offsetting network from the cathode of C4 to the
non-inverting input of U1B.  This roughly halves the the C4 cathode
voltage and offsets it by about 1.5V so that C1 must charge a little
before U1B's output will start to rise.  When U1B's output rises, Q1
will turn on, charging C4 until it reaches about twice the level of
charge of C1.  As C1 continues to slowly charge, so will C4, which
will be fully charged when C1 is charged to around 5V.

U1A is a simple voltage comparator configured as a Schmitt Trigger.
When C4 reaches around 85% (around 10.5V) of being fully
charged, the output will switch from high to low, turning on the LED
(J5, J6) and enabling the Power Control Latch switch (J7, J8).  If
optional C6 is provided, the Power Control Latch will be automatically
turned on at this time.

When input power is disconnected, U1As non inverting input will be
pulled low, disabling the switch and turning off the LED.

When input power is disconnected, U1B's non-inverting input will be
pulled low by R3, causing U1B's output to go high truning Q1 on hard.
This will allow power to be drawn from Q4 with almost zero loss.
Also, C1 will now quickly discharge through R1 and D3, readying it for
the next time power is applied.  Finally, the non-inverting input of
U1A will drop to 0V sending U1A's output high and turning off the LED
(J5, J6).

#### Charging Time and Current

[This](https://www.amazon.ca/LMUWF-Capacitor-Module-Capacitors-Protection/dp/B09FXK86RZ/ref=sr_1_2?crid=354P0FGQLHK3U&keywords=2f+super+capacitor&qid=1648493702&sprefix=2f+supercapacitor%2Caps%2C146&sr=8-2)
is the supercapacitor module I used, which is actually 1.6F.

The charge current for the supercapacitor is given by the formula:

    C * V / t

where C is the capacitance, V is the voltage to which it is to be
charged, and t is the time taken to charge.

With the circuit as given, the full charge time (for 12V) is around 8 seconds,
which gives us 1.6 * 12 / 8 = 2.4 Amp.

The Schottky diodes are rated at 4A, so using any larger
supercapacitor will require circuit modification to limit the charge
current.  This can be done by increasing the value of C1 and/or R3. 

### Power Control Latch

This section of the circuit is from:

http://www.mosaic-industries.com/embedded-systems/microcontroller-projects/raspberry-pi/on-off-power-controller

The MOSFET pair of Q2 form a latching circuit that provides power to a
5V regulator.

When power is applied to J1 or J2 the gate of Q2b is driven high
causing it to not conduct (R17 and C5 assist in this).  Q2a's gate
will be drawn low at this point (via internal resistance in the pi)
keeping Q2b from conducting.

To turn on Q2a, the Voltage at its gate (label A in the circuit) must
be dropped to zero.  This can be done by momentarily closing the
contacts of a switch across J7 and J8.  The optional capacitor C7 will
automatically switch power on once the output of U1A drops to zero
volts.  If you do not want automatic switch-on, leave C7 out of the
circuit.

Once Q2b turns on, Q2a's gate is drawn high, turning it on, and
keeping Q2b's gate low which keeps Q2b conducting.

To turn off the power, the gate voltage at Q2b must be brought low.
This can be done by the pi bringing GPIO4 (header pin 7) low, or by a
long press on the switch (J7, J8) which will slowly discharge
capacitor C7 until Q2a's gate voltage is sufficiently brought down to
turn it off.

The Pi's GPIO4 pin can also be used as an input to monitor the
switch state.  If a short press is detected it should start the power
down process, finishing by rconfiguring GPIO4 as an output and setting
it low.

Shutdown should also be initiated when input power is disconnected
(from J1 and J2).  This is determined by monitoring GPIO5 and GPIO6
(header pins 29 and 31).

### 5V Regulator

5V for the Rpi is provided by a buck convertor.  This part of the
circuit was generated by Texas Instruments' WEBENCH tool.

Q3 provides an unbacked-up 5V supply.  When input power is
disconnected Q3 will no longer conduct.  This is intended for
peripherals such as monitors that need not be powered during the
shutdown of the computer. 
