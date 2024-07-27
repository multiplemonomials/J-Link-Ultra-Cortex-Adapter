# J-Link Ultra Cortex Adapter

J-Links, as debug probes, are amazing.  They can talk to just about any processor, ARM or otherwise, in common use in the last decade.  The huge breadth of software and hardware compatibility available from one debug probe plus a set of adapters is incredible, and SEGGER deserves credit for getting this right.

Except... one thing.  The adapters they sell for connecting a J-Link to a 9-pin or 19-pin Cortex-Debug connector are just... bad.  They are bad, in fact, in several ways:
- They are expensive.  $40 for a tiny PCB with two connectors on it is too much.
- They don't connect the VCP (Virtual COM Port) pins.  Having a UART available to talk to the target device is a super useful feature of newer J-Links, but the design of these adapters just ignores it.  In fact, they don't even break the pins out, so they it's impossible to access without sketchy bodge wiring.
- The ribbon cable connectors have no keying, and the included ribbon cables don't either.  I have lost count of the number of times I've plugged in a debug cable backwards because there's nothing preventing one from doing so.
- Each $40 adapter only comes with one connector soldered: the 9-pin or the 19-pin.  Unless you have the knowledge to source and solder your own connector and cable, this means you need to buy two different adapters for working with 9 and 19-pin devices.

The Ultra Cortex Adapter in this repo tries to fix this badness in several ways.

First of all, this adapter uses proper keyed connectors for each of the ARM Cortex connections.  And these aren't just connectors with a little keying tab that can break off -- these are fully shrouded connectors that can take a beating and keep working.  

(note: the ARM standard says that the connections should be keyed via a missing pin 7 on the header and a filled pin 7 hole in the cable.  However, I've had a hard time sourcing this, so I've gone for the next best thing.  Plus, this way if you replace the cable the connector will still be keyed.)

Additionally, this adapter gives you not one, not two, but *three* different connection options:
- Standard 9-pin Cortex Debug
- STMicro 14-pin STDC14
- Standard 19-pin Cortex Debug

Last but not least, the Ultra Cortex Adapter gives you access to all of the JTAG/SWD/VCP signals via convenient test points!  Because you never know when things are just going haywire and you need to be able to hook in a logic analyzer or oscilloscope, and it absolutely sucks to not have a convenient place to do that.

## ARM Debug Connector FAQ

- Q: Why are there so many GND lines on the connectors?
  - A: This is to create some level of electrical isolation between the JTAG lines by putting a GND conductor between each of the signal lines on the ribbon cable.
- Q: How to protect JTAG/SWD pins so that random electrical noise doesn't cause something to happen on the JTAG port of the MCU?
  - A: If your MCU has a TRST pin, one of the easiest ways to do this is to connect it to the debugger pull the TRST pin to ground with a weak pulldown (10k?).  This ensures the JTAG TAP is held in reset at all times, except when a debugger is plugged in and brings TRST high.  However, TRST is not mapped on the standard ARM Cortex debug connectors.  SEGGER's adapters offer the option to convert the GNDDetect line into TRST, and that is supported by this adapter, but it might break compatibility with other standard-compliant debuggers.
  - If you don't have TRST, the [ARM recommended method](https://developer.arm.com/documentation/101416/0100/Hardware-Description/Target-Interfaces/Cortex-Debug-ETM) is to use 100kOhm resistors to pull up TMS and TDI to logic high and pull down TCLK to logic low.
- Q: What's the deal with the GNDDetect pin?
  - A: It's not documented well, but as far as I can understand, the idea of this pin is that it's pulled high on the board side and grounded on the debugger side.  This allows the board to know if a debugger is plugged in or not.  Unfortunately, SEGGER slightly disrupted this by leaving GNDDetect as NC by default, with a solder bridge option to connect it to TRST or GND.
- Q: How do we get Virtual COM Port on the 19-pin connector?
  - A: Unfortunately, even with 19 pins available, ARM didn't define a standard way to add UART to this connector.  Sadness.  And with SEGGER's default mapping (VCP_TX = TDI, VCP_RX = DBGREQ), the VCP_RX pin doesn't make it to *any* pin on the 19-pin connector.  To work around this, this adapter defines a custom mapping of VCP_RX to pin 7, the normally unused key pin on the connector.  This mapping is enabled by default, but can be disabled via cutting a jumper if you are using pin 7 for something else on your board.

## Choosing a Debug Connector for Your Board

With the large variety of debug connectors available, it can be difficult to determine the one you need for your application.  I've put together the below table to summarize which ones possess which features.

| Feature | 9-Pin Cortex Debug | 14-Pin STDC14 | 19-Pin Cortex Debug |
|---|---|---|---|
| JTAG | Yes | Yes | Yes |
| SWD | Yes | Yes | Yes |
| UART / Virtual COM Port | Only via nonstandard mapping. Not supported at the same time as JTAG. | Yes. Not supported at the same time as JTAG with J-Link. | Only via nonstandard mapping. Not supported at the same time as JTAG or with the J-Trace. |
| JTAG RTCK (return clock) | No | Yes | No |
| JTAG TRST | Only with SEGGER nonstandard mapping.  Requires changing a solder jumper on this adapter. | Only with SEGGER nonstandard mapping.  Requires changing a solder jumper on this adapter. | Only with SEGGER nonstandard mapping.  Requires changing a solder jumper on this adapter. |
| GNDDetect (ability for board to detect if a debugger is connected) | Yes (unless TRST is used) | Yes (unless TRST is used) | Yes (unless TRST is used) |
| Target Power | No | No | Yes |
| ETM (Embedded Trace Macrocell) | No | No | Yes |