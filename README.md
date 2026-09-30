# Z1
Digital recreation of Konrad Zuse's Z1 reconstruction, to be run using the Z1 Simulator (https://tectato.itch.io/z1-simulator)

For the repository: the `main` branch is where development happens, and, while functional, contains a lot of floating and unused parts. The `clean` branch removes those and places the machine's sections closer to their intended positions.

The vertical scale is displayed greatly exaggerated. You can set the "sheet spacing" in settings to 0.021 for something closer to reality, but keep in mind that the placement of comment boxes assumes a spacing of 0.045

# DISCLAIMER
This project makes no claim to be a 1:1 identical twin to the physical machine, due mainly to the fact that so far the data to make that possible has not been published. This recreation is based primarily on the technical drawings available on the Konrad Zuse Internet Archive (https://zuse.zib.de/) and photos of the machine. Numerous changes had to be made to enable the digital version to operate correctly, these are marked in each scene through comment boxes and/or through suffixes (usually -) in sheet file names. Changes to the machine's microcode are listed below (WIP)

It should further be noted that, in order to facilitate timely progress, the sheets were digitised only approximately. Hole diameters were rounded to the nearest full milimeter and outlines to the nearest half milimeter. Should one intend to create physical machines from these files, be sure to refer to the original plans to verify the dimensions first, or produce smaller test pieces and check their fit and interaction before mass-producing parts.

# File overview
- Z1: Entire Z1. Will take a while to load.
- Z1Rechenwerk: Z1 Processor half. Can run instructions autonomously if you set the registers via the value interface and the opcode during clock step III
- Z1MemoryFull: Z1 Memory half.
- Z1Memory_DataProgrammable: Memory that can load, store, and move data through programs in the `Sequencer` tab of the side window.

# Working with the Z1
/!\ The simulation can't run arbitrarily fast, and may break if the clock speed is too high for your machine to handle! If red and yellow lines start appearing (except on the tape reader), something has gone wrong. In this case, reload the scene (File > Reload current scene) and try again with a slower clock speed.

Programs can be written in the `Programming` tab of the side window (Press N or click the arrow on the right edge of the screen to toggle). Click the `Selected` checkbox next to a program name to select it and run the simulation to step through it. The reset button on top of the tape reader can be used to jump back to the first instruction.

The Machine contains two operand registers, F and G. Loading from memory twice will fill both, a following arithmetic instruction will operate on those two values and put the result either in F or write it to memory, if followed by a store operation.

When the machine reaches an input instruction, it will remain idle until the user input is confirmed. Stop the clock, set your value by pulling the digit tabs on the keyboard (right click), set the decimal point (right click on the red buttons next to the slider), and pull the lever with the ↗ arrow, then start the clock again.

/!\ You can't reset the input digits by just pushing the tabs back in, use the clearing lever (to the right, marked `Lö`) instead!

Reading an input always writes to F, so if you intend to add/subtract/multiply/divide two input values, write the first one to memory before reading in the next.

If a program contains two output instructions, the second one will not start until the display has been cleared, use the lever with the ↘ arrow for that. The machine further treats an output instruction like any other arithmetic operation, and will write the "result" (consisting of all zeroes) to register F. While this has no effect on the content of F, the control unit still considers the register as "full" and a subsequent load from memory will target G. To work around this, write to an unused address after an output operation.

/!\ The machine cannot handle zero or infinity. The input and subtraction operations contain a normalizing step, where the mantissa is shifted up until it starts with a 1. If it's all zero, this phase never finishes. An emergency stop button is built into the microprogram unit which you can right-click during clock step III to abort the current operation.

/!\ The machine has no over- or underflow detection in the exponent addition unit. For instance, squaring 9999 * 10^6 will result in a near-zero value, as the exponent overflows during multiplication. Keep your calculations reasonable and you should be fine.

F and G are only reset when an arithmetic operation finishes, so a sequence of reading and writing to memory to copy/move data around will fill both registers, and then keep combining more data into G. To clear them again, run some operation like addition and write the result to an unused address.


# Microprogram changes:

## DEC2BIN (↗):
To prevent an overflow of the mantissa after too many multiplications by ten when factoring in the input radix slider, an extra phase has been added between phases 9 and 10, where the mantissa is shifted down by two digits before the multiplications start. The constant that the exponent is set to has been changed from 15 to 17 to compensate. Also, the adjustment to the right in the original phase 10 has been swapped to a leftwards adjustment equal to phase 8, and the check for u7 was moved to a new phase, since the adjustment may need more than one cycle. These changes are identical to the "↗P5" patch on https://zuse-z1.gitlab.io/ALU-Sim/

This is the best option we could identify until now. While the addition unit could instead be easily modified to only perform the multiplication by 10 if Be+1 is zero and a down-shift otherwise, the core issue is that the movement of the radix slider is still controlled by the microprogram unit (which has no access to the state of Be+1), and would still be performed even if no multiplication by 10 was executed.

## BIN2DEC (↘):
The output operation keeps multiplying by ten until the exponent becomes greater or equal zero, so up to 22 such multiplications may be required (minimum exponent value is -64). Thus a dynamic solution is necessary, since shifting the mantissa down far enough to prevent an overflow would not be feasible.

New insights from documents archived at the Deutsches Technikmuseum in Berlin have revealed a likely fix, documented in the following. Our previous approach (↘P8 on https://zuse-z1.gitlab.io/ALU-Sim/) is still described further below for reference.

### Current patch
Phase 2 of the output operation was split into two criteria, 73a and 73b, where the former executes a multiplication by ten and the latter performs a corrective shift downwards if necessary. Which of the two is active is determined by the flag S1, where one criterion sets it high and the other low. Initially, there was no signal other than Lz (end of an operation) which could set S1 back to zero; instead, the Be=>Bg output of the microprogram unit was repurposed for this.

Photos show that the line carrying the S1 signal from the ALU's control outputs to the microprogram unit has been fitted with an extra hook towards the Be=>Bg pin, and the topmost layer of the distributor block features an extra linkage such that this signal line mirrors the state of S1 and can reset it as well.

### Previous patch (P8, no longer implemented)
In this patch, aforementioned modifications to the arithmetic unit to dynamically switch between multiplication and shifting were made. To address the issue of the microprogram unit moving the slider even during an adjustment, that movement was made conditional and controlled by the arithmetic unit.

The six output pins on the side of the exponent half of the arithmetic unit show up in several plans, but solely in the control unit's (which were drafted after the other occurences), the pin setting the S1 flag is marked "S1(d3)". The sheet in the exponent unit instructing the mantissa unit to add Be/4 further has a hook to pull on this S1(d3) pin, even though the microprogram would not call for an S1 signal at this point. Photos show that the line connecting this pin to the signal distributor of the microprogram unit has had an extra hook attached to it, which acts to reset the Be=>Bg pin. Said pin was never connected to the control unit (the function instead being coalesced into the equivalent output of the exponent adder), as such could be reused for a conditional d3 signal (d3 moves the output slider leftwards). Be=>Bg is regularly reset in step II already, but the S1(d3) signal would arrive earlier in step IV.

This approach adds a relay to the basement section, to add a conditional step I impulse line, connected to the rotating element which executes a shift of the output slider. Both the original d3 signal and this impulse line need to activate for a leftwards shift to occur. The relay is closed whenever Be=>Bg is in its resting state, and we've added sheets to activate Be=>Bg during the multiplication phase of BIN2DEC.

The two cases go as follows:

Case A (Be+1 is set, corrective shift instead of multiplication):

- III: Microprogram unit activates d3 and Be=>Bg, relay is opened
- IV: Shift is performed, S1(d3) does not activate
- I: Relay is open, so output slider does not move
- II: d3 and Be=>Bg are reset, relay is closed

Case B (Be+1 not set, multiplication can occur):

- III: Microprogram unit activates d3 and Be=>Bg, relay is opened
- IV: Multiplication is performed, S1(d3) is activated and resets Be=>Bg
- I: Relay is closed, output slider moves to the left
- II: d3 is reset, Be=>Bg already in resting state

The rotating element which is now controlled by the new relay also executes shifts to the right, but since the relay is closed in its resting state and Be=>Bg is regularly reset, this operation is not affected. Likewise, since the original d3 still has to be active for the shift to occur, S1(d3) being activated during other operations like the Sum has no effect on the output slider.
