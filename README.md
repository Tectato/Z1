# Z1
Digital recreation of Konrad Zuse's Z1 reconstruction, to be run using the Z1 Simulator (https://tectato.itch.io/z1-simulator)

# DISCLAIMER
This project makes no claim to be a 1:1 identical twin to the physical machine, due mainly to the fact that so far the data to make that possible has not been published. This recreation is based primarily on the technical drawings available on the Konrad Zuse Internet Archive (https://zuse.zib.de/) and photos of the machine. Numerous changes had to be made to enable the digital version to operate correctly, these are marked in each scene through comment boxes and/or through suffixes (usually -) in sheet filenames. Changes to the machine's microcode will be listed below (WIP)

It should further be noted that, in order to facilitate timely progress, the sheets were digitised only approximately. Hole diameters were rounded to the nearest full milimeter and outlines to the nearest half milimeter. Should one intend to create physical machines from these files, be sure to refer to the original plans to verify the dimensions first, or produce smaller test pieces and check their fit and interaction before mass-producing parts.


# Microprogram changes:

- DEC2BIN (↗):
To prevent an overflow of the mantissa after too many multiplications by 10 when factoring in the input radix slider, an extra phase has been added between phases 9 and 10, where the mantissa is shifted down by two digits before the multiplications start. The constant that the exponent is set to has been changed from 15 to 17 to compensate. Also, the adjustment to the right in the original phase 10 has been swapped to a leftwards adjustment equal to phase 8, and the check for u7 was moved to a new phase, since the adjustment may need more than one cycle. These changes are identical to the "↗P5" patch on https://zuse-z1.gitlab.io/ALU-Sim/

This is the best option we could identify until now. While the addition unit could instead be easily modified to only perform the multiplication by 10 if Be+1 is zero and a down-shift otherwise, the core issue is that the movement of the radix slider is still controlled by the microprogram unit, and would still be performed even if no multiplication by 10 was executed.