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
    * Seems to be the cheapest on digikey with ~500ma sustained current
    * Should switch fast enough
        * NTZD3152P may be a faster alternative, but tolerates less peak
        current (sustained currents is similar though)
* Board connectors
    * DigiKey
        * *Ph2ra-16-ua*
            * cheapest on digikey
            * mating connector from same manufacturer seems to be rs2br-16-g? but
            this is not readily available and of odd shape...
        * *pptc082ljbn-rc*
            * Cheapest mating connector on digikey
        * *Consider buying those from lcsc where they should be much cheaper*
    * LCSC
        * *2541WR-2x08P*
            * Extremely cheap
        * *PM254-2-08-W-8.5*
            * Mating part from same manufacturer: PZ254-2-08-W-8.5


### Notes
* There seems to be no cheap LED driver which supports row *and* column driving
* One option may also to use just a RP2040 per tile as they are so cheap
    * More development time required for firmware
    * Seems to only run at 12MHz without external oscillator
    * Restricts us to one microcontroller while with using the shift registers
    the driving microcontroller can be anything that can shift in the data

#### LED currents
* Use 5V supply
    * The TLC59283 are constant current and support up to 10V
        * The constant-current value of all 16 channels is set by a single external resistor
    * 3.3V may also work, but may be insuficcient for some chips, so use 5V
* Peak forward current is 60mA/90mA (green/red)
    * 16 columns (8 red 8 green) on one row so peak current of 1200 mA per row
* Average current is 25mA (same for both colors)
    * Total of 400mA average current per row (if all 16 channels are on)
    * A 500mA sustained/1.2A peak driver should be sufficient
    * In actual operation the rows are multiplexed so actually only about an
    eight of the average current is expected. So actually 400mA sustained
    current per tile if all LEDs are on.
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

## Mechanical Design
* Board interconnects using horizontal pin headers on the back of the board (tile is on the front)
    * 8 pins seem to fit just right, so two rows with 8 pins each seems to be a good choice
