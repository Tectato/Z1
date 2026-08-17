# Z1
Digital recreation of Konrad Zuse's Z1 reconstruction, to be run using the Z1 Simulator (https://tectato.itch.io/z1-simulator)

For the repository: the `main` branch is where development happens, and, while functional, contains a lot of floating and unused parts. Use the `clean` branch for a more true-to-life view.

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

/!\ Due to the way the algorithm for the output instruction has been fixed (↘P8), the decimal point may be shown several places too far to the left, but the digits should be correct. Fixes are being investigated.

/!\ The machine cannot handle zero or infinity. The input and subtraction operations contain a normalizing step, where the mantissa is shifted up until it starts with a 1. If it's all zero, this phase never finishes. An emergency stop button is built into the microprogram unit which you can right-click during clock step III to abort the current operation.

/!\ The machine has no over- or underflow detection in the exponent addition unit. For instance, squaring 9999 * 10^6 will result in a near-zero value, as the exponent overflows during multiplication. Keep your calculations reasonable and you should be fine.

F and G are only reset when an arithmetic operation finishes, so a sequence of reading and writing to memory to copy/move data around will fill both registers, and then keep combining more data into G. To clear them again, run some operation like addition and write the result to an unused address.


# Microprogram changes:

- DEC2BIN (↗):
To prevent an overflow of the mantissa after too many multiplications by ten when factoring in the input radix slider, an extra phase has been added between phases 9 and 10, where the mantissa is shifted down by two digits before the multiplications start. The constant that the exponent is set to has been changed from 15 to 17 to compensate. Also, the adjustment to the right in the original phase 10 has been swapped to a leftwards adjustment equal to phase 8, and the check for u7 was moved to a new phase, since the adjustment may need more than one cycle. These changes are identical to the "↗P5" patch on https://zuse-z1.gitlab.io/ALU-Sim/

This is the best option we could identify until now. While the addition unit could instead be easily modified to only perform the multiplication by 10 if Be+1 is zero and a down-shift otherwise, the core issue is that the movement of the radix slider is still controlled by the microprogram unit (which has no access to the state of Be+1), and would still be performed even if no multiplication by 10 was executed.

- BIN2DEC (↘):
To solve the same overflow issue, aforementioned modifications to the addition unit were made here, since in the case of very small exponents (minimum value -64), up to 22 multiplications by ten may be necessary, and shifting the mantissa down far enough to prevent an overflow would not be feasible. The decimal point slider will still move when a shift instead of a multiplication occurs, so may end up too far to the left.

Photos and plans indicate that the same approach was used to fix the issue on the real machine, but with an additional fix for the slider issue by adding extra functionality to the S1 output pin of the exponent addition unit and reusing the unused Be=>Bg pin of the microprogram unit, but exactly what this fix looks like has yet to be determined.