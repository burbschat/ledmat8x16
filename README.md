# *LTP-12188M-08* Based Two Color 8x16 LEDs in 8x8 Slots LED Matrix Display Module

## Motivation
Found a place that sold their dead stock of *LTP-12188M-08* modules for
extremely cheap and bought a whole bunch.
To drive those I would like to have a minimal and cheap solution which is also
modular, meaning I can build an (for any practical purposes) arbitrarily sized
display by just assembling identical modules (and perhaps a few cables etc.).

## Pictures
### Renders
Had some fun with [this incredible tool](https://github.com/30350n/pcb2blender).
| Front                                                              | Back                                                              | Oblique
| ------------------------------------------------------------------ | ----------------------------------------------------------------- | ---------------------------------------------------------------------- |
| ![Front Render](./figures/render/ledmat_render_front_nomodule.png) | ![Front Render](./figures/render/ledmat_render_back_nomodule.png) | ![Oblique Render](./figures/render/ledmat_render_oblique_nomodule.png) |

### Prototype
Testing three prototype modules driven by a Raspberry Pi Pico (which is
debugged from a second Pico via SWD).
The colors do look much more vibrant in person. Also, the apparent differences
in brightness are an artifact of the way this picture was captured.
![3 Module Test Setup with Pico](./figures/photos/3module_test_pico.jpg) 

## Parts
* *LTP-12188M-08* (8x8 two-color (red/green) LED dot matrix)
* *TLC59283* (16 channel constant current LED driver with pre-charge FET)
    * Seems to be cheapest LED driver available
    * No dimming supported but perhaps possible by updating the frames at a high enough rate
    * Can be daisy chained
    * Up to 32MHz operation advertised
        * One frame is 8 rows x 16 bits = 128 transfers. At 32MHz one transfer
        takes 3.125e-08s so 4e-6s per frame per tile. For 60 tiles thats
        240e-6s for the whole panel. If we want 60 frames per second
        (16.666e-3s per frame), we can sub-divide each frame into 69 sections
        for which a pixel can be either on or off. Thus a 69 level brightness
        adjustment may be possible.
* *DMP31D7LDW-13* (dual p-channel enhancement mode MOSFET)
    * Seems to be the cheapest on DigiKey with ~500ma sustained current
    * Should switch fast enough
        * NTZD3152P may be a faster alternative, but tolerates less peak
        current (sustained currents is similar though)
* Board connectors
    * DigiKey
        * *Ph2ra-16-ua*
            * cheapest on DigiKey
            * mating connector from same manufacturer seems to be rs2br-16-g? but
            this is not readily available and of odd shape...
        * *pptc082ljbn-rc*
            * Cheapest mating connector on DigiKey
        * *Consider buying those from lcsc where they should be much cheaper*
    * LCSC
        * *2541WR-2x08P*
            * Extremely cheap
        * *PM254-2-08-W-8.5*
            * Mating part from same manufacturer: PZ254-2-08-W-8.5

### Notes
* There seems to be no cheap LED driver which supports row *and* column driving
    * IS31FL3731 seems to be pretty capable. Costs a little more but is still
    affordable. However, choosing the simpler part will surely teach one more
    things than just buying the solution.

#### LED Currents
* Use 5V supply
    * The TLC59283 are constant current and support up to 10V
        * The constant-current value of all 16 channels is set by a single external resistor
    * 3.3V may also work, but may be insufficient for some chips, so use 5V
* Peak forward current is 60mA/90mA (green/red)
    * 16 columns (8 red 8 green) on one row so peak current of 1200 mA per row
* Average current is 25mA (same for both colors)
    * Total of 400mA average current per row (if all 16 channels are on)
    * A 500mA sustained/1.2A peak driver should be sufficient
    * In actual operation the rows are multiplexed so actually only about an
    eight of the average current is expected. So actually 400mA sustained
    current per tile if all LEDs are on.
    * Eventually set the current limit at the TLC59283 to around 25mA
        * 2k resistor should give 26.5mA, which is alright
* Want row drivers separately for each tile to avoid driver current scaling
with number of tiles
    * Make sure that connections between tiles support high enough currents
* Consider [MOSFET arrays](https://www.digikey.com/en/products/filter/transistors/fets-mosfets/fet-mosfet-arrays/289)
as we need 8 of them per tile
    * Need P-channel to drive by setting gate voltage low (below source voltage)
    * Maybe not too long rise/fall times 
    * DMP31D7LDW-13 seems fine (20JPY at 200 pieces, doesn't seem to get much cheaper on digikey)
        * There are many SOT-363 options so if this one does not work, one can
        probably find a pin compatible alterative
    * [Search on digikey](https://www.digikey.jp/en/products/filter/transistors/fets-mosfets/fet-mosfet-arrays/289?s=N4IgjCBcoEwAwHYqgMZQGYEMA2BnApgDQgD2UA2iAgMwAcCALEsWDPHHCC2xzCALrEADgBcoIAMoiATgEsAdgHMQAX2IA2OLWQg0kLHiKkKIGGFoBWauq6nrNBrZj2rThq2p9iMBvHXU3ajAATgDvBiDgm2IEdVoYC2CnCwtLMNMUy21vVNiLZODguGjTdWc4p38IkpgEGGDUpwQLLUdvWlC2JyKEUO6YeLbTYIGE7uCwFNswEIQ62wZaScSFsGavEHdm9MXl7M2WmlsWos5iCwGwEpS4d2OWxvPGanTE53TNEYgNWvrp9WawSGV1iYHSrABYGBMEh%2B1YtCY%2BW4CLm01qCCWaOCtABWPo33AQSsBLBkwYwKJDCSLBe7g2YIYiTh1g4JTBdQq3jgkyBTm5ZXS8CuL2mCHMCNFAz6LDqHTZssKouoUWBjCiJLV8xlmkx2q08o5SJA8TycIxMKNayKUxl1up4FovGYDo4nictRCGzYdUQfM0NtMxTgFmd2xoBL8jpqiHJ9vgjBWXPidD5gyGvVY5lTCSN8BGCDOpjA3LBTmLM2dZmK8TLYM5RfZhZ8c2Kbjm1CGzcYleoWi0TiC1F6tlo1CsZQHzhCk5xPYY3M7HfccfUZXUEdXA32MP8t2S9VxOXnk2STHnp8YEcdBe3cVojsqIwf3gL7kFr66L7Wve6Q58v5oXNQnWJxRwRDZe0WRViEg61bFiVdfRg25QjjKIUiGHF6GdfwrUtBl1GBXw4CKf4ZjWaZRypcFRxaQtzDHeBklXGYBGEMRIEkGQFGUNQQAAWj4aBdCgGQAFcjDISBKCNJJ%2BD4-ikmEvRxMkkwIHkvibGE2QABNxH48tbFEcRbBEABPIR8HETBcDQFQVCAA)

#### Clock
The clock will have to arrive at each TLC59283 before the output of the
previous one had a chance to change. According to the data sheet, the delay
until SOUT changes is normally around 11ns. As we are only concerned about
adjacent modules in close proximity, the clock propagation delay should be well
below 10ns.
A possible concern mentioned [here](https://www.eevblog.com/forum/projects/shift-register-delay-tlc59283/msg5714097/#msg5714097)
is that the clock signal integrity may be degraded due to large capacitance if
we fan it out too far. To solve this we may employ a buffer, this however would
mean we need either a very fast buffer to not skew the clock relative to the
data signal too much, or also have to buffer the data signal at the same time.
Buffering two signals should be doable.
This is probably not an issue, but technically if the buffers are to slow, this
could limit the rate at which we can operate the panel (i.e. clock in data).
As, if it works, no buffers should be the easier solution, for now I'll try
without. If required, we may try to add clock line terminations to avoid
ringing.

## Mechanical Design
* Board interconnects using horizontal pin headers on the back of the board (tile is on the front)
    * 8 pins seem to fit just right, so two rows with 8 pins each seems to be a good choice
    * These also will provide mechanical support within each row
* Consider some sort of locating feature to use to locate modules in maybe a
3d-printed back plane or with small clips to be inserted between modules (**TODO**) 
    * Could shave off the corners and have short square (45 deg. rotated) posts on the back plane
    * High symmetry is probably a good thing so locating features on maybe only
    top/bottom may not be as useful as the shaved off corners
    * To assemble multiple rows such a locating feature would be required
    * For a large display with many rows and columns a backplane is probably mandatory
    
## Heat Dissipation
Found that when I run a fully lit frame (all 16 LEDs enabled while multiplexing
the rows) the whole module gets quite hot. Not entirely sure what produces the
heat. If it is the TLC59283, one could simply remove the solder mask on the
back side of the board which is a ground plane to which the ground pad on the
TLC59283 is connected to and place some sort of heat sink there.
This is of course the reason why the TLC59283 is placed under the LED matrix
module on the front and not at the back of the board and not simply attributed
to be not checking the orientation of the modules footprint before doing the
tedious routing which I do not want to do twice.
Will have to further check what is the source of the heat. However, as long as
the display is not constantly fully illuminated, it does not appear to get
excessively hot.

## Driving the Modules
I use an Raspberry Pi Pico (2 to be exact) to the modules. The serial interface
for the TLC59283 is implemented using a PIO state machine (with DMA) which can
clock out serial data independent of the processor cores and thus is a very
performant solution. Switching between the rows is done by the processor which
also feeds data from a global frame buffer to the DMA (which feeds it to the
PIO state machine).
Other microcontrollers will probably also work. Not sure how one would handle
the serial communication though. AT a slower rate one can perhaps just bit bang
using GPIO. Didn't try.
The code I use on the Pico is available in a separate repository [here](https://github.com/burbschat/ledmat8x16pico/).
