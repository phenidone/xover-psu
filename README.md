# Preamp Power Supply

This is a PCB design for an analogue preamplifier power supply

## Inputs

Power can be provided as one of:
* high voltage (240V) AC, relay-switched
* high voltage (240V) AC, unswitched,
* low voltage (20+20V) unregulated DC

An opto-isolated input (5-24V DC) is provided to control the power-switching relay.  Low-power (2A) switched AC loads can be
connected via terminals on the PCB, and high power loads (20A) can be switched using spade terminals on the rear of the 
relay.

If the PCB transformer is omitted, unregulated dual-rail DC can be fed into the linear regulator section.  Additional
filter capacitors may also be inserted at this point.  The unmute circuit will NOT work unless the transformer is present or
low-voltage AC is otherwise supplied to the board.

## Outputs

The circuit provides:
* +/- 12-15VDC, linear regulated
* 12VDC from always-on switchmode brick, for control circuits
* un-muting relay output
* 50Hz opto-isolated pulses for external power-sensing logic
* power-relay output

The power-relay terminals can be used as an output port to drive an external
relay, or can be used as an input port to drive the onboard relay if the
remote/relay driver circuit (U2, Q1, etc) is not populated.

The 12VDC output terminal could be used as an input from an external plugpack
if the HLK10M12 is not present.

### Unmute Logic

The UNMUTE output is designed to switch a 12V muting relay and thereby suppress bad noises that occur during power-up/down
brownout of preamplifier and power-amplifier circuits.

The unmute logic uses a slow turn-on RC timer and a fast turn-off RC timer, both of which are charged from opto-isolated pulses
derived from the mains transformer secondary.  The UNMUTE output (12VDC up to 500mA) is enabled after the slow timer charges, 
which is about 1s after power is applied.

When power is withdrawn, the fast timer discharges in about 50ms, which causes Q2 to discharge the C in the slow timer,
thereby immediately shutting off the UNMUTE output and resetting the startup time.

Each RC timer uses a TL431 to provide a distinct transition point, for fast(ish) turn-on/off transitions.

A secondary AC pulse output is provided as bare optoisolator pins, allowing an external logic circuit to safely monitor the presence of AC
power going into the linear regulator.

### Linear Regulation

The dual-rail DC output is designed to power opamp-based preamplifier circuits and as such, has an output of approx 12-15V
DC depending on resistor selection choices.  The circuit should be good for about 250mA, depending on the transformer regulation.

## Construction

The board is very densely populated, and on both sides.  First assemble all the surface-mount components on the board front,
then install all through-hole components (except screw terminals) on the board rear.

### Component Sourcing

The major/expensive items required are:
* TE T9GS1L24-12 power relay
* HiLink 10M12 AC/DC switchmode brick
* Vigortronix VTX-120-5412-415 transformer for 15V operation
    * alternative: VTX-120-5412-412 for 12V operation
    * alternative: Block PT-13/2/12 which has a similar footprint
* Bourns PM3700 common-mode choke
* Rectifier filter caps: Panasonic EEEFK 3300u or similar 18mm square electrolytics
* Other silicon and passives - buy it on eBay from China

An 0805 passive sample book is very handy for filling out most of the circuit.

### Output Voltage

Output voltage is selected by voltage dividers on the LM317 and LM337.  
Choose R18 and R28 to be 240R or 270R, then select values for R20 and R23
so that

> R20 = R18 (VOUT / 1.25 - 1)

If R20 is not a convenient value, make a parallel combination e.g. 5k1 and 6k8
in lieu of 3k to get about 15V.

### Regulator Caps

Regulator output capacitors should preferably be tantalum, or aluminium electrolytic.
You can use either C6/C14 (SMD) or C7/C15 (through hole) footprints.  Make 
sure the capacitors are rated for notably more than the output voltage, 
e.g. 25V.

Ceramic caps here will cause regulator instability and oscillation.

### Pi Filter

The rectifier filter is a C-R-C pi-filter with 3300u per side.  Choose the resistors (R31, R26 etc) so that
the net voltage drop across them is as much as you can handle, without:
* bringing the unregulated DC rail so low that ripple breaks through the regulators, or
* the resistors overheating (max 100mW each) at your max design-load current

Larger resistors will provide greater ripple smoothing, at the expense of lower voltage
input to the regulators.  Choice will depend on your selection of transformer voltage, 
output voltage, and max expected load current.  A 15V supply has more room here for 
voltage drop (and therefore smoothing) than a 12V supply does.

Bridge them with 0R if you want a simple capacitor-only rectification filter.  Loading all
eight with 10R is probably OK for a 250mA output and will lose you about 0.65V DC and 160mW per rail.

### AC bits

The MOVs and common-mode choke are probably not critical.  Bridge over the 
choke if you're feeling EMI-antisocial, though this may result in HF conducted
EMI reaching your regulated outputs.

Use 500mA M205 slow blow glass fuses; there is one for each of the switchmode and
linear-regulated parts of the supply.  The switched-AC output is not fused.

### Grounding

There are three separate grounds:
* AGND, the centre of the dual-regulated supply
* DGND, ground for the 12V VDD switchmode/logic/control supply
* EARTH, a chassis safety connection

AGND is connected to EARTH via four opposed diodes in a bridge configuration,
which will keep them to no more than about 1.4V apart, but otherwise uncoupled
with respect to noise.

DGND is connected to EARTH via two opposed diodes and a capacitor.  They may
be up to 0.7V apart, and noise from the DC side of the 10M12 should be 
conducted into EARTH via C18.

These earth-lift circuits should be strong enough to trip an ELCB if there
is a transformer-isolation fault that brings mains AC power to the low
voltage side.

The Vigortronix transformers have exposed steel cores which must be grounded via
mounting screws through the PCB, which are connected via the PCB to the 
EARTH screw terminal.


## License

This hardware design is (C) 2021 William Brodie-Tyrrell and is licensed under the CERN OHL; see cern_ohl_v_1_2.pdf
