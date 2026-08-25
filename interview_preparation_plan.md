# Interview Preparation Plan — Mid-Level Embedded Systems Engineer

## Role scope (drives the ordering below)

This plan targets a mid-level **Embedded System Design Engineer** role of the hardware-plus-firmware kind, where the scope typically reads:

- Design and implement both the hardware and the firmware of embedded devices, from requirements through manufacturing completion
- Hardware design and development including PCB design, fabrication, bring-up and release to manufacturing
- PCB and circuit simulation tools: **Allegro/OrCAD, Altium, PSpice**
- **DC-DC, DC-AC, bidirectional converters, inverters, DC energy metering**
- **8-bit and 32-bit microcontrollers**, sensors, Bluetooth and Wi-Fi modules
- Protocols: **I2C, SPI, UART, CAN, I2S**
- Hardware design and code reviews; hardware/software QA; pre- and post-production support
- 3–4 years experience, BSc in CS/CE/EE

Common product domains for this profile: healthcare devices (ECG, smart scales, wellness monitoring), agriculture and livestock monitoring, security and detection, POS, supply chain.

### Study priority tiers

- **Tier 1 (must clear, highest risk of failure):** Power electronics & DC energy metering · Board bring-up & hardware debug · PCB/EDA tooling (Altium, OrCAD/Allegro, PSpice)
- **Tier 2 (must be fluent):** I2C/SPI/UART/CAN/I2S · Embedded C · STM32/ESP-IDF design patterns · 8-bit vs 32-bit MCU selection & Cortex-M · Sensors and analog front end
- **Tier 3 (expected, lower depth):** RTOS/bootloader/OTA · Low power · System architecture & lifecycle · BLE/Wi-Fi · Security
- **Tier 4 (your existing strength — revise, don't study):** Embedded Linux, Yocto, device drivers, edge/AI pipelines

---

## 1. Board Bring-Up & Hardware Debug

Near-universal opening question for this role, because bring-up is squarely your responsibility when you own both the board and the firmware.

**Bring-up sequence**
- Walk me through your exact process the first time you power up a new PCB revision, before firmware runs.
- What do you check with power off, before applying any voltage? (visual inspection, rail-to-ground continuity/short check with a DMM, assembly vs BOM/silkscreen)
- How do you bring up power rails on a board with multiple supplies — all at once or one at a time?
- How do you verify power sequencing requirements are met, and what happens if you violate them?
- The board powers on and draws far more current than expected — what is your triage process?
- How do you check the crystal/oscillator is actually running before trusting anything downstream?
- What does a good vs bad reset waveform look like?
- What are boot straps / boot mode pins, and how do you confirm the MCU samples them correctly at reset?
- You cannot get JTAG/SWD to connect on a brand-new board — what do you check, in order?
- The board is completely dead, no current draw — what is your debug sequence?
- The board draws current but nothing happens — how do you isolate hardware vs firmware?

**Instruments and probing**
- When do you reach for a scope vs a logic analyzer vs a protocol analyzer vs a DMM?
- How do you choose oscilloscope bandwidth and sample rate for a given signal? (BW ≥ 3–5× highest frequency content, driven by rise time)
- Why does a long ground lead on a scope probe corrupt the signal, and what is the fix? (ground spring; minimise probe loop inductance)
- When do you need a differential probe rather than single-ended?
- How would you measure the actual current an IC draws? (current probe, shunt + scope, precision current sense)
- What are SWO/ITM and how have you used them to debug without halting the CPU?
- You see intermittent glitches — how do you configure the scope to catch them? (trigger modes, infinite persistence, single-shot)
- How does probe loading affect a high-impedance or high-speed node?

---

## 2. Communication Protocols — I2C, SPI, UART, CAN, I2S

Named in essentially every job description at this level. Expect depth, not definitions.

**I2C**
- Why open-drain rather than push-pull on SDA and SCL? What breaks with push-pull and multiple masters?
- Given 200 pF bus capacitance and a 300 ns rise-time target, size the pull-up (R_max = t_rise / (0.8473 x C_bus)). What does that imply for Fast Mode on a long bus?
- Why does a smaller pull-up improve rise time but hurt power and V_OL?
- What does the 400 pF bus capacitance limit actually constrain, and is exceeding it a hard or soft failure?
- Explain 7-bit vs 10-bit addressing on the wire. Two devices share a fixed address with no ADDR pin — what are your real options?
- Describe a full write transaction bit by bit: START, address+R/W, ACK, data, ACK, STOP.
- What is a repeated START and why use it instead of STOP+START for a register read?
- Who drives ACK, and what does a NACK on the address byte mean vs on a data byte?
- What is clock stretching, which side does it, and how must the master's driver handle it correctly?
- How does non-destructive arbitration work between two simultaneous masters, and why does open-drain make it possible?
- A slave is stuck holding SDA low. What does the bus look like on a scope, and what is the recovery procedure? Why 9 clocks specifically? What do you send afterwards to guarantee idle?
- How would you implement lockup detection and auto-recovery in firmware?
- List the speed modes (100 k / 400 k / 1 M / 3.4 M) and what changes electrically between them.
- On a scope, how do you distinguish "no pull-up" from "slave never answers" from "wrong address"?

**SPI**
- Define CPOL and CPHA and derive Modes 0-3. In Mode 0, which edge samples and which shifts?
- Every byte is shifted by one bit — which mode mismatch produces exactly that?
- Why does SPI have no addressing, and how does chip select substitute?
- Why one CS per slave? Explain daisy-chain vs parallel CS and the constraint daisy-chaining imposes.
- How would you drive 8 SPI slaves with only 4 GPIOs?
- Why is SPI full duplex even when only one side has data? What do you clock out on MOSI during a read?
- Why is there no ACK at the protocol level, and how do higher layers detect failure?
- What limits maximum SPI clock in practice? At tens of MHz over a ribbon, what SI problems appear and how do you fix them?
- Garbage data but clean clock and CS — first suspect?

**UART**
- Draw a frame: idle level, start bit, data, parity, stop. What is it asynchronous *with respect to*?
- Two devices are each 1% off in opposite directions — does the link work? Where does the ~2-3% budget come from, and why does tolerance tighten as frame length grows?
- Explain 16x oversampling and why majority voting on the middle three samples rather than one.
- Works at 9600 but drops characters at 921600 with the same crystal — mechanism?
- Framing vs parity vs overrun error — what triggers each? RX FIFO overflowing: which is it, and what is the fix?
- TTL vs RS-232 voltage levels, and why RS-232 logic is inverted.
- Why does RS-485 use differential signalling? How do DE/RE control direction, and what happens if two nodes drive at once?
- Why terminate an RS-485 bus at both ends, at what value, and what is fail-safe biasing for?
- Hardware RTS/CTS vs software XON/XOFF — why does XON/XOFF fail for binary data?
- You have a UART line and no idea of the baud rate. How do you find it on a scope?

**CAN**
- What do CAN_H and CAN_L do during a dominant vs recessive bit? Why is 0 dominant rather than 1?
- Why exactly two 120 ohm terminators at the physical ends? What happens with one, none, or one at every node?
- Describe the controller/transceiver split — why can the MCU not drive the bus directly? What is listen-only mode for?
- Walk through bitwise arbitration with two nodes transmitting different IDs. Why does the lower ID win, and what does that mean for priority design?
- Standard 11-bit vs extended 29-bit IDs — how does the IDE bit let a receiver distinguish them mid-arbitration?
- List the frame types (data, remote, error, overload). Why has the remote frame fallen out of favour?
- Explain bit stuffing: why insert after 5 identical bits, and what does the receiver do on seeing 6?
- What are TEC and REC? Explain error-active, error-passive and bus-off, with rough thresholds. What causes bus-off and what is the recovery behaviour?
- Name the four bit-timing segments and what each compensates for. Where is the sample point, and what is the tradeoff in moving it later?
- Why does propagation delay trade off against bus length for a given bit rate? Why does 1 Mbit/s top out near 40 m?
- What does CAN FD change, and why must arbitration still run at the classic rate?
- On a scope, what differential swing do you expect dominant vs recessive, and what does a reduced swing indicate?
- One node keeps going bus-off — what do you check first? (termination, ground offset, bit-timing mismatch, stuck-dominant transceiver)

**I2S**
- Name the signals (BCLK, LRCLK/WS, SDATA, optional MCLK) and each role.
- Derive BCLK = sample_rate x bits_per_channel x channels. For 48 kHz, 24-bit stereo, what BCLK?
- Why does LRCLK toggle at exactly the sample rate?
- Why do many codecs need MCLK in addition to BCLK, and what consumes it internally? Why 256xFs or 512xFs specifically?
- Explain the one-BCLK delay in standard Philips I2S. Contrast standard vs left-justified vs right-justified — where does the MSB land in each?
- What is TDM/PCM mode and when would you use it over stereo I2S?
- Difference between word length and slot length, and why pad a 16-bit sample into a 32-bit slot?
- Which device should be I2S master in a typical MCU+codec design, and what do you see if both think they are master?
- Left and right channels are swapped — most likely misconfiguration?
- Audio is grainy but the waveform looks right — what do you suspect, and why does clock jitter corrupt audio quality rather than causing dropouts?

**USB (brief)**
- Walk through enumeration: reset, address assignment, descriptor requests.
- Name the four endpoint types and a use case for each. Why does isochronous guarantee bandwidth but not delivery?
- What are CDC, HID and MSC, and why does using a standard class avoid a custom host driver?
- Why is D+/D- a differential pair, and at what impedance? How does the host detect device speed from a pull-up?

**Cross-cutting**
- One sensor 5 cm away, read-only vs 12 sensors around a 3 m chassis needing low latency and error detection — which protocol for each, and why?
- Rank I2C, SPI, TTL UART, RS-485 and CAN by realistic maximum cable length, and name the electrical property that limits each.
- Why does differential signalling reject common-mode noise that single-ended cannot?
- What is galvanic isolation, when do you need it on a bus, and what implements it? What breaks if you isolate data lines but not the return path?
- 3.3 V MCU to a 5 V I2C peripheral — why is a resistive divider wrong, and what is the standard bidirectional approach?
- A bus "isn't working" and you have a scope and a logic analyzer. What do you probe first, and why? How does triage order differ for I2C vs SPI vs CAN?
- Waveforms look correct but the device still does not respond — what non-electrical causes do you check? (wrong address/mode, missing clock enable, power sequencing, wrong rail)

---

## 3. Power Electronics & DC Energy Metering

**This is the biggest gap between a firmware-leaning background and this role's hardware scope. Highest study leverage.**

**Firmware for power control (most likely angle for a firmware-leaning candidate)**
- How would you generate complementary PWM with dead-time insertion using a timer peripheral (e.g. STM32 advanced timer TIM1/TIM8)?
- How do you synchronise ADC sampling to the PWM cycle so you sample current/voltage at the right instant?
- Walk me through implementing a PI/PID control loop in firmware for output voltage/current regulation.
- Why fixed-point rather than floating point, and how do you implement it (Q-format)?
- How do you size control-loop timing — what sets your sample/update rate relative to switching frequency?
- How do you detect and respond to overcurrent — hardware comparator into a timer break input vs software polling? Why does the difference matter?
- How do you implement overvoltage, undervoltage and thermal shutdown in firmware?
- Describe a state machine for a charger/converter: startup, soft-start, regulation, fault, shutdown.
- How do you prevent shoot-through purely from the firmware side (dead-time register configuration)?

**DC-DC converters**
- Explain how a buck converter works — topology and the two switching states.
- Explain boost and buck-boost — when would you choose each?
- What is duty cycle and how does it relate to output voltage in buck vs boost?
- What is the difference between CCM and DCM, and why does it matter for control design?
- How do you select inductor value for a target ripple current? Output capacitance for a target voltage ripple?
- What determines switching frequency choice (efficiency vs size)?
- Synchronous vs asynchronous rectification — why a sync FET instead of a diode, and what is the risk?
- Main loss mechanisms — conduction vs switching loss — and how each scales with current and frequency.
- When would you pick an isolated topology (flyback, forward) over non-isolated?
- Explain flyback operation and why it dominates low-power isolated supplies. How does it differ from a forward converter?
- When would you use SEPIC instead of buck-boost?
- LDO vs switching regulator — when do you accept the efficiency loss? (noise, PSRR, cost, dropout, EMI)

**Control loop theory**
- Voltage-mode vs current-mode control — practical difference, and why current-mode is often preferred?
- What is a compensation network doing, conceptually (poles/zeros, stability)?
- What is phase margin and why does it matter?
- How does loop bandwidth relate to load transient response?
- How do you handle integrator windup?

**DC-AC inverters**
- Explain how an H-bridge produces AC from a DC source.
- What is SPWM and how do you generate it with a timer (sine reference vs triangle carrier)?
- Why is dead time required, and how do you choose its duration?
- What is a bootstrap circuit and why is it needed to drive a high-side N-channel MOSFET/IGBT?
- Single-phase vs three-phase inverter — what changes in control (120° phase-shifted PWM, space vector modulation conceptually)?
- What is THD and why does it matter in inverter output?
- What is anti-islanding and why is it required in grid-tie?
- Grid-tie vs off-grid — what differs in synchronisation and control?
- What is a bidirectional converter, and how does the same hardware both charge and discharge?

**DC energy metering**
- How does a DC energy meter measure V and I, compute instantaneous power, and accumulate energy in Wh?
- What current-sensing methods do you know — shunt, Hall effect, current transformer, Rogowski — and which are valid for **DC**? (CT and Rogowski are AC-only; DC needs shunt or Hall/fluxgate)
- Why is a shunt common for DC, and what precision/thermal issues come with it (Kelvin sensing, TCR, self-heating)?
- How do you amplify a shunt's millivolt signal before the ADC (instrumentation amp / current-sense amp)?
- What does accuracy class mean, and what design choices drive meter accuracy (ADC resolution, reference stability, calibration)?
- How would you calibrate a DC energy meter in firmware (gain/offset against a known reference)?
- What is your ADC sampling strategy — rate, resolution, anti-aliasing on V and I channels?
- Would you use a dedicated metering IC (ADE, STPM3x) or raw ADC + MCU computation? Why?
- What is coulomb counting, and how does it serve both energy accumulation and battery SoC?
- Do you need isolation for measurement, and how would you achieve it?
- How do you persist accumulated energy across power loss (NVM, wear levelling)?

**Battery management (frequently paired)**
- How is State of Charge estimated — coulomb counting vs OCV lookup vs Kalman — and the tradeoffs?
- What is cell balancing, passive vs active, and why is it needed in a series stack?
- Explain a CC/CV charging profile — why constant current then constant voltage?
- How do you protect against overcharge, over-discharge, overcurrent, overtemperature?

---

## 4. Embedded C & Firmware Fundamentals

**volatile / const / static / linkage**
- Why does `volatile` exist, and what breaks if you omit it on a memory-mapped register or an ISR-shared flag?
- Does `volatile` guarantee atomicity or ordering?
- Difference between `const int *p`, `int * const p`, and `const int * const p`?
- Can a variable be both `const` and `volatile`? Give a real example.
- What does `static` mean for a local variable, a global, and a function?
- Explain internal vs external linkage. What does `extern` do?

**Bit manipulation and register access**
- Set, clear, toggle and test a specific bit in a register using macros.
- Write a read-modify-write sequence changing 3 bits without disturbing the rest.
- Why is `reg |= mask` sometimes unsafe on a hardware register? (write-1-to-clear, side effects)
- Portability pitfalls of bitfields — bit order, padding, compiler-dependent layout.
- When would you use bitfields vs explicit masks/shifts for a register map?

**Pointers**
- Declare a pointer to a function taking `int` returning `void`; declare an array of function pointers (jump table / vector table).
- What is a pointer-to-pointer used for in embedded code?
- Why does `ptr+1` move by `sizeof(struct)` rather than one byte?
- What happens when you cast an integer address to a pointer and dereference it (memory-mapped I/O)?

**ISR safety, reentrancy, atomicity**
- What is unsafe inside an ISR (blocking calls, `malloc`, `printf`, long loops)?
- Why is `volatile` alone not enough for a variable shared between ISR and main loop?
- How do you safely share a multi-byte variable between an 8-bit MCU's ISR and main code?
- What is a critical section, and how do you implement one on Cortex-M (`__disable_irq`, `BASEPRI`) vs AVR (`cli`/`sei`)?
- Give a race-condition example between main loop and ISR, and three ways to fix it.
- What makes a function reentrant, and why must ISRs be?
- How do nested interrupts and priorities affect data consistency?

**Memory, alignment, endianness**
- Why is dynamic allocation avoided or banned in embedded and safety-critical firmware?
- What is heap fragmentation and why is it especially dangerous in a long-running device?
- Alternatives to dynamic allocation — static allocation, memory pools.
- What determines stack size, and what happens on overflow?
- Explain struct padding with an example (`char, int, char`). How do you pack a struct, and what is the cost?
- What is data alignment and why do some architectures fault on unaligned access?
- What is endianness, and how do you write endian-safe pack/unpack for a byte stream?

**Linker, startup, memory map**
- Walk through what happens between reset and `main()`.
- What are `.text`, `.data`, `.bss`, `.rodata`, and where does each live in Flash vs RAM?
- Why is `.data` copied from Flash to RAM at startup while `.bss` is zeroed?
- What does a linker script define?
- What is in the vector table, and what is the role of `Reset_Handler`?
- How do you read a `.map` file to find where a variable landed or diagnose a region overflow?

**Traps and optimisation**
- Explain integer promotion — what happens with arithmetic on two `uint8_t`?
- Give a surprising signed/unsigned comparison result and explain it.
- Why is signed overflow undefined while unsigned wraps?
- Why can `-O2` break a polling loop that works at `-O0`, and what does that tell you about the root cause?
- When do you need a compiler memory barrier in addition to `volatile`?

**Fixed-point and MISRA**
- Why avoid floating point on a Cortex-M0+ or 8-bit AVR?
- Explain Q15 and how to multiply two Q15 numbers correctly.
- What is MISRA C and why does safety-critical industry require it? Name rules you have followed.

**Debugging**
- Walk through diagnosing a HardFault on Cortex-M — which registers first? (CFSR, HFSR, BFAR/MMFAR, stacked PC/LR)
- Difference between bus fault, usage fault, memory management fault, and how each escalates.
- How do you detect or prevent stack overflow (canaries, watermarking, MPU guard region)?
- What causes a watchdog reset, and how do you root-cause it after the fact?
- Hardware debugger vs `printf` — tradeoffs on a timing-sensitive system.

---

## 5. Design Patterns in Production Embedded C/C++ (STM32 & ESP-IDF)

How real vendor code is structured. Expect these as "how would you architect this driver" questions.

**Driver and structural patterns**
- Why put a HAL between application code and registers instead of writing `GPIOA->ODR` directly? What does it cost?
- When would you choose STM32 LL drivers over HAL, and what do you give up? Where does HAL actually spend the extra cycles? (status returns, handle State/Lock updates, `HAL_GetTick()` timeout polling)
- Why does `HAL_UART_Transmit(&huart2, ...)` take a handle instead of the driver keeping global state? What does the handle buy you for multi-instance peripherals?
- What does an opaque handle (`spi_device_handle_t`, `i2c_master_bus_handle_t`, `TaskHandle_t`) give you that a raw struct pointer does not? (definition lives in the .c, so layout changes cannot break callers)
- Why does ESP-IDF favour `foo_config_t cfg = {...}; foo_init(&cfg, &handle);` over a long argument list? (designated initializers, zero-friendly defaults, adding a field does not break call sites)
- How does STM32's `HAL_XXX_Init(&handle)` differ structurally from ESP-IDF's config-then-init? (STM32 folds config into the handle; ESP-IDF separates request from result)
- How do you get polymorphism in plain C — e.g. one sensor API over BMP280 and BME680? (struct of function pointers + `void *ctx`; `const` ops table lives in flash) Where does ESP-IDF use this? (VFS layer)
- What is the init/deinit contract, and what must deinit be safe against? (partially-initialised handle after a failed init)
- What happens if you `i2c_master_bus_add_device()` and never remove it before deleting the bus?
- What is in an ESP-IDF component, and why Kconfig rather than a `config.h` full of `#define`s?
- What are CubeMX `USER CODE BEGIN/END` blocks, and why do experienced STM32 engineers never write outside them? Where should application logic actually live?

**Concurrency and event patterns**
- When is a bare superloop the right architecture rather than FreeRTOS, and when is it not?
- Why should an ISR almost never do real work? What is the deferral pattern?
- What is wrong with the classic `volatile uint8_t rx_flag` set in an ISR and polled in `main()`?
- Show the `xQueueSendFromISR` idiom and explain `xHigherPriorityTaskWoken` and `portYIELD_FROM_ISR`.
- Why prefer `xTaskNotifyFromISR` over a queue when exactly one task is woken?
- How do you implement a lock-free single-producer/single-consumer ring buffer for UART RX? Why does SPSC need no mutex while MPSC does?
- What problem does the `esp_event` loop solve that direct calls do not? When is a direct callback better?
- Why do interviewers prefer queues over "shared global + mutex" for inter-task data?
- Differentiate FreeRTOS queue vs stream buffer vs message buffer.
- Explain the STM32 `__weak` callback idiom end to end. What breaks when two modules need `HAL_UART_RxCpltCallback` for two different UARTs, and what is the fix?
- Why does nearly every ESP-IDF callback carry a `void *user_data`/`arg`? (the C answer to closures)
- Explain circular DMA with half-transfer and transfer-complete callbacks. Why is half-transfer what buys you gapless capture?
- What does zero-copy mean here, and where does it break down? (data outliving the next DMA half-cycle; cache maintenance on STM32H7 / ESP32-S3 PSRAM)

**State management**
- Switch-based vs table-driven vs function-pointer state vs hierarchical FSM — what is the ceiling of each, and how do you choose?
- Why is table-driven favoured for safety-relevant firmware? (transition data separate from behaviour, auditable, illegal transitions detectable)
- When is a flat FSM insufficient, and what does an HSM buy you? (shared entry/exit and any-state-to-FAULT handled once)

**Error handling**
- What is `esp_err_t`, and why return codes rather than exceptions?
- What does `ESP_ERROR_CHECK` actually do, and why is it fine in `app_main` init but wrong inside a reusable driver?
- How does `ESP_RETURN_ON_ERROR` differ, and when do you use `ESP_GOTO_ON_ERROR`?
- Name the four `HAL_StatusTypeDef` values. What does `HAL_BUSY` specifically tell you?
- Why is goto-to-cleanup idiomatic rather than a code smell in C?
- How do you choose between an assert, a recoverable error return, and a deliberate reboot?
- What happens on an ESP-IDF crash, and how do you decode a core dump? What is the STM32 equivalent?

**C++ on MCUs**
- How do you get RAII benefit when exceptions are disabled, and why does RAII still matter without them?
- Why do embedded C++ codebases build with `-fno-exceptions -fno-rtti`? What replaces exceptions?
- What does CRTP solve versus a virtual interface, and when is CRTP the wrong choice? (runtime-detected hardware needs real dispatch)
- What does a `constexpr` type-safe register map or a strong typedef for units prevent at compile time?
- Why is PIMPL usually the wrong tradeoff in firmware?
- How do you construct C++ objects without a heap? (placement new into `alignas` storage; explicit destructor call)
- What is ETL and why reach for `etl::vector` over `std::vector` on an MCU?

**Testability**
- How do you make a driver that calls `HAL_I2C_Master_Transmit` directly testable on a host?
- Explain the linker-seam trick for mocking HAL without refactoring every call site.
- What are Unity, Ceedling/CMock, CppUTest and GoogleTest each used for in embedded practice?
- Host-based tests vs hardware-in-the-loop — what should each catch, and what can neither catch?

---

## 6. PCB Design, EDA Tools, Signal Integrity, EMC, DFM

**EDA tooling (named explicitly in most such postings)**
- Walk me through your schematic-to-Gerber workflow in Altium — library part creation through DRC to fabrication output.
- How do you manage component libraries and keep symbols, footprints and 3D models in sync?
- What is your process for verifying a footprint against the datasheet land pattern before release?
- Which DRC rules do you always set before routing (clearance, min trace/space, annular ring, silkscreen-over-pad)?
- Describe generating a release package — Gerber/ODB++, drill, pick-and-place, BOM — for a contract manufacturer.
- Difference between OrCAD Capture and Allegro PCB Editor — where does each fit?
- How do you handle constraint management in Allegro for controlled-impedance or length-matched nets?
- What class of errors does ERC catch before layout that DRC will not?
- **PSpice/LTspice:** when do you simulate vs prototype on an eval board?
- Walk through a DC operating point, a transient analysis and an AC sweep on the same circuit — what does each tell you?
- How do you use Monte Carlo or worst-case analysis against component tolerance spread?
- Give an example where a SPICE simulation caught a problem before you cut a board.

**Power design**
- What is the purpose of a decoupling capacitor, and why more than one value per IC?
- How do you choose decoupling values and placement relative to the IC?
- Bulk vs local decoupling — where does each go?
- What is inrush current and how do you control it (soft-start, NTC, MOSFET)?
- Why do you need brown-out detection?
- How do you measure microamp sleep current accurately, and why is a standard DMM often wrong?

**Signal integrity**
- When must a trace be treated as a transmission line? (trace delay > ~1/6 to 1/4 of rise time)
- What causes ringing and reflections, and how do you fix them?
- Series vs parallel/Thevenin termination — when each?
- What is a stub and why is it a problem at high speed?
- How do you control trace impedance — which stackup parameters matter?
- What is crosstalk and how do you minimise it (3W rule, guard traces, reference plane continuity)?
- What is ground bounce / simultaneous switching noise?

**Layout and grounding**
- Why is splitting a ground plane usually wrong, and when is it justified?
- How does return current actually flow at high frequency? (under the trace — least impedance, not least resistance)
- How do you choose a stackup for a mixed digital/analog/RF board?
- What is via stitching for?
- How does current loop area relate to radiated emissions?

**EMI/EMC**
- Main sources of EMI on a digital board? (clock edges, switching regulators, high di/dt loops)
- Common-mode vs differential-mode noise, and why it matters for filter design.
- When and why add a ferrite bead, and how do you pick one (impedance vs frequency curve)?
- How do you protect I/O from ESD, and how do you select a TVS diode (clamp voltage, capacitance)?
- Have you done pre-compliance testing — what setup, and what are you looking for?
- What does FCC/CE radiated and conducted emissions testing involve, and a typical failure mode?

**Schematic and datasheet fluency**
- How do you calculate a pull-up for an I2C bus or a GPIO?
- How do you level-shift between 3.3 V and 1.8 V or 5 V?
- Open-drain vs push-pull — when do you need each?
- Difference between "recommended operating conditions" and "absolute maximum ratings", and why brief excursions matter.
- Walk through how you read a new part's datasheet before designing it in.

**DFM and production**
- What test points do you add, and how do you place them for ICT and bring-up?
- What is panelization and what constraints does it impose?
- How do you design a JTAG/boundary-scan chain for production test, and what does it catch that functional test does not?
- How do you handle production programming and provisioning (serial numbers, keys, calibration) at scale?
- Which layout choices commonly hurt yield at fab or assembly?
- How do you handle impedance tolerance in a stackup spec with a fab?

**Thermal**
- How do you estimate power dissipation for a regulator before layout?
- How do you use copper pour and thermal vias to manage heat?

---

## 7. MCU Selection & ARM Cortex-M

- How do you decide between an 8-bit MCU (PIC/AVR/8051) and a 32-bit Cortex-M — cost, peripherals, memory, power?
- Give an example where an 8-bit part was the better choice even though 32-bit was available.
- What toolchain differences have you dealt with across PIC (MPLAB X/XC8), AVR (avr-gcc), 8051 (Keil C51), ARM (Keil/IAR/GCC+CMSIS)?
- Explain the Cortex-M exception model — vector table contents, how the NVIC prioritises and preempts.
- What is SysTick used for, and when would you use a general-purpose timer instead?
- What is the MPU for on a Cortex-M, and when have you actually needed it?
- What is Thumb-2 and why does it matter for code density?
- What does CMSIS give you, and how does it keep peripheral code portable across vendors?
- How do you estimate flash/RAM budget early to decide if an 8-bit part has headroom?

---

## 8. Sensors & Analog Front End

**ADC and sampling**
- SAR vs sigma-delta — when would you pick each?
- Resolution vs accuracy vs ENOB — why can a 24-bit ADC deliver far less effective resolution?
- What does 1 LSB mean at a given resolution and reference, and how does that map to real-world precision?
- Explain Nyquist and what aliasing looks like in a sampled signal.
- How do you design an anti-alias filter for an ECG front end — cutoff and order, and why?
- Explain oversampling and decimation — how does oversampling improve effective resolution, and what is the tradeoff?
- How do you reduce ADC noise (averaging, filtering, layout, reference stability)?

**Instrumentation and biopotential (their healthcare vertical)**
- Why an instrumentation amplifier rather than a plain op-amp difference amp for a bioelectric signal?
- Explain CMRR — why is it critical for ECG, and what is a typical target?
- What causes offset and drift in an in-amp chain, and how do you compensate?
- Why does input impedance matter at the electrode interface?
- How do you deal with 50/60 Hz mains interference in ECG — hardware (shielding, notch, RLD) vs software?
- What is Right Leg Drive and why is it used?
- What patient-safety and isolation considerations apply to a skin-contact device — leakage current, galvanic isolation, IEC 60601 awareness?

**Load cells (their smart-scale vertical)**
- Explain a Wheatstone bridge — how does a strain gauge convert load into a measurable signal?
- Why is an HX711-class 24-bit integrated-PGA ADC well suited to load cells?
- How do you calibrate a load cell (zero/tare and span) and handle temperature drift?
- How do you implement tare in firmware so the scale zeroes correctly regardless of prior load?

**Digital sensors**
- How do you choose between I2C and SPI for a sensor (speed, pin count, multi-drop, noise immunity)?
- How do you handle IMU sensor fusion (complementary vs Kalman)?
- What is your debug process when an I2C sensor intermittently does not ACK or returns garbage?

---

## 9. RTOS, Bootloaders & OTA Updates

**Scheduling**
- Preemptive vs cooperative scheduling — what does each guarantee, and what fails under each?
- How does a fixed-priority preemptive scheduler pick the next task? What happens to equal-priority tasks (time slicing vs run-to-completion)?
- What is rate-monotonic scheduling, and what is the utilisation bound that makes a task set schedulable?
- What is the idle task for, and what runs in it? (idle hook, tickless idle, stack cleanup of deleted tasks)
- What is the tick interrupt, and what happens on every tick?
- Hard vs soft real-time — give an example of each. What is determinism, and why is average latency the wrong metric?
- What is jitter, and what causes it in an RTOS system?
- What is WCET, and why is it hard to establish on a modern MCU? (caches, branch prediction, DMA contention, interrupt nesting)

**Synchronisation**
- Mutex vs binary semaphore vs counting semaphore — what is each correct for, and what is the key difference between a mutex and a binary semaphore? (ownership and priority inheritance)
- Why must you never use a mutex from an ISR?
- What is priority inversion? Walk through the unbounded case. What was the Mars Pathfinder failure and how was it fixed?
- What is priority inheritance, and what is priority ceiling? What does priority inheritance not solve?
- Name the four Coffman conditions for deadlock and how you break each. How do you prevent deadlock in practice? (lock ordering, timeouts, single-lock design)
- What is a race condition between two tasks, and what are your options to prevent it?
- What is a critical section in an RTOS, and how does `taskENTER_CRITICAL` differ from `vTaskSuspendAll`?

**Tasks, stacks and context**
- What exactly is saved and restored on a context switch on Cortex-M? (automatic vs manual stacking, R0-R3/R12/LR/PC/xPSR pushed by hardware; R4-R11 by the port; FPU context if used)
- What is in a TCB?
- How do you size a task stack? How do you measure actual usage? (`uxTaskGetStackHighWaterMark`, fill pattern watermarking)
- What happens on stack overflow, and how do you detect it? (`configCHECK_FOR_STACK_OVERFLOW` methods 1 and 2, MPU guard region)
- What are PSP and MSP on Cortex-M, and which does a task use vs an ISR?

**IPC and interrupts**
- Queue vs event group vs task notification vs mailbox — when is each right?
- What is interrupt latency, and what contributes to it? (highest-priority ISR runtime, critical sections, tail-chaining, late arrival)
- Why must ISRs use the `FromISR` API variants?
- Explain deferred interrupt processing and why it bounds ISR duration.
- What is priority inversion caused by an ISR, and what is `configMAX_SYSCALL_INTERRUPT_PRIORITY` for?
- Why do Cortex-M NVIC priorities run "lower number = higher priority" while FreeRTOS task priorities run the opposite way, and what bug does confusing them cause?

**RTOS vs bare metal**
- When do you choose an RTOS over a superloop, and when is a superloop genuinely better?
- What does an RTOS cost you? (RAM per task, scheduler overhead, harder WCET analysis, priority-inversion risk)
- What is a cooperative superloop with a scheduler table, and when is it a good middle ground?

**FreeRTOS specifics**
- Walk through `xTaskCreate` vs `xTaskCreateStatic` — where does the stack live in each?
- Explain heap_1 through heap_5 — which allows freeing, which coalesces, which spans non-contiguous regions, and which would you ship?
- What does `configASSERT` catch, and why should it stay enabled in production builds?
- What is tickless idle and what must you do for the RTOS to keep correct time across it?
- What are stream buffers and message buffers, and what constraint do they impose? (single writer, single reader)
- `vTaskDelay` vs `vTaskDelayUntil` — which one gives a genuinely periodic task, and why does the other drift?
- Task notifications vs queues and semaphores — notifications are the fastest primitive and need no extra RAM, but what is the hard limitation? (strictly one receiving task; no broadcast)

**Zephyr contrast (worth knowing if they ask)**
- How does Zephyr thread priority numbering differ from FreeRTOS? (inverted: in Zephyr lower, and negative for cooperative, is higher priority — the reverse of FreeRTOS)
- What is a Zephyr workqueue (`k_work`) and when would you use it instead of spawning a thread? (pre-existing thread with a FIFO job list; the Zephyr form of ISR-to-thread deferral)
- How do devicetree and Kconfig divide responsibility in Zephyr? (devicetree describes what hardware exists; Kconfig decides what is compiled in)

**Bootloader**
- What are a bootloader's responsibilities?
- Describe the flash memory layout for a device with a bootloader and application. Where do the vector tables live?
- What is vector table relocation and how do you do it on Cortex-M? (`SCB->VTOR`)
- Walk through jumping from bootloader to application: what must you do first? (deinit peripherals, disable interrupts, set MSP from the app's vector table, then branch to the reset vector)
- Why must you deinitialise peripherals and clocks before the jump?
- How does the bootloader decide whether to boot the app or stay in update mode? (magic value in a shared RAM region or backup register, GPIO strap, invalid app signature, boot counter)
- How do you verify the application is valid before jumping to it?

**A/B partitions, rollback, integrity**
- How does an A/B (dual-bank) update scheme work, and what problem does it solve?
- What is the tradeoff of A/B versus a single-slot update with a download staging area? (flash cost vs bricking risk)
- What is rollback, and what is an anti-rollback counter for? Where must the counter live so an attacker cannot reset it?
- How do you confirm a new image is good rather than rolling back? (boot-confirm flag written by the application after a self-test, watchdog-driven revert if it never confirms)
- CRC vs checksum vs cryptographic hash vs digital signature — what does each actually protect against?
- Explain secure boot and the chain of trust from ROM to application. Where is the root public key stored?
- Why is verifying a signature over the image not enough on its own? (rollback to an older signed-but-vulnerable image)

**OTA**
- Walk through an OTA update end to end, from server to running new firmware.
- How do you make an update atomic and power-fail safe? What is the exact moment of commitment?
- What happens if power is lost mid-write? Mid-erase? How do you resume rather than restart?
- How do you handle a device that boots the new image but cannot reach the network?
- How do you prevent bricking a fleet with a bad image? (staged rollout, canary devices, forced rollback path, factory-reset image)
- What is delta/differential OTA, and what is its cost? (smaller transfer, more complex and riskier apply step)
- How do you manage the watchdog during a long flash write or a slow download?
- ESP-IDF specifics: what do the `ota_0`/`ota_1` partitions and the `otadata` partition do? What do `esp_ota_begin`/`esp_ota_write`/`esp_ota_end`/`esp_ota_set_boot_partition` and `esp_ota_mark_app_valid_cancel_rollback` each do?

**Flash behaviour**
- Why must flash be erased before it is written, and what is the erase granularity vs write granularity?
- What is write endurance, and what is a typical cycle count? What is wear levelling and where does it belong?
- What is read-while-write, and why does it matter on a single-bank device during OTA? (the CPU cannot fetch code from a bank being erased)
- Why can flash writes only turn bits one way, and what does that enable for tricks like write-once flags?
- How do you store configuration that survives updates and power loss? (dedicated partition, journal/log-structured store, ESP-IDF NVS, wear-levelled key-value)

---

## 10. Low-Power Design

- Explain sleep vs stop vs standby on a typical MCU — what is retained, what wakes you, what is the wake latency?
- What wake sources would you use for a sensor node, and why?
- How do you build an average current budget and turn it into a battery-life estimate?
- What is duty cycling, and how do you pick the sleep/active ratio for a multi-year target?
- What is coulomb counting vs a voltage-based fuel gauge?
- How does clock source selection (HSI/HSE/LSE) affect power and wake time?
- Why use DMA instead of interrupt-driven CPU wakeups?
- What is tickless idle and why does it matter?
- How do you measure microamp sleep current in the lab — equipment and pitfalls (transition spikes, ammeter burden voltage)?
- What silently kills low-power numbers? (floating GPIOs, pull-ups left enabled, debug port active, unterminated ADC inputs)

---

## 11. System Architecture & Product Lifecycle

- Design a battery-powered sensor node that reports every 15 minutes and must last 5 years — walk through the power budget. **(Rehearse this on paper; extremely likely.)**
- How do you decide between an MCU and an MPU running Linux?
- What criteria drive MCU selection for mass production (peripherals, memory headroom, cost, lifecycle, second sourcing)?
- How do you handle a component going end-of-life mid-production?
- Design the firmware architecture for an ECG device / smart scale / livestock tracker — sensors, state machine, connectivity, storage.
- How would you structure a state machine for a device with sleep, measuring, transmitting and fault states?
- Explain producer-consumer with a ring buffer — when and why on an MCU?
- How do you timestamp sensor readings accurately when the MCU wakes intermittently?
- What is your approach to fault tolerance — watchdogs, brown-out, error recovery, field diagnostics?
- Walk through your process from requirements to production — design reviews, EVT/DVT/PVT, certification planning.
- How would you design logging/telemetry for a field device with no easy physical access?

---

## 12. Wireless Connectivity (BLE / Wi-Fi)

- Explain BLE GAP vs GATT — advertising, discovery, and the services/characteristics model.
- How does connection interval affect power and latency, and how would you tune it for a wearable?
- Walk through BLE pairing and bonding — what security level for a healthcare device?
- What limits BLE throughput, and how would you stream an ECG waveform within it?
- Nordic nRF vs ESP32 for BLE — toolchain and power profile differences?
- ESP32/ESP8266 as a Wi-Fi module vs running application logic directly on it?
- How do you provision Wi-Fi on a headless device (SoftAP, BLE-assisted, captive portal)?
- How do BLE and Wi-Fi coexist on the same 2.4 GHz radio, and what interference issues have you hit?
- PCB trace antenna vs chip antenna — tradeoffs in space, cost, range, tuning.
- Antenna keepout and 50 Ω matching considerations in layout.
- What does FCC/CE certification involve, and why does a pre-certified module reduce the burden?
- When would you choose LoRa/LoRaWAN over Zigbee, Thread or NB-IoT for livestock/agriculture? What does a LoRaWAN gateway do?
- MQTT vs CoAP vs HTTP for a constrained device?
- How do you implement TLS on a constrained MCU, and what are the risks?

---

## 13. Embedded Security

- Explain secure boot and chain of trust — how do you ensure only signed firmware runs?
- Where do you store cryptographic keys — internal flash, secure element, OTP fuses?
- When would you add a discrete secure element (e.g. ATECC608) vs MCU built-in crypto/TrustZone?
- How do you encrypt firmware images for OTA, and how does the device verify authenticity before flashing?
- How do you lock down SWD/JTAG and enable flash readout protection before shipping? What is the risk of forgetting?
- Threat-model a connected health device — top attack surfaces (BLE, cloud API, physical debug, update path).

---

## 14. Embedded Linux & Edge Computing

**Your existing strength — revise for fluency, do not over-invest. Relevant to their edge/AI and camera work.**

- Walk through the boot chain: ROM bootloader → SPL/MLO → U-Boot → kernel → init/systemd. What does each stage do?
- Why does a two-stage bootloader (SPL then full U-Boot) exist?
- Why does Linux need a Device Tree? How does a `compatible` string bind to a driver's `of_device_id` table?
- What is a Device Tree overlay and when would you use one?
- Walk through `struct file_operations` — what happens on `open()`/`read()`/`ioctl()`?
- Why can you not dereference a user-space pointer directly in a driver? Explain `copy_to_user`.
- `kmalloc` vs `vmalloc` — why does physical contiguity matter for DMA?
- Spinlock vs mutex in kernel code — why can you not sleep holding a spinlock?
- Top half vs bottom half; tasklet vs workqueue vs threaded IRQ — which can sleep?
- Yocto vs Buildroot — when would you choose each?
- What is a layer, a recipe, a `.bbappend`, and what goes in a BSP layer?
- How does an A/B update scheme work, and what problem does it solve? Compare RAUC, SWUpdate, Mender.
- What is a sysroot, and why does ABI mismatch (soft-float vs hard-float, glibc vs musl) break binaries?
- What does PREEMPT_RT change, and how do you measure latency with `cyclictest`?
- How does V4L2 abstract a camera driver — what is the buffer flow?
- How would you build a GStreamer pipeline capturing from a camera, hardware-encoding, and streaming out?
- What is DMA-BUF / zero-copy, and why does it matter between camera, ISP, encoder and inference engine?
- What is INT8 quantization and why is it necessary on an edge NPU? Accuracy/latency tradeoff.

---

## 15. Behavioural & Consultancy Fit

- Walk me through the hardest bug you have debugged — how did you isolate it?
- Tell me about a bug nobody else could find. What made it invisible?
- Describe owning a product end-to-end from requirements to a manufactured unit.
- A board comes back from fab and does nothing on power-up — what is your triage?
- Tell me about finding a schematic error after boards were already spun.
- How do you work with hardware engineers when you suspect the bug is in the board, not your code?
- Describe a client changing scope mid-project. How did you handle the deadline?
- How do you estimate firmware effort? Have you been badly wrong?
- Tell me about supporting a production or field issue after handoff to manufacturing.
- How do you structure firmware code review — what do you look for in embedded C?
- Git workflow for firmware shared across hardware revisions?
- Have you set up CI for firmware — what builds/tests without hardware in the loop?
- Experience with hardware-in-the-loop testing?
- How do you unit test firmware that touches hardware registers (mocking, HAL abstraction, host-based tests)?

---

## 16. Study Resources

**Power electronics & metering (Tier 1 — start here)**
- Erickson & Maksimovic, *Fundamentals of Power Electronics* — buck/boost/SEPIC/flyback derivations, CCM/DCM, loss analysis, compensation
- Mohan, Undeland & Robbins, *Power Electronics: Converters, Applications, and Design* — inverters, SPWM, three-phase
- TI *Power Topologies Handbook* (free PDF) — fast topology-selection reference
- ST AN4013 / AN2794 and equivalent motor-control timer app notes — complementary PWM and dead-time in firmware
- ADI current-sensing app notes (AN-1560, MT-101/MT-060) — shunt sizing, amplifier selection, Kelvin sensing
- ADE7753/ADE9000 or STPM32/34 datasheets and app notes — how metering ICs implement RMS, power, energy accumulation
- Battery University (batteryuniversity.com) — CC/CV charging, SoC estimation, cell balancing

**Hardware, SI, EMC (Tier 1)**
- Henry Ott, *Electromagnetic Compatibility Engineering* — the EMC reference
- Howard Johnson & Graham, *High-Speed Digital Design: A Handbook of Black Magic* — transmission lines, termination, return paths
- Eric Bogatin, *Signal and Power Integrity — Simplified*
- Horowitz & Hill, *The Art of Electronics*
- IPC-2221 / IPC-2141 — trace width/current and controlled-impedance standards
- TI app notes: SLVA073 (decoupling), SLUA271 (power sequencing); ADI AN-139, AN-202 (mixed-signal grounding/layout)
- FCC Part 15 / CISPR 32 — skim scope and limits

**EDA tools (Tier 1 — hands-on, not reading)**
- Altium Designer documentation + "Altium Academy" video series
- Cadence OrCAD Capture / Allegro PCB Editor user guides, Constraint Manager tutorials
- LTspice getting-started tutorials (Analog Devices) — free stand-in for PSpice practice

**Embedded C & firmware (Tier 2)**
- Barr Group *Embedded C Coding Standard* and the Embedded C Quiz — Michael Barr's canonical question set
- Memfault Interrupt blog — "How to debug a HardFault on an ARM Cortex-M MCU", "A Guide to Watchdog Timers"
- John Regehr, "A Guide to Undefined Behavior in C and C++"
- Eric S. Raymond, "The Lost Art of Structure Packing"
- Joseph Yiu, *The Definitive Guide to ARM Cortex-M3/M4/M0+ Processors*
- MISRA C guidelines (misra.org.uk)

**Design patterns, STM32 & ESP-IDF (Tier 2)**
- *Mastering the FreeRTOS Real Time Kernel* (free PDF) — queues, notifications, ISR-safe API, priority inversion
- James Grenning, *Test-Driven Development for Embedded C* — linker-seam mocking, host vs HIL tradeoffs
- Miro Samek, *Practical UML Statecharts in C/C++* and the QP/C framework — hierarchical state machines
- ESP-IDF Programming Guide — read `esp_driver_i2c/i2c_master.c`, `esp_event` docs, and `esp_check.h` directly
- ST community FAQ "STM32 HAL UART driver — API and callbacks"; AN4989 debug toolbox
- ETL (etlcpp.com) for fixed-capacity containers; Dan Saks / Odin Holmes CppCon talks on templates as a zero-overhead HAL
- Read production source: `espressif/esp-idf` and `stm32duino/Arduino_Core_STM32`

**RTOS, bootloader & OTA (Tier 3)**
- *Mastering the FreeRTOS Real Time Kernel* + the FreeRTOS API reference (free PDFs)
- Memfault Interrupt blog — firmware update/OTA series, "Device Firmware Update Cookbook", watchdog and rollback posts
- ESP-IDF OTA and partition-table documentation; `esp_ota_ops.h`
- Joseph Yiu, *Definitive Guide to Cortex-M* — VTOR, MSP/PSP, NVIC priorities, exception entry/exit
- Zephyr MCUboot documentation — the reference open-source secure bootloader with A/B and anti-rollback

**Sensors & analog front end (Tier 2)**
- TI ADS1292/ADS1298 and ADI AD8232 ECG AFE datasheets and app notes — instrumentation amps, RLD, mains rejection
- HX711 datasheet + a strain-gauge/Wheatstone primer — load cell calibration and tare
- IEC 60601-1 and 60601-2-25/2-27 summaries — enough vocabulary for the healthcare vertical

**Architecture, low power, RTOS, connectivity (Tier 3)**
- Elecia White, *Making Embedded Systems* — architecture, state machines, debugging
- Qing Li, *Real-Time Concepts for Embedded Systems*
- Nordic Power Profiler Kit II docs; ST AN4899 (STM32 low-power modes)
- Nordic DevZone BLE tutorials; Bluetooth SIG GAP/GATT primer; Townsend et al., *Getting Started with Bluetooth Low Energy*
- Espressif ESP-IDF programming guide — Wi-Fi provisioning and coexistence
- OWASP IoT Top 10; ARM PSA Certified security model

**Embedded Linux (Tier 4 — revision only)**
- Bootlin training materials (free slides + labs) — best single source for boot chain, kernel, rootfs, drivers
- Corbet, Rubini & Kroah-Hartman, *Linux Device Drivers 3rd Ed.*
- "Device Tree for Dummies" (Petazzoni, Bootlin)
- Yocto Project Mega Manual; Buildroot manual

**Interview practice**
- r/embedded — search "interview questions" threads for recent employer-reported sets
- Embedded.fm podcast back catalogue — engineering war stories aligned to consultancy-style questions
- Rehearse 2–3 STAR-format stories each for: hardest bug, hardware/firmware conflict, scope-change crunch, production support
