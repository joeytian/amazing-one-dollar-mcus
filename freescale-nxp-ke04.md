![img](https://jaycarlson.net/wp-content/uploads/2017/09/20170928-8484-1.jpg)

FREESCALE (NXP)
KINETIS KE04

https://jaycarlson.net/pf/freescale-nxp-ke04/

September 15, 2017  Microcontrollers  [8](https://jaycarlson.net/pf/freescale-nxp-ke04/#)

The Kinetis E series, part of NXP’s acquisition of Freescale, is a line of 5V Arm processors that use Cortex-M0+ and Cortex-M4 cores. The E series is optimized for harsh, noisy environments, and is built for primarily for updating 8-bit mid-range designs — though the KE1xF series reaches out above 160 MHz with a Cortex-M4, making it a decidedly high-performance part (and probably the only 5V-compatible part, other than the [Cypress FM4](http://www.cypress.com/products/fm4-32-bit-arm-cortex-m4-microcontroller-mcu-families), that can go that fast).

These are older parts now — while they were extremely inexpensive when they were released, they don’t compete as well with newer parts at pure pricing/peripheral comparisons.

What you’re paying for, in my opinion, is the Processor Expert workflow. The Freescale Kinetis parts were my first Arm experience, and I think Processor Expert was a major factor in helping me get comfortable with the complexity of clock and peripheral initialization and use.

I’ve narrowed in on the KE04 for this review, which is the 48-MHz KE-series part, but there’s also a KE02 available in 20- or 40-MHz configurations that have nearly identical peripheral sets and typically comes in bigger packages with more memory (and costs more).

The KE04 is built on a larger process that doesn’t have particularly good power consumption figures; having said that, the larger process makes it less leaky, which allows the KE04 it to remain powered in stop mode — with a 1 kHz oscillator running — at only 2 μA of current consumption.

- 48 MHz Cortex-M0+
- 8 KB of flash, 1 KB of RAM
- 2.7 – 5.5V operation
- 32 kHz and 1 kHz internal oscillators
- 48 MHz FLL
- Five timer units that support 10 capture channels, 8 PWM outputs, plus pulse-width timing
- Real-time clock
- 12-channel, 12-bit ADC
- Two analog comparators

 

![img](https://jaycarlson.net/wp-content/uploads/2017/09/Acrobat_2017-08-04_13-27-50-1024x618.png?cbc196&cbc196)The KE04 clocking system is simple and somewhat inflexible. Because most peripherals are clocked off the bus/flash clock, which is always at or below the core clock, it’s impossible to snooze the CPU — critical for high-power Arm cores.

# CORE & PERIPHERALS

The KE04 is a Cortex-M0+ Arm microcontroller; I discuss this core in detail in my [main article](https://jaycarlson.net/microcontrollers/#arm).

Clocking on this microcontroller is simple: the primary clock source is an internal low-speed RC oscillator driving an FLL with a fixed frequency multiplier of 1280. This RC oscillator can be tuned between 31.25 and 37.5 kHz, resulting in a frequency output of 40-48 MHz. There’s a basic binary divider supporting 1-128x reduction in powers of 2. This hits the CLK_GEN module which has a natural number prescaler of 1, 2, 3, or 4.

Designs aiming for maximum flexibility will probably use the internal 32 kHz oscillator, or an external 32 kHz crystal for projects with time-keeping requirements.

One interesting thing about memory layout with this part is that registers are logically organized in both 32-bit and 8-bit sets. For example, all the timers are configured through 32-bit-wide registers, but the communication peripherals are all 8-bit-wide.

## TIMERS

The KE04 has a six-channel FlexTimer and a two-channel FlexTimer. There’s also a two-channel periodic interrupt timer, a pulse-width timer, and a separate RTC timer.

Each FlexTimer works as a 16-bit auto-reload (period) timer and can perform PWM and capture/compare with any of its channels.

Periodic Interrupt timers function as interrupt-only auto-reload 16-bit timers.

The pulse width timer module has a two-channel input and can perform pulse-width measurement and miscellaneous capture duties, as well.

The real-time clock can be driven from the internal 1 kHz, internal 32-kHz, or an external clock, too.

## COMMUNICATIONS

The KE04 has separate UART, SPI, and I2C modules for communication duties.

The UART uses a fixed 16x oversampling with a 13-bit modulo divider baud generator. It supports break detection and generation for LIN compatibility. There’s no support for hardware flow-control or RS-485 TXEN signals, unfortunately.

The SPI peripheral supports master or slave operation and can operate in a single-wire bidirectional mode in addition to the standard full-duplex SPI mode.

The I2C supports master, multi-master, and slave operation up to 2.4 MHz. In slave mode, there’s address matching, second-address matching, general call, and address-range matching interrupt support. The peripheral also supports managing timeouts internally without the use of external timers. There are separate interrupt bits that will have to be individually managed by the ISR to determine state; the documentation illustrates the flow chart required to implement a master or slave role.

## ANALOG

The KE04 has a 400 kHz 12-bit ADC with 12 external channels. The ADC supports single or continuous conversion, and can dump conversion results in an eight-deep FIFO buffer. The ADC also has a compare-to functionality (though no window comparator capabilities). In addition to the 12 external inputs, there’s a bandgap reference, a temperature sensor, and external VREF pins as well. The ADC can operate from an asynchronous clock source, and supports asynchronous conversion hardware triggering, too.

There’s also an analog comparator that can select from external pins, internal bandgap reference, or a 6-bit DAC (basically, a 64-tap resistor divider).

# DEVELOPMENT ECOSYSTEM

The acquisition of Freescale by NXP comes with some growing pains.

Newer Kinetis parts, such as the [Kinetis KL03](https://jaycarlson.net/pf/freescale-nxp-kinetis-kl03/), are supported by NXP’s MCUXpresso Eclipse-based IDE. Older Kinetis parts, like the KE04 reviewed here, are supported by Freescale’s Kinetis Design Studio — or even versions of CodeWarrior. All three of these environments are Eclipse-based, but Kinetis Design Studio is the most “stock” — Freescale essentially packaged up Eclipse with all the standard [GNU MCU Eclipse](https://gnu-mcu-eclipse.github.io/) plugins in a one-click installer. I’m not complaining, because this is the route everyone else seems to be going, and it makes it easier to move between different vendor toolchains.

As for SDKs, things get even sillier.

There’s essentially three entirely different sets of SDKs floating around for different generations of Kinetis parts — even parts within the same family. The oldest-generation parts were supported mainly by Processor Expert — a code generator that was originally part of Code Warrior and then integrated into Kinetis Design Studio. Processor Expert uses a collection of high-level and low-level components that it auto-generates based on configuration panes. It’s a big, complicated system that isn’t really designed for user modification.

Slightly newer parts had support from Kinetis SDK 1.0 — a radical departure from Processor Expert that (for better or worse) brought the Kinetis software ecosystem in line with other manufacturers who only provide run-time peripheral libraries. While the SDK was designed to be used entirely from C, this SDK had optional Processor Expert support for generating the config structures passed to these peripheral libraries, too.

However, the *newest* parts are supported by Kinetis SDK 2.0, which has dropped Processor Expert support altogether.

One of the main reasons I’m reviewing two different Kinetis Arm chips is because of these radical SDK changes. For more information on the new Kinetis SDK, check out the [KL03 review](http://jaycarlson.net/microcontrollers/freescale-nxp-kl03/).

One important thing to note is that these NXP has not back-ported these new SDKs to older parts; so the part you use will generally support one (and only one) of these development routes.

The KE04, for example, only supports the “legacy” Processor Expert system — there’s no support for either the 1.x or 2.x Kinetis SDK. If you don’t like Processor Expert, you’ll be writing all your code from scratch using nothing more than the header files for the chip (which contain zero documentation in them).

However, just to add some confusion to the whole ordeal, there are ad-hoc driver library packages associated with many processors that have nothing to do with Kinetis SDK or Processor Expert. The KE04 has such a package, called the [FRDM-KEXX Driver Library Package](https://www.nxp.com/webapp/sps/download/license.jsp?colCode=KEXX_DRIVERS_V1.2.1_DEVD&Parent_nodeId=1393270890704711176673&Parent_pageType=product&Parent_nodeId=1393270890704711176673&Parent_pageType=product&Parent_nodeId=1393270890704711176673&Parent_pageType=product&Parent_nodeId=1393270890704711176673&Parent_pageType=product&Parent_nodeId=1393270890704711176673&Parent_pageType=product) (plus, because it’s Freescale, a [random update floating around on their “community” site](https://community.nxp.com/docs/DOC-329476)). I’ve never used this package, but if you’re someone who wants a simple peripheral library without any code-generation stuff to worry about, this is for you — every peripheral has a single .h file and .c file, and the functions are fairly well self-documented.

![img](https://jaycarlson.net/wp-content/uploads/2017/09/kinetis-design-studio_2017-08-04_16-10-20.png?cbc196&cbc196)Processor Expert provides a dizzying amount of configuration options; exposing essentially every peripheral function through some sort of configurable property-based panel. Yet, it always starts with sensible defaults (and even has a “Basic” and “Advanced” mode), and the GUI always uses MCU-agnostic terms for describing functionality, which keeps you out of the datasheet.

## PROCESSOR EXPERT

The big advantage with the Processor Expert code generator is that as soon as you learn the basics, you can essentially get any peripheral up and running without ever reading an MCU datasheet or really understanding the specifics of the MCU you’re targeting. I know that sounds insane — but it’s not far from the truth.

Processor Expert uses a component-oriented model with dependency resolution; for example, two high-level “PWM” component instances will share a common “Timer Unit” low-level component (as they will end up on different output-compare channels of that timer unit).

High-level components implement conceptual functionality, not peripheral functions. For example, if you wanted a function to execute every 25 ms, you would add a “TimerInt” component, and set the interval to 25 ms. Processor Expert will figure out which timer to use (FlexTimer, LPTimer, PIT, etc), route the clock appropriately, and calculate the necessary period register values, enable interrupts, generate an interrupt handler that takes care of any bits you need to set or clear. Of course, everything is extremely configurable — you can intervene and instruct it on these specifics.

Processor Expert generates linker files, initialization/interrupt callbacks as well as runtime libraries. Unlike code-gen tools like [Infineon DAVE CE](https://jaycarlson.net/pf/infineon-xmc1100/), which generates code that calls into standard runtime peripheral libraries, Processor Expert generates specific API calls on request, with any initialization values pre-calculated as constants. As an example, while some code-gen tools will generate a UART module that calls a standard runtime initialization routine with a human-readable baud rate, Processor Expert will generate code to directly write the correct baud rate generator values to the appropriate registers.

A lot of code-gen tools suffer from a loss of generality that makes them work well in toy example cases, but become useless when deployed in real application scenarios — especially for projects that have low-power requirements, or have dynamic pin-muxing in use. However, Processor Expert supports essentially anything imaginable — multiple system configurations allow you to shift between different clock and run modes, and components can be explicitly configured to share pins with other components.

While this all makes Processor Expert sound extremely fast and optimized, performance is actually somewhat mixed. As [Erich Styger wrote on his MCU on Eclipse blog](https://mcuoneclipse.com/2015/10/18/overview-processor-expert/), the introduction of Logical Device Drivers (LDDs) for RTOS-oriented applications has added measurable performance penalties and code-size increases. On entry-level devices like the KE04, Processor Expert feels a bit bloated, but on large processors with DMA, auto-scanning ADCs, FIFOs, and other advanced features, Processor Expert will take advantage of these, which can provide a huge performance boost over runtime peripheral libraries that don’t always expose complex functionality like this to the end user.

The 500-lb gorilla in the room, however, is this: if you’re new to Processor Expert, I think the first thing you’ll notice is how incredibly slow it is to use. I don’t mean “complicated” or “intricate” — I mean that even on my 4.5 GHz 12-thread desktop, creating instances of components, switching views, changing values, generating code, and building projects *takes forever*. The entire system is single-threaded, and every time a property is changed, everything has to be re-evaluated. I’m not sure it could be made faster — because of how flexible Processor Expert is, almost everything has a huge dependency graph; and because almost everything is automated, the whole system has to solve for the proper register values from a near-infinite possible selection.

Having said all that, in my testing, it was “fast enough” to not be completely frustrating to use, and it’s one of the only development environments I tested where I was able to literally complete an entire project without even glancing at a datasheet for the microcontroller. That would be impressive enough on an 8-bitter, but on a modern Cortex-M0+ Arm microcontroller, with complex (“flexible”) peripherals, it’s downright incredible. It is, by far, the most complete, flexible code-generator tool I’ve ever used.

 

![img](https://jaycarlson.net/wp-content/uploads/2017/09/kinetis-design-studio_2017-10-01_21-20-08.png?cbc196&cbc196)Kinetis Design Studio features a nearly-stock Eclipse debug environment with EmbSys Registers view for manipulating peripherals.

## DEBUGGING

Kinetis Design Studio is a nearly-stock Eclipse environment, so debugging is what you would expect: a clear, side-by-side disassembly and source view, with memory inspection, variables, and core registers are built-in. KDS uses the [EmbSysRegView Eclipse plug-in](http://embsysregview.sourceforge.net/), which is a powerful tool for inspecting and modifying register vales.

## DEVELOPMENT TOOLS

The official dev kit for the KE04 is the [FRDM-KE04Z](https://www.nxp.com/support/developer-resources/hardware-development-tools/freedom-development-boards/nxp-freedom-development-platform-for-kinetis-ke04-mcus:FRDM-KE04Z), a $15 dev board that integrates the KE04 along with some crap you don’t want (tri-color LEDs with low-impedance resistors tying up your GPIO and a random accelerometer you won’t use) — but it also comes with an on-board debugger that — with a bit of PCB hacking — can be used to debug external targets.

I have to applaud NXP for continuing to sell a massive number of dev boards covering the Kinetis line. [DigiKey lists 30 boards starting at $14](https://www.digikey.com/products/en/development-boards-kits-programmers/evaluation-boards-embedded-mcu-dsp/786?k=FRDM) that provide really good coverage of the entire Kinetis ecosystem.

The FRDM boards are workable, but there’s Freescale-specific peculiarities you’ll have to get used to with them. Schematics aren’t considered “documentation” but rather “downloads” — and you’ll be using the schematics to figure out why the hell your MCU is using way more current than it’s supposed to, and which parts you need to rip off to reduce power consumption.

I didn’t use a FRDM board for the KE04 tested here, but I did for my KL03, [and it gave me a ton of problems](https://jaycarlson.net/pf/freescale-nxp-kinetis-kl03/#development-tools).

I wish these boards were simpler — a debugger and a target MCU is all I need, and the Arduino form-factor does nothing for anyone (has any Freescale user, in the history of FRDM boards, actually plugged an Arduino shield into one of these boards?).

Speaking of debuggers, these dev boards come with OpenSDA-compatible debuggers on-board, which has some interesting properties not found in most debug adapters.

I talk a lot about the spectrum connecting “simple” and “flexible” together — OpenSDA plods toward the “flexible” side of the spectrum. The idea is that you can flash different “apps” to the debugger to do different things. There are OpenSDA apps that turn the debugger into a USB Mass Storage Device for drag-and-drop programming (I guess that’s cool?), and you can install different debuggers, too, including the P&E Micro one, the CMSIS-DAP (I guess now called [DAPLink](https://github.com/ARMmbed/DAPLink)), and [SEGGER J-Link](https://www.segger.com/downloads/jlink#JLinkOpenSDABoardSpecificFirmwares).

I suppose this is kind of neat, but quite frankly, whenever I buy a FRDM board, I immediately install the J-Link firmware on it, and never look back. I’m not sure if this is still the case, but early FRDM boards shipped with the MSD “drag and drop” downloader instead of even having a proper debugger installed. Silly.

# PERFORMANCE

## BIQUAD FILTERING

The KE04 pulled a 1717.28 ksps biquad filtering rate at 14.31 mA. This processing speed is in line with other 48 MHz Cortex-M0+ parts. Since all these parts have the same core, this really is a test of the flash accelerator (since none of these parts have zero-wait-state flash memory at these speeds). The efficiency — 27.53 nJ/sample — is much worse than most other 48 MHz Arm parts — only the [Nuvoton M0](https://jaycarlson.net/pf/nuvoton-m0/) was worse.

## PIN TOGGLING

The KE04 has single-cycle GPIO access via its Fast GPIO peripheral. Add in two cycles for a jump, and the KE04 should sit at a 3-cycle pin toggling speed, which is as good as it gets in any microcontroller on the market.

Unfortunately, some of the Processor Expert generated code simply makes you shake your head: it takes 40 cycles to toggle a GPIO pin, since the high-level “Bit” component calls into a low-level GPIO component, passing it a configuration structure and other unused parameters.

The LDD call isn’t inlined, and even compiling with -O3 won’t eliminate these function calls (I’m not sure why GCC refuses to inline these calls — but I suspect is has something to do with the pointer use inside the function body).

 

![Logic_2017-08-04_13-51-19.png](https://jaycarlson.net/wp-content/uploads/2017/08/logic_2017-08-04_13-51-19.png?cbc196&cbc196)Even with an 8 MHz clock supplying the core and the bus, the MCU was missing more than every other byte in the DMX-512 receiver demo. This illustrates the severe performance problems with using Processor Expert for projects like this. However, with higher-end MCUs that have DMA capabilities, and more RAM, Processor Expert could get

 

## DMX-512 RECEIVER

Of course, for performance-critical GPIO calls, it’s easy enough to bypass Processor Expert and use direct register calls, but for most use-cases, it’s hard to replace PE code with optimized application-specific routines — especially when interrupts are involved.

That’s the case with the DMX-512 receiver. The 16x oversampling UART means we need a 4 MHz-minimum bus clock for the UART. That should be plenty fast for the processor’s ISR, but when we interject Processor Expert into the mix, its runtime inefficiencies really start to show. I had to run the processor at 12 MHz just to give the core enough time to fetch the byte before the UART FIFO overflowed.

That’s because it took 218 clock cycles to complete a single byte-save-to-buffer routine in Processor Expert. Ouch.

Using a 16 MHz core clock / 8 MHz bus clock, the power consumption was 4.76 mA. A 12/12 clocking scheme yielded the lowest power at 3.77 mA (a major contributor is that the clock could be divided with BDIV instead of the Clock Gen’s integer divider).

Interestingly, the simplistic clocking scheme hurt us a bit: by looking at a logic trace, there’s room to further reduce the bus clocks, unfortunately, the UART (which needs a 4 MHz input — or a multiple of 4 MHz) is chained to the flash clock, and there’s no way to get an 8 or 4 MHz flash clock with a 12 MHz core clock.

Despite the poor power consumption, there’s an important victory for the KE04: Even though I’ve never used this part before, I wrote the entire DMX-512 Receiver project in 15 minutes, without cracking open the datasheet (let alone the reference manual) for the part. I had to type precisely 7 lines of code to hook all the Processor Expert modules together, and it worked immediately the first time I compiled the code. That — itself — is an amazing result from this experiment.

# BOTTOM LINE

The KE series has a decent selection of peripherals, but testing shows this part clearly isn’t meant for low-power applications. The sizable collection of timers and 12-channel 12-bit 400 ksps ADC are probably the stand-out peripherals on paper, but what you’re really paying for is a 5V-capable Arm part that has rugged EMI resistance — and, maybe more importantly, the Processor Expert workflow.

The two Arm parts that seem more capable than this part — the [SAM D10](https://jaycarlson.net/pf/atmel-microchip-sam-d10/), and the [Nuvoton M0](https://jaycarlson.net/pf/nuvoton-m0/) — either have clunkier code-gen tools (in the case of the SAM part), or no code-gen tools at all (in the case of the Nuvoton part). Neither of them has vendor-provided cross-platform tools, either — both are stuck in Windows, and neither has a true Eclipse-based IDE.

I think the KE04 — and all the “legacy” Kinetis parts that target a Processor Expert workflow — work well for consultant/contractors and engineers working in garage-based start-ups: with just a few lines of code, you can rapidly prototype your ideas, and worry about optimization/code efficiency later.

I also see the KE series (and other Kinetis MCUs) as useful parts for the hobbyist market, too: here’s an ecosystem that’s reasonably priced, extends past 150 MHz, and provides really high-level, easy-to-use configuration of peripherals that cover most functions out-of-the-box. Plus, in Kinetis Design Studio, you get a nearly-stock Eclipse environment that’s completely cross-platform — and one of the only manufacturer-supplied IDEs that supports OpenOCD debuggers out-of-the-box. These low-end KE parts come in easy-to-solder SOIC and TSSOP packages, but even the larger chips come in 0.8mm QFPs, which are still well within the abilities of most hobbyists to solder.

As for educational use, the KE04 is not really appropriate for use in an embedded systems classes, where students really should use a platform with better header files (and, in my opinion, should probably stick to 8-bit parts). But for people studying the *use of* embedded systems (say, in a mechanical engineering mechatronics class), Processor Expert would let non-EE/CE students learn concepts like UART communication, PWM, interrupts, and timer capture — all without having to get stuck at the peripheral register level.