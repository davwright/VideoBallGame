# Video Ball Game
A faithful reproduction of  May 1976 ["Video Ball Game"](https://archive.org/details/EA1976/EA%201976-05%20May/page/n39/mode/2up) by [Electronics Australia](https://en.wikipedia.org/wiki/Electronics_Australia) magazine.  
Thanks to the kind help of [Jon Stanley](https://www.electronixandmore.com/projects/pong/index.html), who also built this. John's [demo](https://vimeo.com/801468536).  

The files are [kicad](https://www.kicad.org/) schematics and PCB.

# Fixes
The [errata](https://archive.org/details/EA1976/EA 1976-07 July/page/n125/mode/2up) in July 1976 on page 125 list a few corrections.
* IC 5 is incorrectly shown as NAND on the schematic. Jon also discovered this independently.
* 2 0.0047uF polystyrene.
* There is also an error in a the RF circuit, but we are not building that.
* The Quad NAND and NOR integrated circuits today have diffirent pin configurations than those in 1976. This means the circuit board as is cannot be used.

# Improvements
* The 2200uF Capacitor is replaced with a 0.33uF capacitor and a 7806 6V regulator, that converts the 9V battery down to a stable 6V.
* 0.1uF decoupling capacitors are added to the VCC pins.
* pin headers are made for all switches and connectors so that they are all on one side of the board.
* ground and probe pins are added across the circuit for debugging.
* the board is printed double-sided, with a ground back plate, and removing the need for any jumper wires.
* the RF tuner circuit is completely removed, and the video signal is directly fed to composite video.
* includes Jon's 10MΩ fix for the flipflop on IC5 holding the ball's horizontal direction.
