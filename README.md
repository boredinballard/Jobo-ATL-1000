This may be niche, but for anyone with a Jobo ATL 1000 here, you may find this useful in the future. I won't go over all the programs, just the basics and info on what the hex code is doing.

The ATL 1000 programs are stored on an ST M27C256B EPROM. Mine came with a M27C256B-12XFI. I currently have a M27C256B-12F1 in it. I've seen -15F1 parts in other ATL 1000 controller heads. It probably doesn't matter. I've read that someone successfully used an M27C512 EPROM and doubled the amount of programs available to them.

If you would like to program your ATL 1000, you'll need a UV eraser and an EPROM programmer. I suggest getting a used or NOS eraser from eBay, not the cheap new nonsense from Amazon. It's up to you on how you want to program the EPROM, but I used a programmer from MCU Mall.

I highly suggest you by an EPROM and program that and do not erase the original EPROM. Make a backup of it using your programmer.

Basic program info:

* Programs 1-8 is tempered and fills the bath. Temp is set to 38°C.
* Programs 9-15 is room temp and does not fill the bath.
* Program 16 is cleaning. Does not fill the bath.

The microcontroller reads the EPROM at it's selected program (I don't know if you can move the programs to different addresses in the memory) and each hex byte corresponds to an instruction that runs for about 2.2 seconds or so. The program ends when it reaches **E0**.

The first hex digit corresponds to a chemical tank, or the pre-wash/wash of the respected tank/step. The second hex digit corresponds to an action, such as rotate, lift, pumping, etc. 

Tank codes (first digit):

    Rinsing/washing step
    3x - Tank 1
    5x - Tank 2
    7x - Tank 3
    9x - Tank 4
    Bx - Tank 5
    Dx - Tank 6

    Chemical step
    2x - Tank 1
    4x - Tank 2
    6x - Tank 3
    8x - Tank 4
    Ax - Tank 5
    Cx - Tank 6

Action codes (second digit):

    x0 - Select or rotate (?)
    x8 - Water intake for drum
    x2 - Lift/drain
    x4 - Pumping of selected tank

Other codes:

    10 - Pre-rinse
    x1 - Moves rotary valve (?)
    00 - No operation

For example, **20** selects **Tank 1** in the **Development** step. It may also signal the rotation start/stop. **24** begins pumping of **Tank 1**. To lift and drain on this step, the code is **22**. Each step through the process corresponds to a tank and/or step. They also correspond to a rinse or dev step, and I'm not sure if the distinction matters, but it is there. For example, in the C41 program, once you get to **Tank 2**, the code for pumping the chemicals is **44**, and lifting/draining is **42**. There are some other details in-between switching steps that I'll go over later.

Here is an example of the first line of the cleaning program (16):

    D0 B0 90 70 50 30 00 00 24 24 24 24 24 24 24 24

* **D0 B0 90 70 50 30** - Select each tank in a **rinse** step, one after the other. The light for each tank lights up on the controller for about 2 seconds.
* **00 00**  - No operation for about 4 seconds
* **24 24 24 24 24 24 24 24** - Begins pumping from **Tank 1** in the **Development** step. Lasts about 15 seconds.

Here is the next line:

    20 22 22 22 22 22 20 21 20 21 20 44 44 44 44 44

* **20** - I believe stops rotation. Still on the **Development** step of **Tank 1**.
* **22 22 22 22 22** - Lifts the drum and drains for the **Development** step of **Tank 1**.
* **20** - Starts rotation again, and lowers the lift. I believe the lift lowers whenever it receives any new command.
* **21** - Still a question, but apparently moves the rotary valve.
* **20** - Continues to rotate.
* **21** - Rotary valve again.
* **20** - Continues to rotate.
* **44 44 44 44 44** - Begines pumping from **Tank 2** in the **Development** step.


And so on, and so on. Regarding the **Rotary valve**, I'm not sure how it works. You can hear it doing something, and the bytes of **x0 x1 x0 x1 x0** Seems to happen between every step. I also noticed that every program always runs through these rotary valve steps six times, even if only two tanks are used. It will run through them at the end of the program to if it didn't use all steps/tanks. For example, here is the end of the the two bath C41 program (7):

    52 50 60 61 60 61 80 81 80 81 A0 A1 A0 A1 C0 C1
    C0 C1 C0 E0 00 00 00 00 00 00 00 00 00 00 00 00

* **52 50** - Just finished lift/drain steps for **Tank 2** in the **Rinse** step.
* **60** - Selects **Tank 3** in the **Development** step.
* **61** - **Rotary** valve during the **Development** step of **Tank 3**.
* **60 61** - Repeat of above steps
* **80 81 80 81** - Same as above, but for **Tank 4** in the **Development** step.
* **A0 A1 A0 A1** - Same as above, but for **Tank 5** in the **Development** step.
* **C0 C1 C0 C1 C0** - Same as above, but for **Tank 6** in the **Development** step.
* **E0** - End of program.

I don't know how important the rotary valve is to have go through each step, during testing I did not do that and didn't run into any issues, but I suppose I'll find out if I broke something when I attempt to actually develop film with a custom program.

Here's an example cleaning program that only cleans drains tanks 1-3:

    00 00 00 70 50 30 00 00 24 24 24 24 24 24 24 24
    20 22 22 22 22 22 20 21 20 21 20 44 44 44 44 44
    44 44 44 40 42 42 42 42 42 40 41 40 41 40 64 64
    64 64 64 64 64 64 60 62 62 62 62 62 60 61 60 61
    60 70 70 78 78 78 78 70 70 70 70 70 70 72 72 72
    72 72 70 70 78 78 78 78 70 70 72 72 72 72 72 70
    80 81 80 81 A0 A1 A0 A1 C0 C1 C0 C1 C0 E4 00 00

The last few lines it is doing a rinse of the drum, and then it moves the rotary valve a few times, matching up with how other programs behave at the end.
