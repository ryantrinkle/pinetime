# My PineTime notes

I'm documenting things I learn here, in case they're useful to other people.  Everything written here is correct to the best of my knowledge, but it's possible I've misunderstood some things.  Please be careful if trying this stuff with your own hardware - make sure everything makes sense to *you* before proceeding.  And, if you find any problems with what I've written, or just want to expand on it, I'd very much appreciate pull requests or issues!

## Unlocking PineTime

There's a tutorial [here](https://medium.com/@ly.lee/coding-nrf52-with-rust-and-apache-mynewt-on-visual-studio-code-9521bcba6004#1285) on how to unlock the nRF52 (i.e. PineTime) using a Raspberry Pi.  I don't have one of those lying around, but I do have an FT232H breakout board ([this one](https://www.amazon.com/gp/product/B07T9CPMHT), to be exact, but I suspect any FT232H would work pretty much the same; note that this board comes with the pins, but they are not soldered on, so you'll need to do that yourself).  FT232H [is listed](http://openocd.org/doc/html/Debug-Adapter-Hardware.html) as a "low-level adapter" for OpenOCD, which means OpenOCD is in full control of what gets sent to the target.  This is in contrast to ST-Link v2 adapters (like [these](https://www.amazon.com/gp/product/B01EE4WAC8)), which are "high-level adapters", meaning OpenOCD can only send commands that they support.

To physically hook up the FT232H to the PineTime, I followed the wiring diagram [here](https://github.com/arduino/OpenOCD/blob/c404ff5d3a2ec568daa106455845dd403b08dab4/tcl/interface/ftdi/swd-resistor-hack.cfg).  This does require one additional component: a 470Ω resistor.  I didn't have any resistors on hand, so I picked [this](https://www.amazon.com/gp/product/B072BL2VX1) up, which has lots of different resistors including 470Ω.

The pins names for the FT232H should be printed on the breakout board.  For the PineTime, it looks like this:
![PineTime SWD pins](https://wiki.pine64.org/images/2/29/PineTime_SWD_location.jpg).

Here's the circuit diagram with some additional notes.  Note that *two* of the FT232H's pins are connected to the PineTime's SWDIO pin, one with a resistor in between.

```
FTDI (FT232H)                         Target (PineTime/nRF52)
-------------                         -----------------------
1  - Vref   (+3.3V) ----------------- Vcc
3  - nTRST          -
4  - GND    (GND)   ----------------- GND
5  - TDI    (AD1)   ---/\470 Ohm/\--- SWDIO
7  - TMS            -
9  - TCK    (AD0)   ----------------- SWCLK
11 - RTCK           -
13 - TDO    (AD2)   ----------------- SWDIO
15 - nSRST          - - - - - - - - - nRESET (doesn't exist on PineTime; just leave it disconnected)
```

After hooking things up this way, I was able to invoke run `sudo openocd -f openocd_nrf52.cfg` (with [the `openocd_nrf52.cfg` in this repository](./openocd_nrf52.cfg)) and it successfully connected to the PineTime.  Note that that `.cfg` file is probably not minimal - I stopped editing it when it worked.

At that point, I was able to pick up from [`telnet localhost 4444`](https://medium.com/@ly.lee/coding-nrf52-with-rust-and-apache-mynewt-on-visual-studio-code-9521bcba6004#66e2) in Lup Yuen Lee's post, and everything went smoothly!
